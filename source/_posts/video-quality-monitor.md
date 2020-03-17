title: 前端视频质量监控
date: 2018-9-7 00:10
categories: 前端
tags: [视频, 质量监控]
---
## 背景
业务中使用到视频播放后，一些不确定的因素例如用户端网络异常、CDN 异常等等，导致视频加载缓慢和发生卡顿，这些质量问题会给用户体验带来较大的伤害，也会影响产品的留存转化率。因此对线上视频进行质量监控意义很大，可以让我们明确知道用户端的异常发生概率，以便做进一步的优化。
<!-- more -->
在我们的业务中，是不支持用户自己控制视频播放/暂停以及播放进度的，所以我们不需要考虑一些用户控制的边界情况；另外一部分视频会在挂载到真实 DOM 前做预加载（preload）处理，因此我们要留意下这类视频和无预加载的视频的差异。  
另外我们的业务只在 PC 上，而且用户使用的都是现代浏览器（Chrome 为主），所以不需要考虑移动端的兼容性以及老浏览器的兼容性。

## 思路与实现
`video` Element 在播放视频的过程中会触发一系列的媒体事件（Media events），通过查阅 MDN Media events 列表，可以发现值得关注的几个事件分别是 `canplay` `canplaythrough` `error` `loadeddata` `loadedmetadata` `loadstart` `play` `playing` `waiting`。  

首先 `error` 事件可以不用特别关注，和常规错误监控一样处理即可；重点在于如何统计加载耗时及卡顿现象。  

接下来我们拿一个视频简单做下试验，看下这几个事件触发的时机以及先后顺序。  
**测试环境：macOS 10.13.6 / Chrome 68**

``` JavaScript
['loadstart', 'loadedmetadata', 'loadeddata', 'waiting', 'canplay', 'canplaythrough', 'error', 'play', 'playing', 'ended'].forEach((eventName) => {
  video.addEventListener(eventName, () => {
    const key = `_${eventName}_time`;
    video[key] = Date.now();

    console.log(eventName, Date.now());
  });
});
```

对于一个无预加载的视频来说，从开始加载到正常播放，输出日志为：

| event | time | cost |
|---|---|---|
| play | 1536561748438 | 0 |
| waiting | 1536561748438 | 0 |
| loadstart | 1536561748455 | 17 |
| loadedmetadata | 1536561748588 | 133 |
| loadeddata | 1536561748613 | 25 |
| canplay | 1536561748621 | 8 |
| playing | 1536561748625 | 4 |
| canplaythrough | 1536561748627 | 2 |

对于一个有预加载处理的视频来说，对应的输出日志为：

|event|time|cost(ms)|
| --- | --- | --- |
| loadstart | 1536562506189 | 0 |
| loadedmetadata | 1536562506271 | 82 |
| loadeddata | 1536562506304 | 33 |
| canplay | 1536562506305 | 1 |
| canplaythrough | 1536562506307 | 2 |
| play | 1536562526391 | 20084 |
| playing | 1536562526392 | 1 |

观察上述事件触发的顺序，对应 MDN 上对事件的描述，与视频初始化相关的几个事件 `loadstart` `loadedmetadata` `loadeddata` 在视频加载的时候会依次触发，分别代表着开始加载、元信息加载成功、首帧加载成功，因此在统计视频加载延迟的时候，我们基本可以确定统计这三个事件触发的时间差即为加载延迟。

``` JavaScript
const logLatency = (video) => {
  if (video._loadstart_time && video._loadedmetadata_time && video._loadeddata_time) {
    const loadedmetadataCost = video._loadedmetadata_time - video._loadstart_time;
    const loadeddataCost = video._loadeddata_time - video._loadstart_time;

    logger.log('Latency', [loadedmetadataCost, loadeddataCost]);
  }
};

['loadstart', 'loadedmetadata', 'loadeddata'].forEach((eventName) => {
  video.addEventListener(eventName, function callback() {
    const key = `_${eventName}_time`;
    video[key] = Date.now();

    // 在 loadeddata 时记录延迟
    if (eventName === 'loadeddata') {
      logLatency(video);
    }

    // 延迟只需记录一次，相关的监听器触发过后则可移除
    video.removeEventListener(eventName, callback);
  });
});
```

接下来我们在播放过程中用 Chrome Dev Tool 的 Network 模拟弱网（网络抖动），输出日志为：

|event|time|cost(ms)|
|---|---|---|
| waiting | 1536565784729 | 0 |
| canplay | 1536565785075 | 346 |
| playing | 1536565785076 | 1 |

观察上述事件触发的顺序，对应 MDN 上对事件的描述，与视频初始化相关的几个事件 `waiting` `canplay` `playing` 在视频播放过程中发生卡顿到再次播放的时候会依次触发，分别代表着等待、加载了足够的数据可以播放、`play` 事件后有足够多的数据可以开始播放或者从卡顿缓冲中恢复过来。在统计视频卡顿的时候，我们基本从这三个事件入手。  

对比初次加载时的事件触发顺序，可以不用关注 `canplay` 事件，只关注 `waiting` 和 `playing`，但是这里又有一个问题就是无预加载的视频在刚开始播放时会触发 `waiting` 和 `playing`，预加载的视频在刚开始播放时只会触发 `playing`，所以我们要忽略掉第一次 `playing` 事件，从第二次开始记录 `waiting` 和 `playing` 的时间差，即为卡顿时长（卡顿缓冲耗时）。  

``` JavaScript
const logBuffer = (video) => {
  video._buffer_times += 1;

  // 初次缓冲为正常缓冲，不上报
  if (video._buffer_times === 1) {
    return;
  }

  if (video._waiting_time && video._playing_time) {
    const bufferCost = video._playing_time - video._waiting_time;
    logger.log('Buffer', [bufferCost]);
  }
};

video._buffer_times = 0;

['waiting', 'playing', 'ended'].forEach((eventName) => {
  video.addEventListener(eventName, () => {
    const key = `_${eventName}_time`;
    video[key] = Date.now();

    // 在 playing 时记录卡顿
    if (eventName === 'playing') {
      logBuffer(video);
    }

    // 播放结束重置卡顿计数器
    if (eventName === 'ended') {
      video._buffer_times = 0;
    }
  });
});
```

注意在视频播放结束的时候，我们要把 `_buffer_times` 计数器重置为 0，否则在该视频二次播放的时候，初次加载就被认为是卡顿。

## 优化
当卡顿超过一定时长时（暂定 1 秒），用户可能会失去耐心关闭页面或者刷新页面，如果我们一味的等待视频缓冲完再统计，可能会丢失这种情况的数据。另外实际上当卡顿时间超过忍耐时长后，再统计具体的时间已经没有太大意义了。因此我们可以对上面的方案进行优化，增加卡顿超时直接统计的逻辑。  

原理也很简单，在 `waiting` 时设置一个定时器与 `playing` 监听器回调竞争：

``` JavaScript
const BufferTimeout = 1000; // 卡顿超时阈值

const logBuffer = (video, timeout) => {
  if (video._buffer_reported) {
    video._buffer_reported = false;
    return;
  }

  video._buffer_times += 1;
  video._buffer_reported = true;

  // 初次缓冲为正常缓冲，不上报
  if (video._buffer_times === 1) {
    return;
  }

  // 超时上报
  if (timeout) {
    logger.log('Buffer ', [BufferTimeout]);
    return;
  }

  // 正常上报
  if (video._waiting_time && video._playing_time) {
    const bufferCost = video._playing_time - video._waiting_time;
    logger.log('Buffer', [bufferCost]);
  }
};

video._buffer_times = 0;
video._buffer_reported = false;

['waiting', 'playing', 'ended'].forEach((eventName) => {
  video.addEventListener(eventName, () => {
    const key = `_${eventName}_time`;
    video[key] = Date.now();

    // 加载超时直接记录
    if (eventName === 'waiting') {
      setTimeout(() => {
        logBuffer(video, true);
      }, BufferTimeout);
    }

    // 在 playing 时记录卡顿
    if (eventName === 'playing') {
      logBuffer(video);
    }

    // 播放结束重置卡顿计数器
    if (eventName === 'ended') {
      video._buffer_times = 0;
    }
  });
});
```

其实这里的超时逻辑还可以用 `Promise.race` 来实现，不过现在这样通过 Flag 来控制竞态也是没啥问题的。

## 参考
- [Media events - MDN](https://developer.mozilla.org/en-US/docs/Web/Guide/Events/Media_events)


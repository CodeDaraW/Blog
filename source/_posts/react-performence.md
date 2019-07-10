title: React 应用性能优化一例
date: 2019-07-02 21:26:00
categories: 前端
tags: [React]
---
## 背景
在我们的应用中有一个存储在 Redux 中的全局状态机，保存着整个应用的核心状态，其类型大概如下：
``` typescript
enum FSMStatus {
    Unknown = 0,
    Active = 1,
    Inactive = 2,
}

interface FSM {
    global_id: number; // 状态机实例的全局唯一 ID
    timestamp: number; // 当前时间，每隔一秒会变化
    fsm_status: FSMStatus; // 整个状态机的状态
    x_mod_status: FSMStatus; // x 模块的状态
    x_mod_data: any; // x 模块的核心数据
    y_mod_status: FSMStatus; // y 模块的状态
    y_mod_data: any; // y 模块的核心数据
}
```
还有一些 React 组件，其中 `Main` 组件包含了 `Inner` 组件，`Inner` 组件中使用了 `rc-tooltip` 实现了弹窗效果。弹窗中有一个列表，列表中有一条带有特殊样式的处于激活状态的数据，且可以通过按键操作切换处于激活状态的数据项。  

某天突然用户报告说操作弹窗列表时感到卡顿，而且在弹窗开着的时候其他模块会明显感觉比较卡。
<!-- more -->
## 解决过程
### 定位原因
拉上一个大佬一起帮忙看了 Performence 的相关信息，发现大部分掉帧时伴随着大量 CPU 占用，而相关时间段内大部分 CPU 耗时在 DOM 操作上，DOM 操作的来源在于缩略图所使用的 `rc-tooltip` 组件和和其使用的 `dom-align` 库。

通过阅读 `rc-tooltip` 相关实现源码得知，每次组件重新渲染时都需要重新计算相关的位置信息，以保证浮层以正确的大小展示在正确的位置。即这一块很难有优化空间，在现有的需求下放弃使用 `rc-tooltip` 我们也不大可能能写出一个性能明显更好的库。

### 解决问题
直觉猜测是上层业务组件（`Inner`）使用了 `fsm` 作为 `props` 的一部分，而 `fsm` 的频繁更新导致了组件的频繁重新渲染。
为了验证猜测，打开 `rc-tooltip` 弹窗，但不进行任何操作，录制性能信息，发现果然伴随着时间的变化（`fsm` 的变化），会出现来自 `rc-tooltip` 的 DOM 操作，即验证了猜测。

于是开始着手优化 `Inner` 组件，从 `fsm` 中只挑出使用到的属性：
``` typescript
import pick from 'lodash/pick';

// 和实际情况作了一定简化
export default connect(({ store }: { store: Store }) => {
    // before
    // return {
    //     fsm: store.fsm
    //     user_id: store.user_id
    // };

    // after
    const fsm = pick(store.fsm, ['x_mod_status', 'y_mod_status']);
    return {
        fsm，
        user_id: store.user_id
    };
})(Toolbar);
```

并将 `React.Component` 更换为 `React.PureComponent`，以获取自动化的 `shouldComponentUpdate` 检查。

但是在这一顿操作时候性能依然没有得到改善，debug 后发现实际每次 `props.fsm` 都是新构造的对象，而 `React.PureComponent` 自动化的 `shouldComponentUpdate` 检查只会做 shallow compare，导致了前一次与后一次的 `props` 不相等，`shouldComponentUpdate` 永远返回 `false`。  
于是自己手动填充 `shouldComponentUpdate` 的逻辑，借助 `lodash/isEqual` 对 `props` 和 `state` 做 deep compare ：

``` typescript
import isEqual from 'lodash/isEqual';

shouldComponentUpdate(prevProps: IProps, prevState: IState) {
    // 每次的 fsm 都是新的对象，shallow compare 会认为变化了，需要 deep compare
    return !(isEqual(prevProps, this.props) && isEqual(prevState, this.state));
}
```

再次测试，在静止状态下 `fsm` 的变化不再触发 `rc-tooltip` 的重新渲染。但是在按住按键不放快速切换激活项的情况下，依然有卡顿感。通过观察 Performence 录制的信息，发现 `setState` 的性能损耗也很可观，在其调用链上发现最终和上面一样触发了 `rc-tooltip` 的重新渲染。

通过阅读代码发现在 `Main` 组件中存储了列表中激活状态的数据，并将这个状态从 `Props` 中依次传给了 `Inner` 、`rc-tooltip` 中的列表组件 ，在按住按键不放的时候，按键事件的回调中会不断 `setState` 更新激活状态的数据 ，从而导致组件从 `Main` 开始依次往下重新渲染，在 `Inner` 这里触发了 `rc-tooltip` 的重新渲染。

直觉的思路是在组件内部存储一个私有变量，延迟并批量进行 `setState` 的调用，但想了下这样并不可行，因为列表页当前被激活的选项的 CSS 样式效果是需要实时跟着当前状态变化的。

批量更新的思路不可行，只能想办法把这个被频繁更新的状态和使用了 `rc-tooltip` 的 `Inner` 剥离开来，从而避免 `Inner` 的重新渲染和 `rc-tooltip` 的重新渲染。通过阅读代码，发现这个状态实际只有列表组件才会使用到，于是将该状态和部分逻辑剥离到最下层的列表组件中。

经过再次修改后，验证发现这一块不再有性能问题了。

## 结论
### 尽可能细化使用到的对象 props 属性
不要直接将整个 `fsm` 挂在 props 上，由于它是个对象，且上面有很多无关的信息在频繁地更新，即使 deep compare 也可能会因为不相关的属性变化带来不必要的重新渲染。

因此要么借助 `lodash/pick` 将 `fsm` 使用到的属性 `pick` 出来，并在 `shouldComponentUpdate` 中借助 `lodash/isEqual` 进行 deep compare ；要么将 `fsm` 上使用到的属性直接挂到 `props` 上（但是这对 `xx_data` 这种对象依然无解），并配合 `React.PureComponent` 使用。

### 尽可能将 state 存到真正使用到的子组件中
将状态放在祖先组件中，通过一层层的 `props.xx` 和 `props.onXXChange` 不但开发体验糟糕，还会导致无意义且无法避免的不相关的组件重新渲染。

如果真的是很多地方使用到的状态，可以放在 `redux` 中。
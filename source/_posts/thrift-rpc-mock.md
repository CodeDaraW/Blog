title: Thrift RPC Mock 方案探索
date: 2019-06-12 11:53:00
categories: 服务端
tags: [RPC, Thrift, Mock, Node.js]
---
## 背景分析
在传统前后端分离开发的场景下，前端和后端一般定好 HTTP API 接口后就各自进行开发，前端开发中使用 EasyMock、webpack-api-mock 等平台/工具进行接口的 mock，后端通过 Postman / curl 等工具进行接口的自测。

在微服务场景下，各服务之间通过 IDL 定义好 RPC 接口。但是接口调用方依然有 mock 接口的需求，接口提供方也有着自测接口的需求。公司内的服务化平台已经提供了较为完善的接口测试工具，自己实现一个相对也比较容易，但目前却没有一个比较完善的 RPC Mock 方案。

在新项目启动后，前端、API 层和依赖的 Service 往往同步开始开发，只要依赖的 Service 未提供，API 和前端的开发、自测都会被阻塞，在侧重数据展示类需求的项目中这种问题更加严重。

所以，有必要尝试探索一套 RPC Mock 的方案，在保证开发者使用体验的前提下，解决上述问题。

<!-- more -->
## 方案调研
虽然我们的目标和常见的 HTTP API Mock 不一样，但是在设计思路上，RPC Mock 与 HTTP API Mock 其实没有本质的区别，因此我们将传统的 HTTP API Mock 方案也纳入调研范围内。

### [Mock.js](http://mockjs.com)
Mockjs 侧重 Mock 数据的生成，自定义了一套描述数据的 DSL，可以生成较为真实的 Mock 数据。
另外 Mockjs 还通过在开发本地劫持了 XHR 对象，提供了简单的 HTTP API Mock 的功能。

### [EasyMock](https://easy-mock.com/)
Easy Mock 属于 HTTP API Mock。
在数据生成方面，需要用户自己编辑接口信息以及 Response 结构体，引入了 Mockjs，使用 Mockjs 的语法描述 Response 结构体以实现生成随机数据，平台服务端还提供了 HTTP 接口服务可供调用。

### [mocker-api](https://www.npmjs.com/package/mocker-api)
同上属于 HTTP API Mock，前身为 [webpack-api-mocker](https://www.npmjs.com/package/webpack-api-mocker)。
在数据生成方面，用户可以从方法的维度配置固定的 Mock 数据或者配置数据生产函数，并在开发本地起了一个 HTTP Server，将本地开发的请求代理到了这个 Server 上。

### [lushijie/thrift-mock](https://github.com/lushijie/thrift-mock/)
只有 Mock Data 的部分，根据 Thrift 的字段与类型信息自动生成 Mock 数据。

### [adispring/thrift-mock](https://github.com/adispring/thrift-mock)
只有 Mock Data 的部分，和上面的类似，可以进行二次修改生成的数据。

## 总结
在 HTTP API 方面，由于协议之类的是非常通用的，所以已经有不少比较成熟的方案，而 RPC 由于各家公司的技术架构不一样，目前没有什么比较成熟的通用方案，而目前公司内也没有这方面的方案。

借鉴比较成熟的那些 HTTP API Mock 方案，基本可以确定我们的思路，方案整体分为两部分：Mock 请求 + Mock 数据。首先我们需要搭建一个能够和正常服务一样可被调用的 Mock 服务，接着在调用时返回 Mock 数据。

## 方案设计 V1（CLI）

### 思路分析
上述方案中最成熟的应该是 EasyMock，受其影响，第一反应是做成类似的样子，开发一个平台，开发者在平台上配置 Mock 数据，平台提供可被调用的 Mock RPC Server。但是由于 Thrift RPC 协议的特殊性，无法简单的在一个 IP:PORT 下部署多个服务，所以平台化提供可被调用的 Mock 服务不太可能实现。

既然无法做到类似 EasyMock 那样，那退一步和 mocker-api 那样在开发本地起一个 RPC Mock Server 肯定是没有问题的，例如把 RPC Mock Server 起在了本地的 8888 端口，业务中想要调用本地的服务而不是真实的服务时，只需要将 RPC Client 配置中的 Service Name（服务发现使用了 Consul ）改为写死的 IP:PORT 127.0.0.1:8888 即可。

于是，第一版方案的思路大致如下：
初版方案的设想是做成类似 mocker-api 的 CLI。CLI 想要在本地启动 RPC Server，则必须要有的三个信息：Service Name、Thrift 文件位置以及服务端口，另外还需要为用户提供可选的类似 mocker-api 的配置 Mock 数据的入口。

有了上面的基本信息后，就足以起一个 RPC Server 了，接下来是 Mock 数据生成的工作。生成 Mock 数据主要有两个思路，自动生成与用户配置。
1. 自动生成数据：已经有 Thrift IDL 文件的情况下，可以 parse thrift 文件得到数据结构字段和类型等信息，并根据类型信息使用 Mockjs 构造随机数据。
2. 用户配置数据：Mock 数据配置文件的书写方式和 mocker-api 也很相似，提供一个 `commonjs` 文件，默认输出的对象是 key 为方法名，value 是对象或者是函数。例如我们需要配置 ID 生成器服务 `IDGenService` 的 `Gen` 方法：

- 当使用固定对象时，每次返回的都是配置的 Response。
``` js
// IDGenService
module.exports = {
    Gen: {
        Id: "123456",
        Extra: {
            Message: "success",
            Code: 0
        }
    }
};
```

如上在调用 `rpc.IDGenService.Gen(req)` 时，返回的永远是：
``` js
{
    Id: "123456",
    Extra: {
        Message: "success",
        Code: 0
    }
}
```

- 有时候我们想配置一些简单的规则，以避免固定对象导致业务逻辑冲突：
``` js
// IDGenService
let globalId = 100;
module.exports = {
    Gen: () => ({
        Id: `${globalId++}`,
        Extra: {
            Message: 'success',
            Code: 0
        }
    })
};
```

如上在调用 `rpc.IDGenService.Gen(req)` 时，返回的 `Id` 每次都会递增：
``` js
{
    Id: "100", // 101, 102, 103, ...
    Extra: {
        Message: "success",
        Code: 0
    }
}
```

综上，用户启动 CLI 后，CLI 会在本地起一个 RPC Server，收到调用请求后，如果用户为该方法配置了 Mock 数据，则返回配置好的 Mock 数据，否则根据 Thrift 文件的类型信息动态构造出 Mock Response 内容。

![CLI 处理流程](//sf1-ttcdn-tos.pstatp.com/obj/ttfe/ev/4d42f7aa78192ebab572cbe468e984b9.png)

### 方案实现
在确定方案后，很快开发出了基本可用的版本，由于和公司内基础设施耦合比较严重，这里不放出相关实现。

### 方案总结
在最近的两个项目中使用了上面的 CLI，基本能满足业务开发中的 RPC Mock 需求，但是抛开 TODO 中未实现功能点（例如 HotReload）外，发现了一些问题：

__不能控制方法级别的 Mock__
比如 `IDGenService` 有两个方法 `Gen` 和 `GenMulti`，我只想 Mock `Gen` 方法，不想 Mock `GenMulti` 方法，在这种方式下没有比较简单的实现方式。

__自动 Mock 数据基本不可用__
直接把 Thrift 中的类型信息转为 Mockjs 的配置会存在下面两个问题，从而导致数据基本不可用：
1. 可选项的处理
可选项存在两种情况：
- 业务意义上的选填，这种可选可不选，构造 Mock 数据的时候可以随机填写；
- 迭代过程中添加的字段，遵循 Thrift 的最佳实践，为了保证线上服务的正常运行，新增的字段应该选为可选项，但是在业务上是必选的，如果不选可能会报错，而我们在构造 Mock 数据的时候是不知道到底是不是该必选。
2. Mock 数据的真实性
很多字段是有约定或者有业务意义的，例如 `Code`、`Name`、`ID` 等等，不能直接按照 `int` `string` 等类型信息构造 mock 数据。轻则只影响展示效果，重则导致逻辑错误（例如生成一个不存在的 `Code`、负数的时长等等）。

__CLI 形式不够友好__  
1. 如果需要同时配置 N 个 Service 的 Mock，那就需要打开 N 个 Terminal Tab 并使用 CLI 起 N 个 Mock Server。
2. 提交代码的时候得注意不能把写死的临时配置代码带上去。


__语言无关__  
当然，这个 CLI 也有一定的优点，也是 Thrift RPC 协议的优点：他是语言无关的。无论你在开发的业务使用的什么语言和框架，都可以使用。

针对上面发现的一系列问题，首先决定暂时先放弃数据的自动化生成，只支持用户自行配置数据的方式；另外和 Mentor 沟通后，他提醒我一点，既然在 Node.js 项目中，RPC Middleware 是把 RPC Client 对象挂到了 `ctx` 上（这里是 Koa，在 Express 中是挂到了 `req` 上），那么完全可以通过中间件的形式再把 `ctx.rpc` 劫持了，在 Node.js 项目中做到更简单的实现、更精细的粒度控制和更好的使用体验。

于是，有了下面的 V2 版本的方案设计。

## 方案设计 V2（Middleware）
### 思路分析
由于方案 V2 中选择通过劫持 `ctx.rpc` 对象上的 `xService.yMethod` 函数实现 Mock，和 Mockjs 劫持 XHR 思路很相似，所以不再需要在本地启动 RPC Server，而且可以轻松同时配置多个 Service 的 Mock，方案 V1 中启动 RPC Server 必要的三个信息（Service Name、Thrift File、Server Port）也不再需要，开发者使用时只需要提供一个 Mock 数据配置文件即可。

在配置文件中，配置需要 Mock 的 Service 和方法，以及方法对应的 Mock 数据 / 数据生成函数（和 V1 相似），配置文件中暴露出来的对象的类型如下：

``` typescript
interface Mock {
    [serviceName: string]: {
        [methodName: string]: any; // object | Function
    } 
}
```

相比 V1 中的配置方式，只是加了一个层级用于配置 Service，以便同时做到多 Service 支持。

### 方案实现
同上，和内部基础设施耦合，不放出相关具体实现。  
使用时只需要在 rpc 中间件之后再添加 mock 中间件即可，对于业务代码的入侵极低。另外可以加一下环境的判断，只有在开发环境才会使用 mock 中间件。

要注意的是，由于需要劫持 ctx.rpc，所以必须要在 rpc 中间件之后使用 mock 中间件，否则 mock 不会生效。

``` js
const mockConfig = path.resolve(__dirname, 'mock/mock.config.js');
// 一定要在 rpc 中间件之后引入 mock 中间件
app.use(rpc(rpcConfig));
app.use(mock(mockConfig));
```

其中只有配置了 Mock 的 Service 的 Method 才会生效。例如 `IDGenService` 有 `Gen` 和 `GenMulti` 两个方法，如果我们配置了其他 Service 但是没有配置 `IDGenService`，那么 `IDGenService` 上的 RPC 方法被调用时会直接请求真实的服务；如果只配置了 `IDGenService` 的 `Gen` 方法，那么在调用 `ctx.rpc.IDGenService.Gen` 会命中 Mock，而 `ctx.rpc.IDGenService.GenMulti` 依然会请求真实的服务。即可以做到 Service + Method 粒度的配置。

在中间件中其实是做了两层嵌套循环：遍历 `ctx.rpc` 对象上的所有 Service，如果在配置文件中命中了该 Service，则对该 Service 对象再做一次遍历，遍历其上的所有 Method，如果在配置文件中命中了 Method，才会劫持该 Method。实现其实很简单，理解了实现也就理解了上面配置的 Service + Method 的粒度控制。

另外在实际使用中，建议在项目代码目录下添加一个 mock 文件夹，将配置文件放到该文件夹下，并将该文件夹添加到 nodemon 等监听文件修改热重载工具的监听范围内，这样在配置文件更新后也能获得热重载的能力，并且团队内可以共用 Mock 数据。

### 方案总结
V2 中间件的方案更多的是解决了 Node.js 项目的 Mock 实现方式，但我们还有其他方面需要优化：  

__Mock 数据构造__  
IDL 中的字段名字、类型是有用的，利用好是可以减少 Mock 数据的构造成本的。但是这块如何做好后续依然需要探索，目前手动构造的方案虽然需要成本，但是也是可以接受的。

__跨语言/框架支持__  
相比 V1 CLI 的形式，V2 中间件的形式目前只支持 Nodejs 的 Koa / Express 框架，但是 Mock 的需求是普遍存在的。由于 JS 动态语言的特点，所以劫持变量可以很简单的实现 Nodejs 项目的 Mock Middleware，如何做其他语言 / 框架的支持，后续依然值得探索。
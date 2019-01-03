在做个人网站的js拆分打包时，最终的解决方案是看着网上的教程手写了Bundle高阶组件来动态加载需要的组件。对于它的运用也仅仅是把路由拆开，访问不同的顶级路由进行动态加载，并没有对其原理进行深入的理解。直到看到了[React 的加载 loading 库——react-loadable](https://github.com/jamiebuilds/react-loadable)。

## react-loadable是什么？
Loadable提倡基于组件分割代码。
![image](http://cdn.blog.yangzhedi.com/loadable/loadable-splitting.png)
> route-centric code splitting is shit, component-centric splitting is cool as shit.

网上的翻译有很多，本文就不过多对readme操作了，简单的介绍一下就是：
Loadable 是一个高阶组件（简单来说，就是把组件作为输入的组件。高阶函数就是把函数作为输入的函数。在 React 里，函数和组件有时是一回事），一个可以构建组件的函数（函数可以是组件），它可以很容易的在组件层面分割代码包。


## react-loadable怎么用？
下面是github库里readme给的例子。
```
import Loadable from 'react-loadable';
import Loading from './my-loading-component';

const LoadableComponent = Loadable({
  loader: () => import('./my-component'), // 要按需加载的组件，用了import()函数
  loading: Loading,    // 一个无状态组件,负责显示"Loading中"
});

export default class App extends React.Component {
  render() {
    return <LoadableComponent/>;
  }
}
```
#### my-loading-component.js
```
import React from 'react';

export default function Loading(props) {
  if (props.isLoading) {
    if (props.timedOut) {
      return <div>Loader timed out!</div>;
    } else if (props.pastDelay) {
      return <div>Loading...</div>;
    } else {
      return null;
    }
  } else if (props.error) {
    return <div>Error! Component failed to load</div>;
  } else {
    return null;
  }
}
```
## 源码分析
### 源码结构
这是loadable的源码结构
![image](http://cdn.blog.yangzhedi.com/loadable/loadable.png)

最后export出来的是一个`Loadable`函数，所以我们从`Loadable`开始分析。

```
module.exports = Loadable;

function Loadable(opts) {
  return createLoadableComponent(load, opts);
}

```
Loadable接受一个参数`opts`,再调用了`createLoadableComponent`函数，传入了load函数和opts。

#### load函数

```
function load(loader) {
    let promise = loader();

    let state = {
        loading: true,
        loaded: null,
        error: null
    };

    state.promise = promise.then(loaded => {
        state.loading = false;
        state.loaded = loaded;
        return loaded;
    }).catch(err => {
        state.loading = false;
        state.error = err;
        throw err;
    });

    return state;
}
```
这里`load`又是一个函数，接受一个`loader`参数。我们先放在这里，一会在回来看这个`loader`。

#### createLoadableComponent
![createLoadableComponent](http://cdn.blog.yangzhedi.com/loadable/createLoadableComponent.png)

这个函数是整个库的毕竟重要的函数了。（因为本文是浅析，所以先只分析主逻辑的函数，别的健壮性支持函数可能会之后再分析）

##### if(!options.loading){...}
`options.loading`就是上文提到的**一个无状态组件,负责显示"Loading中"**，如果不存在，会报错，需要一个Loading中显示的组件。
##### let opts = Object.assign(...,options)
```
let opts = Object.assign({
        loader: null,
        loading: null,
        delay: 200,
        timeout: null,
        render: render,
        webpack: null,
        modules: null,
    }, options);
```
就是初始化一下opts的值，赋给未传入参数初始值，防止接下来的判断报错。

##### function init() {...}
```
function init() {
    if (!res) {
        res = loadFn(opts.loader);
    }
    return res.promise;
}
```
`init`之前声明过一个`res`，`init`就是为`res`赋值。在`init`里调用了`loadFn`,就是上文说过的`load函数`。

load函数接收一个参数，就是之前`() => import('./my-component')`，这里的`import()`方法，返回一个Promise对象，`[[PromiseValue]].default`就是待load组件的function。
![image](http://cdn.blog.yangzhedi.com/loadable/import.png)


load函数里初始化了一个state对象，并在import()方法返回的Promise中，对state的属性赋值，`loading`代表是否加载完成， `loaded`为加载的对象，这里的`state.loaded`已经是`[[PromiseValue]]`了。


##### return class LoadableComponent extends React.Component ...

因为react loadable的描述终究是一个高阶组件，如果对高阶组件不了解的，可以去看
[深入React高阶组件(HOC)](https://juejin.im/post/5adddc57f265da0b8635de56)这篇文章。
所以`createLoadableComponent`最后返回的是一个React Component。

一开始的`constructor`里，调用了init，将res的几个属性，分别赋值给this.state作为组件初始化。
```
this.state = {
    error: res.error,
    pastDelay: false,
    timedOut: false,
    loading: res.loading,
    loaded: res.loaded
};
```
之后在`componentWillMount`中，做了这些操作
- 设置了个标记位`this._mounted`，默认为true
- 进行一些判断，如果`res.loading`为true，说明之前的init执行出错，直接return；
- 如果`opts.delay`和`opts.timeout`有值，且为number属性的话，就加个定时器。
- 声明update函数，如果`this._mounted`为false，重新将res的属性更新到state里。

`render`中进行了判断，
- 如果`this.state.loading`或者`this.state.error`为true，就是整体状态是正在加载ing或者出错了，就用`React.createElement`生成出loading的过渡组件，
- 如果`this.state.loaded`有值了，说明传入的`loader`的promise异步操作执行完成，就开始渲染真正的组件，调用`opts.render`方法.(此时的this.state.loaded就是`[[PromiseValue]]`)
  ![image](http://cdn.blog.yangzhedi.com/loadable/import2.png)

```
function resolve(obj) {
    return obj && obj.__esModule ? obj.default : obj;
}
function render(loaded, props) {
    return React.createElement(resolve(loaded), props);
}
```
render的时候进行判断，当__esModule为true的时候，标识module为es模块，那么obj默认返回obj.default，那么obj默认返回obj。之后再调用`React.createElement`进行渲染。

至于props是什么？在项目里打印发现，就是原组件的props。所以render出的，就和正常在代码中写的React Component是一样的。
![image](http://cdn.blog.yangzhedi.com/loadable/props.png)
然后，整个loadable就完成了。

## 总结
这个库的设计还是非常巧妙，利用import返回一个promise对象的性质，进行loading的异步操作，有点像图片懒加载的原理。下面是划重点系列，
- import() 可以返回一个promise对象
- React.createElement()  API
- Promise对象原理。


本文只是对`react loadable`这个库的核心源代码的分析，还有一些别的代码没有分析到，以及一些分析失误的地方，都欢迎大家交流分享。可以发邮件给我：uiryzd@163.com



[react loadable源码地址](https://github.com/jamiebuilds/react-loadable/blob/master/src/index.js)
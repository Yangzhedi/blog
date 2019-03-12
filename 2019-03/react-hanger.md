## 前言
千呼万唤始出来，React Hooks终于在React 16.8版本中发布稳定版了。最近逛github发现了一个很有意思的库：[react-hanger](https://github.com/kitze/react-hanger)。

## 复习React Hooks
如果对Hooks还不怎么了解的同学，建议去看一下官方文档：[Introducing Hooks](https://reactjs.org/docs/hooks-intro.html).
### 什么是 Hooks?
我们都知道，在Hooks之前，开发react组件主要是class组件和function组件。function组件没有state，所以也叫SFC（stateless functional component），简单的将props映射成view；class组件有state，能够处理更加复杂的逻辑。但是基于class的组建并不是完美的，总结起来就像Dan说的那样，有三个主要的问题：

1. 代码重用：在hooks出来之前，常见的代码重用方式是HOCs和render props，这两种方式带来的问题是：你需要解构自己的组件，非常的笨重，同时会带来很深的组件嵌套
2. 复杂的组件逻辑：在class组件中，有许多的lifecycle 函数，你需要在各个函数的里面去做对应的事情。这种方式带来的痛点是：逻辑分散在各处，开发者去维护这些代码会分散自己的精力，理解代码逻辑也很吃力
3. class组件的困惑：对于初学者来说，需要理解class组件里面的this是比较吃力的(这个理由有点勉强~)，同时，基于class的组件难以优化(举个不恰当的例子，看一下babel转移出来的class代码量增长了多少)

为了解决上面的这三个问题，react hooks提案登场了，它有以下几个特点：

1. 无痛接入，不破坏现有的项目结构
2. 完全向后兼容，不包含任何不兼容breaking changes
3. 现在就能使用

Hooks 允许你在不编写 class 的情况下使用状态(state)和其他 React 特性。 你还可以**构建自己的 Hooks**, 跨组件共享可重用的有状态逻辑。

现在React中内置的Hooks有：

- [Basic Hooks](https://zh-hans.reactjs.org/docs/hooks-reference.html#basic-hooks)
  - [`useState`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usestate)
  - [`useEffect`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useeffect)
  - [`useContext`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecontext)
- [Additional Hooks](https://zh-hans.reactjs.org/docs/hooks-reference.html#additional-hooks)
  - [`useReducer`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usereducer)
  - [`useCallback`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecallback)
  - [`useMemo`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usememo)
  - [`useRef`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useref)
  - [`useImperativeHandle`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useimperativehandle)
  - [`useLayoutEffect`](https://zh-hans.reactjs.org/docs/hooks-reference.html#uselayouteffect)
  - [`useDebugValue`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usedebugvalue)

当然了，授之以鱼不如授之以渔，React官方也提供了教你如何封装自己Hook的文档[Building Your Own Hooks](https://zh-hans.reactjs.org/docs/hooks-custom.html)，有兴趣的小伙伴可以去阅读一下。

## react-hanger初窥
大致的看了下react-hanger的源码之后发现，这个库其实是对React Hooks API的适用性封装。暴露一些更常用的Hooks节省大家造轮子的工作量。

React的核心开发者Dan看到这个库也做了评价: 
> 一个对Hooks的隐喻。你可以将你的state“挂起”在你的function component上，等你回来的时候，它就挂在那。

![](https://camo.githubusercontent.com/361fd581dec4b146de3d0dfbb0e5b918b752a210/68747470733a2f2f692e696d6775722e636f6d2f4a6f42574a78532e706e67)

本文写作时，react-hanger的Usage里提供了6个API，从名字里就可以看出这些Hook都是做什么的(Hooks都以"use"开头，这是一种约定)，

```
import {
  useInput,
  useBoolean,
  useNumber,
  useArray,
  useOnMount,
  useOnUnmount
} from "react-hanger";
```

使用起来也很简单，比如`useNumber`

```
const App = () => {
  const showCounter = useBoolean(true);
  const counter = useNumber(0);

  return (
    <div>
      <button onClick={counter.increase}> increase </button>
      {showCounter.value && <span> {counter.value} </span>}
      <button onClick={counter.decrease}> decrease </button>
    </div>
  );
};
```

初步印象：大致与原始的basic hooks有点不同的是，useState返回一个数组，分别是`值`与`操作`，而react-hanger提供的API貌似是将`值`和`一些操作`封装到一个对象中，比如`counter`就是一个`{value: count, increase: setCount(count + 1), decrease: setCount(count - 1) }`的对象。

还有更多的操作方法可以看react-hanger的sandbox：https://codesandbox.io/s/44m70xm70

## react-hanger源码浅析

其实翻看了react-hanger的源码之后会发现，react-hanger一共引用了四个React内置的Hook，

```
import { useCallback, useEffect, useRef, useState } from "react";
```

然后返回一些“轮子”hooks，包括`useNumber`、`useArray`、`useBoolean`等等。

这些轮子可以大致分为两类：封装Hook和拆分Hook。

### 封装Hook

比如`useStateful`、`useNumber`、`useArray`、`useBoolean`都是对内置Hook`useState`的封装。

#### useStateful

```
export const useStateful = initial => {
  const [value, setValue] = useState(initial);
  return {
    value,
    setValue
  };
};
```

利用ES6的解构赋值，将`useState`返回的数组封装成一个对象重新返回，方便调用。

#### useNumber

```
export const useNumber = (
  initial,
  { upperLimit, lowerLimit, loop, step = 1 } = {}
) => {
  const [value, setValue] = useState(initial);
  return {
    value,
    setValue,
    increase: useCallback(i => {
      setValue(...);
    }, []),
    decrease: useCallback(d => {
      setValue(...);
    }, [])
  };
};
```

`useNumber`接收一个initial number和一个配置项对象，在内部是通过对initial number进行useState Hook，返回一个对象，除了基本的`value`和`setValue`，还有两个方法`increase`和`decrease`。这两个方法都是用`useCallback`对`setValue`进行的进一步封装。

> 而`useCallback`是一个比较重要的内置Hook，`useCallback` 的可以于缓存了每次渲染时 inline callback 的实例，在第二个参数数组内的值发生更改时才会更改。
>
> 这样可以配合上子组件的 `shouldComponentUpdate` 或者 `useMemo` 起到减少不必要的渲染的作用。

而第二个参数为空数组的意思就是告诉React不管参数如何都要记忆。

#### useArray & useBoolean & useInput

至于`useArray` 、`useBoolean` 、 `useInput`这三个hook可以说和`useNumber`大同小异，都是需要一个传入的initial值，在hook内部通过`useState`初始化，再返回一些常用的操作方法。

这里的`useInput`是针对于受控组件，所以不需要`useRef`。

#### useSetState

```
export const useSetState = initialValue => {
  const { value, setValue } = useStateful(initialValue);
  return {
    setState: useCallback(v => {
      return setValue(oldValue => ({
        ...oldValue,
        ...(typeof v === "function" ? v(oldValue) : v)
      }));
    }, []),
    state: value
  };
};
```

> Unlike the `setState` method found in class components, `useState` does not automatically merge update objects. 
>
> 与类组件中的setState方法不同，useState不会自动合并更新对象。 

熟悉React Hook的同学看了代码就知道这个hook是封装了什么了，因为useState返回的类似于`setCount`的方法不会自动合并更新对象。这个hook帮助大家可以获得一个可以merge之前value的Hook型`setState`。

### 拆分Hook

上述几个算是封装hook，那么下面的几个就可以算是拆分hook，对`useEffect`更精细化的处理。

#### useOnMount & useOnUnmount

众所周知，`useEffect`是被用来处理一些原先放在class组件中生命周期函数的副作用，比如`componentDidMount`、`componentDidUpdate`、`componentWillUnmount`，集合而成的一个Hook。

理论上，在每次渲染后都会触发`useEffect`的效果，但是如果我只想在didmount里或者只想在willunmount里做一下事情，该怎么办？

这时就用到了`useEffect`的一个特点：第二个参数为效果依赖的值数组，也就是说只有当数组内的值变化才会触发`useEffect`，

```
useEffect(
  () => {
    const subscription = props.source.subscribe();
    return () => {
      subscription.unsubscribe();
    };
  },
  [props.source],
);
```

而如果第二个参数为一个空数组的时候，则相当于告诉React你的效果**不依赖于组件中的任何值**，因此该效果只能在mount上运行并在unmount上清理，**它不会在更新时运行**。

```
export const useOnUnmount = onUnmount =>
  useEffect(() => {
    return () => onUnmount && onUnmount();
  }, []);

export const useOnMount = onMount =>
  useEffect(() => {
    onMount && onMount();
  }, []);
```

所以`useOnMount`的实现就非常简单，在`useEffect`内执行onMount函数且第二个参数是`[]`，`useOnUnmount`的实现则是返回onUnmount函数且第二个参数是`[]`。

#### useLifecycleHooks

```
export const useLifecycleHooks = ({ onMount, onUnmount }) =>
  useEffect(() => {
    onMount && onMount();
    return () => onUnmount && onUnmount();
  }, []);
```

`useLifecycleHooks`则是对`useOnUnmount`和`useOnMount`的整合，在`useEffect`的第二个参数为`[]`的情况下，执行onMount和返回onUnmount。

#### useLogger

```
export const useLogger = (name, props) => {
  useLifecycleHooks({
    onMount: () => console.log(`${name} has mounted`),
    onUnmount: () => console.log(`${name} has unmounted`)
  });
  useEffect(() => {
    console.log("Props updated", props);
  });
};
```

`useLogger`算是一个为hook commponent封装的log插件，通过在`useLifecycleHooks`内传入onMount和onUnmount打印日志的函数，之后再通过原生的默认`useEffect`不传递第二个参数来实现在更新过程中打印日志。

#### usePrevious

```
export const usePrevious = value => {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  });
  return ref.current;
};
```

usePrevious则可以获取之前的props或者state，来自于React的[官方文档](https://reactjs.org/docs/hooks-faq.html#how-to-get-the-previous-props-or-state)。

## 总结

其实react-hanger的源代码都很简洁，对源码感兴趣的同学请看：https://github.com/kitze/react-hanger/blob/master/src/index.js，更多的是这个库的实现大部分都是用了React Hook的思想，希望大家可以通过本文加深对React Hook的理解和认识。
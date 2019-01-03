redux的核心概念其实很简单：将需要修改的state都存入到store里，发起一个action用来描述发生了什么，用reducers描述action如何改变state tree 。创建store的时候需要传入reducer，真正能改变store中数据的是store.dispatch API。

### 1.概念
dispatch一个action之后，到达reducer之前，进行一些额外的操作，就需要用到middleware。你可以利用 Redux middleware 来进行日志记录、创建崩溃报告、调用异步接口或者路由等等。

换言之，中间件都是对store.dispatch()的增强

### 2.中间件的用法

```
import { applyMiddleware, createStore } from 'redux';
import thunk from 'redux-thunk';

const store = createStore(
    reducers, 
    applyMiddleware(thunk)
);
```
直接将thunk中间件引入，放在applyMiddleware方法之中，传入createStore方法，就完成了store.dispatch()的功能增强。即可以在reducer中进行一些异步的操作。

### 3.applyMiddleware()
其实applyMiddleware就是Redux的一个原生方法，将所有中间件组成一个数组，依次执行。
中间件多了可以当做参数依次传进去
```
const store = createStore(
    reducers, 
    applyMiddleware(thunk, logger)
);
```
如果想了解它的演化过程可以去redux的官方文档：https://redux.js.org/advanced/middleware

### 4.redux-thunk
分析redux-thunk的源码`node_modules/redux-thunk/src/index.js`

```
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```

redux-thunk中间件`export default`的就是`createThunkMiddleware()`过的thunk，再看
`createThunkMiddleware`这个函数，返回的是一个柯里化过的函数。我们再翻译成ES5的代码容易看一点，
```
function createThunkMiddleware(extraArgument) {
    return function({ dispatch, getState }) {
        return function(next){
            return function(action){
                if (typeof action === 'function') {
                    return action(dispatch, getState, extraArgument);
                }

                return next(action);
            };
        }
    }
}
```
这么一看，就可以看出来redux-thunk最重要的思想，就是可以接受一个返回函数的action creator。如果这个action creator 返回的是一个函数，就执行它，如果不是，就按照原来的next(action)执行。

正因为这个action creator可以返回一个函数，那么就可以在这个函数中执行一些异步的操作。
例如：
```
export function addCount() {
    return {type: ADD_COUNT}
}

export function addCountAsync() {
   return dispatch => {
       setTimeout( () => {
           dispatch(addCount())
        },2000)
    }
}
```
`addCountAsync`函数就返回了一个函数，将dispatch作为函数的第一个参数传递进去，在函数内进行异步操作就可以了。


参考文档：
http://www.redux.org.cn/docs/advanced/Middleware.html
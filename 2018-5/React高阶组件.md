## 什么是HOC?
HOC(全称Higher-order component)是一种React的进阶使用方法，主要还是为了便于组件的复用。HOC就是一个方法，获取一个组件，返回一个更高级的组件。

## 什么时候使用HOC?
在React开发过程中，发现有很多情况下，组件需要被"增强"，比如说给组件添加或者修改一些特定的props，一些权限的管理，或者一些其他的优化之类的。而如果这个功能是针对多个组件的，同时每一个组件都写一套相同的代码，明显显得不是很明智，所以就可以考虑使用HOC。

栗子：react-redux的connect方法就是一个HOC，他获取wrappedComponent，在connect中给wrappedComponent添加需要的props。

## HOC的简单实现
HOC不仅仅是一个方法，确切说应该是一个组件工厂，获取低阶组件，生成高阶组件。

一个最简单的HOC实现是这个样子的：
```
function HOCFactory(WrappedComponent) {
  return class HOC extends React.Component {
    render(){
      return <WrappedComponent {...this.props} />
    }
  }
}
```
## HOC可以做什么？
- 代码复用，代码模块化
- 增删改props
- 渲染劫持

其实，除了代码复用和模块化，HOC做的其实就是劫持，由于传入的wrappedComponent是作为一个child进行渲染的，上级传入的props都是直接传给HOC的，所以HOC组件拥有很大的权限去修改props和控制渲染。

### 增删改props
可以通过对传入的props进行修改，或者添加新的props来达到增删改props的效果。

比如你想要给wrappedComponent增加一个props，可以这么搞：
```
function control(wrappedComponent) {
  return class Control extends React.Component {
    render(){
      let props = {
        ...this.props,
        message: "You are under control"
      };
      return <wrappedComponent {...props} />
    }
  }
}
```
这样，你就可以在你的组件中使用message这个props:
```
class MyComponent extends React.Component {
  render(){
    return <div>{this.props.message}</div>
  }
}

export default control(MyComponent);
```
### 渲染劫持
这里的渲染劫持并不是你能控制它渲染的细节，而是控制是否去渲染。由于细节属于组件内部的render方法控制，所以你无法控制渲染细节。

比如，组件要在data没有加载完的时候，现实loading...，就可以这么写：

```
function loading(wrappedComponent) {
  return class Loading extends React.Component {
    render(){
      if(!this.props.data) {
        return <div>loading...</div>
      }
      return <wrappedComponent {...props} />
    }
  }
}
```
这个样子，在父级没有传入data的时候，这一块儿就只会显示loading...,不会显示组件的具体内容

```
class MyComponent extends React.Component {
  render(){
    return <div>{this.props.data}</div>
  }
}

export default control(MyComponent);
```

## HOC有什么用例？

### React Redux
最经典的就是React Redux的connect方法(具体在[connectAdvanced](https://github.com/reactjs/react-redux/blob/master/src/components/connectAdvanced.js)中实现)。

通过这个HOC方法，监听redux store，然后把下级组件需要的state(通过mapStateToProps获取)和action creator(通过mapDispatchToProps获取)绑定到wrappedComponent的props上。

### logger和debugger
这个是官网上的一个[示例](https://reactjs.org/docs/higher-order-components.html#dont-mutate-the-original-component.-use-composition.)，可以用来监控父级组件传入的props的改变:
```
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentWillReceiveProps(nextProps) {
      console.log(`WrappedComponent: ${WrappedComponent.displayName}, Current props: `, this.props);
      console.log(`WrappedComponent: ${WrappedComponent.displayName}, Next props: `, nextProps);
    }
    render() {
      // Wraps the input component in a container, without mutating it. Good!
      return <WrappedComponent {...this.props} />;
    }
  }
}
```

### 页面权限管理
可以通过HOC对组件进行包裹，当跳转到当前页面的时候，检查用户是否含有对应的权限。如果有的话，渲染页面。如果没有的话，跳转到其他页面(比如无权限页面，或者登陆页面)。

也可以给当前组件提供权限的API，页面内部也可以进行权限的逻辑判断。

## 使用HOC需要注意什么？

尽量不要随意修改下级组件需要的props
之所以这么说，是因为修改父级传给下级的props是有一定风险的，可能会造成下级组件发生错误。比如，原本需要一个name的props，但是在HOC中给删掉了，那么下级组件或许就无法正常渲染，甚至报错。

### Ref无法获取你想要的ref
以前你在父组件中使用`<component ref="component"/>`的时候，你可以直接通过`this.refs.component`进行获取。但是因为这里的component经过HOC的封装，已经是HOC里面的那个component了，所以你无法获取你想要的那个`ref(wrappedComponent的ref)`。

要解决这个问题，这里有两个方法：

**a)** 像React Redux的connect方法一样，在里面添加一个参数，比如`withRef`，组件中检查到这个flag了，就给下级组件添加一个ref，并通过getWrappedInstance方法获取。

栗子：
```
function HOCFactory(wrappedComponent) {
  return class HOC extends React.Component {
    getWrappedInstance = ()=>{
      if(this.props.widthRef) {
        return this.wrappedInstance;
      }
    }

    setWrappedInstance = (ref)=>{
      this.wrappedInstance = ref;
    }

    render(){
      let props = {
        ...this.props
      };

      if(this.props.withRef) {
        props.ref = this.setWrappedInstance;
      }

      return <wrappedComponent {...props} />
    }
  }
}

export default HOCFactory(MyComponent);
```
这样子你就可以在父组件中这样获取MyComponent的ref值了。
```
class ParentCompoent extends React.Component {
  doSomethingWithMyComponent(){
    let instance = this.refs.child.getWrappedInstance();
    // ....
  }

  render(){
    return <MyComponent ref="child" withRef />
  }
}
```
**b)** 还有一种方法，在官网中有提到过：
父级通过传递一个方法，来获取ref，具体看栗子：

先看父级组件：

```
class ParentCompoent extends React.Component {
  getInstance = (ref)=>{
    this.wrappedInstance = ref;
  }

  render(){
    return <MyComponent getInstance={this.getInstance} />
  }
}
```
HOC里面把getInstance方法当作ref的方法传入就好
```
function HOCFactory(wrappedComponent) {
  return class HOC extends React.Component {
    render(){
      let props = {
        ...this.props
      };

      if(typeof this.props.getInstance === "function") {
        props.ref = this.props.getInstance;
      }

      return <wrappedComponent {...props} />
    }
  }
}

export default HOCFactory(MyComponent);
```
### Component上面绑定的Static方法会丢失
比如，你原来在Component上面绑定了一些static方法`MyComponent.staticMethod = o=>o`。但是由于经过HOC的包裹，父级组件拿到的已经不是原来的组件了，所以当然无法获取到staticMethod方法了。

官网上的示例：
```
// 定义一个static方法
WrappedComponent.staticMethod = function() {/*...*/}
// 利用HOC包裹
const EnhancedComponent = enhance(WrappedComponent);

// 返回的方法无法获取到staticMethod
typeof EnhancedComponent.staticMethod === 'undefined' // true
```
这里有一个解决方法，就是hoist-non-react-statics组件，这个组件会自动把所有绑定在对象上的非React方法都绑定到新的对象上：
```
import hoistNonReactStatic from 'hoist-non-react-statics';
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  hoistNonReactStatic(Enhance, WrappedComponent);
  return Enhance;
}
```

## 结束语
当你需要做React插件的时候，HOC模型是一个很实用的模型。

希望这篇文章能帮你对HOC有一个大概的了解和启发。

另外，这篇[medium上的文章](https://medium.com/@franleplant/react-higher-order-components-in-depth-cf9032ee6c3e#.wwp0tbukh)会给你更多的启发，在这篇文章中，我这里讲的被分为Props Proxy HOC，还有另外一种Inheritance Inversion HOC，强烈推荐看一看。
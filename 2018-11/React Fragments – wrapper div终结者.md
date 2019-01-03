![image](http://cdn.blog.yangzhedi.com/ragments/react-fragments.png)
原文：https://getstream.io/blog/react-fragments/

hi，大家好，我是Ken，目前就职于[GetStream.io](https://getstream.io/)，为用户提供个性化可定制的feed流服务。

在过去的几个月里，我一直在开发Winds 2.0，一个开源的RSS阅读器和博客订阅应用。用Node.js, Electron, Redux and React开发，并且截止发文（2018-7-11），在Github上已经拥有了超过5000个star。如果你想看看整个项目，可以访问[https://getstream.io/winds/](https://getstream.io/winds/)，或者查看[https://github.com/GetStream/winds](https://github.com/GetStream/winds)上的源代码。

在Winds中，我们在一些特定的情况需要用到**React Fragments**。React Fragments是一个去年年底React v16.2.0发布的简洁小功能——它真的非常小，但是只要你了解了它，就可以在遇到一些非常具体的布局和样式情况时避开很多的麻烦。

## Okay, 什么是React Fragment?
让我们稍微回顾一下 - 我确信每个React开发人员都会在他们职业生涯的某个阶段遇到这种情况（或者即将遇到）：

```
class App extends React.Component {
    render() {
        return (
            <p>I would</p>
            <p>really like</p>
            <p>to render</p>
            <p>an array</p>
        );
    }
}
```
看起来挺不错，但是当我们通过JSX转换器 运行它时......

```
  Failed to compile.

    ./src/App.js
    Syntax error: Adjacent JSX elements must be wrapped in an enclosing tag (6:8)

        4 |         return (<p>I would
        5 |         </p>
        6 |         <p>
          |         ^
        7 |             really like
        8 |         </p>
        9 |         <p>
```
JSX转换器 并不喜欢这样 🙁
> **这背后究竟发生了什么？** JSX 把所有的`<div>`s和`<MyComponent>`s都放进`React.createElement()`中调用——当JSX转换器看到的（最外层如果）是多个元素而不是单个元素时，它就不知道到底要渲染哪个tag，更多：[React.createElement in the React documentation](https://reactjs.org/docs/react-api.html#createelement)。

那么我们该怎么办？每次我们需要将几个元素包装在一起时都会这样做，——Pinky，用一个`div`包裹起来！就像web开发者从div被发明出来一直以来做的那样，在DOM中嵌套另外一个`div`也无伤大雅。

```
class App extends React.Component {
    render() {
        return (
            <div>
                <p>I would</p>
                <p>really like</p>
                <p>to render</p>
                <p>an array</p>
            </div>
        );
    }
}
```
嗯，问题确实解决了。但事实证明，还有另一种方法可以在一个React组件里渲染出这一组内容——通过让`render`方法返回一个节点数组。


```
class App extends React.Component {
    render() {
        return [
            <p>I would</p>,
            <p>really like</p>,
            <p>to render</p>,
            <p>an array</p>
        ];
    }
}
```
如果我们返回这样的一个元素数组，React将会转换并渲染它，而且没有`wrapper<div>`（容器div）。 显得很整齐！
> （还记得`JSX`转换器是如何将`<div>`和`<MyComponent>`转换并被`React.createElement()`调用的吗？在这种情况下，转换器只是将这些一个数组内的元素作为子元素直接附加到父元素，而不是当做没有父元素包含的元素数组。React v16.0.0引入了此功能。

因为这样，Dan Abramov和一些React团队的聪明人看到之后说，

“Okay，所以你可以用两种不同的方式来渲染一个元素数组 - 通过在DOM中引入一个额外的`<div>`，或者使用一些笨重的非JSX语法。这些都无法带来一个良好的编程体验。”

所以，在v16.2.0，[React发布了React Fragments](https://getstream.io/blog/react-fragments/)

## okay,**now** what’s a React Fragment?
这才是是使用React Fragment的正确方法：
```
class App extends React.Component {
    render() {
        return (
            <React.Fragment>
                <p>I would</p>
                <p>really like</p>
                <p>to render</p>
                <p>an array</p>
            </React.Fragment>
        );
    }
}
```
看 - 我们写的就像`wrapper<div>` 的方式一样，但在功能上与`数组渲染`方式相同，只不过是使用了看起来不错的JSX语法。 这样就可以将这些p元素呈现为数组，而不包含任何类型的`wrapper<div>`。
> 使用`React Fragments`还有另一种更简洁的语法：
```
class App extends React.Component {
    render() {
        return (
            <>
                <p>I would</p>
                <p>really like</p>
                <p>to render</p>
                <p>an array</p>
            </>
        );
    }
}
```
> 可能在你的工具，linters，构建流程等等中，还没有起作用。但是发版说明中提到，这种语法的支持已经在开发中了，但是我注意到在`create-react-app`中还没有支持。
### Okay,但什么时候才能使用它们呢？
#### 在你需要摆脱 `wrapper<div>`的时候。
就是这样 - 如果你发现自己处于一个`wrapper<div>`搞乱你的React组件布局的情况下，请使用React Fragment。

所以，就是当你想转换这个：
```
<div class="app">

    (...a bunch of other elements)

    <div> (my react component)
        <ComponentA></ComponentA>
        <ComponentB></ComponentB>
        <ComponentC></ComponentC>
    </div>
    
    (...a bunch more elements)

</div>
```
到这样：
```
<div class="app">

    (...a bunch of other elements)

    <ComponentA></ComponentA>
    <ComponentB></ComponentB>
    <ComponentC></ComponentC>
    
    (...a bunch more elements)

</div>
```
## Example: 2×2 CSS grid
在Winds 2.0中，我们大量的使用了CSS Grid布局，这是博客或者RSS源网站很常见的布局。
![http://cdn.blog.yangzhedi.com/fragments/grid.png](http://cdn.blog.yangzhedi.com/fragments/grid.png)
如果您还不了解CSS Grid布局，下面的代码可以让您快速了解CSS的布局

```
.grid {
    display: grid;
    grid-template-areas: 
        'topnav header' 
        'subnav content';
    grid-gap: 1em;
}
```
Okay，让我们来解释一下：
- 在左上角，我们有标志或者顶级导航位。
- 在左下角，有子导航，可以响应全局和局部状态的一些变化，如“高亮”状态，tabs或折叠导航。
- 在右侧，有一些需要在屏幕上显示的内容，在Winds中，有点类似于与文章列表或文章内容配对的RSS提要或文章标题。 这两个部分将是单一的React组件 - 两个组件的道具都会根据URL导航进行更改。

所有这些作用于全局的组件（redux + URL）和本地状态的交互都略有不同。 这个视图的结构使得我们有三个React组件互为兄弟：
```
<div className="grid">
    <TopNav />
    <SubNav />
    <ContentComponent />
</div>
```
但是，我们希望实际呈现给页面的有四个元素：
```
<div class="grid">
    <div class="topnav"  />
    <div class="subnav"  />
    <div class="header"  />
    <div class="content" />
</div>
```
这......就是没有React Fragments的问题。想象一下，我们正在创建2×2网格视图的两个右侧部分的组件，即`ContentComponent`：
```
class ContentComponent extends React.Component {
    render() {
        return (
            <div>
                <div className="header"/>
                <div className="content"/>
            </div>
        );
    }
}
```
如果我们将渲染的内容包装在`<div>`中，那么我们将获得以下渲染输出：
```
<div class="grid">
    <div class="topnav"  />
    <div class="subnav"  />
    <div>
        <div class="header"  />
        <div class="content" />
    </div>
</div>
```
这不起作用 - 它还会搞乱CSS的grid布局。 从浏览器的角度来看，网格中只有3个项目，其中一个还没有`grid-area`的样式。

还记得我们应该使用React Fragments吗？ 每当我们想要摆脱`wrapper<div>`。 如果我们将`ContentComponent`包装在React Fragments中而不是`wrapper<div>`中：
```
class ContentComponent extends React.Component {
    render() {
        return (
            <React.Fragment>
                <div className="header"/>
                <div className="content"/>
            </React.Fragment>
        );
    }
}
```
然后我们将看到另外一种渲染输出：
```
<div class="grid">
    <div class="topnav"  />
    <div class="subnav"  />
    <div class="header"  />
    <div class="content" />
</div>
```
这就符合我们的预期了！ 没有`wrapper<div>`，我们的4个元素从3个React组件渲染，浏览器看到所有元素具有正确的`grid area`样式，并且我们的CSS grid正确呈现。
### 很整齐！ 现在怎么办？
React Fragments不是近期React版本中发布的最重要的功能，但它们在某些特定情况下非常有用。 只要了解React Fragments的存在，您就可以节省数小时的谷歌引起的头痛。这可以让我们以JSX-y方式呈现元素或者组件数组，这可以解决大量的表格、列表和CSSgrid的布局和样式问题！
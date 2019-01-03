最近在网上也看到了react-router4的好多种按需加载的方法。

传送门：https://blog.csdn.net/foralienzhou/article/details/73437057

虽然自己的项目不大，但是也要区分前台和后台，如果让访问前台的用户也加载了后台的js代码，还是很影响体验的，所以挑了一种按需加载的方法进行实践（基于create-react-app和Bundle组件）。


### import()
这里的import不同于模块引入时的import，可以理解为一个动态加载的模块的函数(function-like)，传入其中的参数就是相应的模块。例如对于原有的模块引入import react from 'react'可以写为import('react')。但是需要注意的是，import()会返回一个**Promise对象**。因此，可以通过如下方式使用：
```
btn.addEventListener('click', e => {
    // 在这里加载chat组件相关资源 chat.js
    import('/components/chart').then(mod => {
        someOperate(mod);
    });
});
```
可以看到，使用方式非常简单，和平时我们使用的Promise并没有区别。当然，也可以再加入一些异常处理：
```
btn.addEventListener('click', e => {
    import('/components/chart').then(mod => {
        someOperate(mod);
    }).catch(err ＝> {
        console.log('failed');
    });
});
```


我们首先需要一个异步加载的包装组件Bundle。Bundle的主要功能就是接收一个组件异步加载的方法，并返回相应的react组件。
```
import React from 'react';

export default class Bundle extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            mod: null
        };
    }

    componentWillMount() {
        this.load(this.props)
    }

    componentWillReceiveProps(nextProps) {
        if (nextProps.load !== this.props.load) {
            this.load(nextProps)
        }
    }

    load(props) {
        this.setState({
            mod: null
        });
        props.load().then((mod) => {
            this.setState({
                mod: mod.default ? mod.default : mod
            });
        });
    }

    render() {
        return this.state.mod ? this.props.children(this.state.mod) : null;
    }
}

```
引入模块的时候需要用Bundle组件包一下
```
import Bundle from './Bundle'
const Dashboard = (props) => (
    <Bundle load={() => import('./Dashboard')}>
        {(Dashboard) => <Dashboard {...props}/>}
    </Bundle>
);
```

路由部分没有变化
```
<HashRouter>
    <Switch>
        <Route path='/' exact component={Index} />
        <Route path='/dashboard' component={Dashboard} />
    </Switch>
</Router>
```

这时候，执行npm start，可以看到在载入最初的页面时加载的资源如下
![image](http://cdn.blog.yangzhedi.com/code-splitting/1.png)

而当点击触发到/dashboard路径时，可以看到
![image](http://cdn.blog.yangzhedi.com/code-splitting/2.png)


代码拆分在单页应用中非常常见，对于提高单页应用的性能与体验具有一定的帮助。按需加载的方式还不止这一种，还可以使用require.ensure()或者一些loader也可以同样实现这个功能。


如果加载的js很大，或者用户的网络状况不好的话，需要加上一个loading的效果，这里我用的是antd的Spin组件。在render函数的mod没set的时候加上就可以了。
```
render() {
    let spin = <div style={{textAlign: 'center', marginTop:50}}><Spin size="large"/><br/>正在玩命加载中。。。</div>;
    return  this.state.mod ? this.props.children(this.state.mod) : spin;
}
```
![image](http://cdn.blog.yangzhedi.com/code-splitting/3.gif)
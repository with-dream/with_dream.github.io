### 一、基础
#### 1.1、项目结构
创建命令 `create-react-app` 项目名
`npm start` 启动项目

```
├── README.md							使用方法的文档
├── node_modules					所有的依赖安装的目录
├── package-lock.json			锁定安装时的包的版本号,保证团队的依赖能保证一致。
├── package.json
├── public								静态公共目录
└── src										开发用的源代码目录

```

#### 1.2、jsx
jsx = js + xml
jsx是React.createElement的语法糖

##### 创建
```js
import React from "react";
import reactDom from "react-dom";

//函数式组件
function Test1(props) {
    return <div>
        props.title
    </div>;
}

//类创建
class Test2 extends React.Component {
    constructor(props) {
        super(props)
		//state状态值
        this.state = {
            n:0
        }
    }

    render() {
        let {title} = this.props

        return <div>
            <button className={'btn'} onClick={}>{this.state.n}</button>
        </div>;
    }

    //按钮的回调函数
    click = ev => {
        this.setState({n:this.state.n + 1})
    }
}

//限定Test2中props中title的数据类型
Test2.prototype = {
    title:PropTypes.string
};


//调用
reactDom.render(<div>
    <Test1 title={'test1'}/>
    <Test2 title={'test1'}/>
</div>)
```

#### 1.3、重要属性
##### this.props.children

在自定义的<Child>组件中 this.props.children代表自定义组件中的内容
即`<span>{'==>app'}</span>`

```js
<Child>
  <span>{'==>app'}</span>
</Child>
```

##### PropTypes类型检查
限定props变量的类型

```js
import PropTypes from 'prop-types';
```

##### ref真是节点
```js
<input type="text" ref="myTextInput" />

handleClick: function() {
	this.refs.myTextInput.focus();
}
```

#### 1.4、生命周期
props、state的改变

![](./react_生命周期.webp)

#### 1.4、组件间通信
##### 父传子
通过子组件的props

##### 子传父
通过子组件的props传递回调

#### 1.5、router
`npm install -S react-router-dom`

##### route

```js
import { HashRouter, Redirect, Route } from 'react-router-dom'
import Home from './Home';

ReactDOM.render((
  //根路由
  <HashRouter>
  	//具体路由
    <Route path="/home" component={Home}/>
    <Route path="/a" component={A}/>
    <Route path="/b" component={B}/>
    //重定向路由 to指向Route的path
    <Redirect from="/" to="/home" exact />
  </HashRouter>
),
  document.getElementById('root')
);
```

##### Link
用于组件跳转 替代`<a>`标签
```js
import { Link } from 'react-router-dom'

<div>
	//to指向Route的path
    <Link to='/a'>AAA</Link>
</div>
```

##### 路径通配符
`:naem`  匹配一部分
`()`  可选
`*`  任意字符 非贪婪
`**`  任意字符 贪婪

### 二、Redux
store全局唯一 可以作为数据分发中心

```js
import { createStore } from 'redux';
//回调 接收数据
function callback(state, action) {
  switch(action.type) {
    case 'type':
      console.log("cb==>"+JSON.stringify(action.data))
    break
  }
}
//创建store
let store = createStore(callback);
//发送事件及数据 action中的数据可以是任何类型
store.dispatch({type: 'type',  data:{d:"test"}});
//store中数据有变化 即回调
store.subscribe(listener);
```

##### applyMiddlewares()
拦截器功能
```js
const store = createStore(
  reducer,
  initial_state,
  applyMiddleware(logger)
);
```

#### 2、样式
行内样式/模块化

```js
import common from "./common.module.css"

styleObj:{
	fontSize:'50px'
}


//行内样式
<h1 style={{'fontSize':'20px'}}>正在热映</h1>
//class样式
<h1 style={this.styleObj}>即将上映</h1>
//可以直接引用模块化样式
<h1 className={common.error}>异常信息</h1>
```
### 三、补充


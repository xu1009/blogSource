---
title: ReactJs之路
date: 2018-01-17 22:36:17
tags: 前端 react
---



* 包管理器 npm
* 构建器 webpack
* 编译器 babel
JSX 语法

javascrip拓展语言，可以写html可以写js，js用大括号就行。
组件名称必须大写

```javascrit
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```

<Welcome />这个是组件，开头大写的，react遇到组件时，他会将属性作为单个对象传递给该组件，这个对象称为props，function那个是定义的组件，这样sara会被传入组件然后渲染到那个root中。

props是当你引用组件时的一个对象，你可以使用这个对象中的属性，当然这个属性也是你定义过的。

使用es6类就允许我们使用其它特性，例如局部状态，生命周期钩子。


es6 类中使用局部状态就是自己的私有属性，不用再通过props进行传参数了，



```javascrit
<!DOCTYPE html>
<html>
  <head>
    <script src="../build/react.js"></script>
    <script src="../build/react-dom.js"></script>
    <script src="../build/browser.min.js"></script>
  </head>
  <body>
    <div id="example"></div>
    <script type="text/babel">
        function tick() {
            ReactDOM.render(
                    <h1>Hello, world!{new Date().toLocaleTimeString()}</h1>,
                document.getElementById('example')
            );
        }
        setInterval(tick, 1000);
    </script>
  </body>
</html>
```
### react Router
> 他可以让你向应用中快速地添加视图和数据流，同时保持页面与URL间的同步。这个是因为这里只有一个根元素的，就是拿不同组件去渲染，所以前端才会用路由，对于不同url使用不同的组件组合去渲染那个根元素？目前看起来是这样子的，这样做的话就可以实现模块复用，模块化更灵活。其中前端怎么get到url的，这个是通过window这个全局对象get到的。其中hash部分是url的#之后的部分。router是建立在history之上的，他会监听浏览器地址栏的变化，并解析这个url为location对象，然后router使用它匹配到路由，正确的渲染相应的组件。

根DOM节点，里面的一切都是react DOM掌控，
ReactDOM.render()两个参数，第一个是react element，第二个是根节点DOM，这样就能将元素加到这个根节点中，react元素不可变，因为本来就定义成const啊。

arrow functions，简单实用，关键是使用箭头函数不用再手动绑定当前类的this了。

> React是一个采用声明式，高效而且灵活的用来构建用户界面的框架，React当中包含了一些不同的组件，各种模块化，实现高度复用。

单独定义组件，使用es6 class或者使用类似于js的函数function。

JSX是一种很特殊的语法，全部混合到一起了，在js中写html，JSX也使用camelCase命名。
Babel转译器会把JSX转换成一个名为Reac.createElement()的方法调用。


JSX浏览器是不会认识的，这就需要Babel将其转译成js代码。


组件尽量最大可能的模块化，从而达到更高的复用。

props只能像纯函数一样去使用，就是只读，不能修改，但是应用界面随时间变化，这就需要一个新的东西，这就是state。

### state
> State可以在不违反上述规则的情况下，根据用户操作、网络响应、或者其他状态变化，使组件动态的响应并改变组件的输出。

这个可以改变，然后呢这时候定义组件就只能使用es6类来了，因为定义成类的话就可以使用面向对象的一些特性，比如私有属性，生命期钩子。

父组件或子组件都不知道某个组件时有状态还是无状态，并且他们不应该关心某组件是被定义为一个函数还是一个类。一个组件可以将自己的属性传给下一个组件，下一个组件对于这个是无感知的，数据自顶向下。

### 事件处理
> react元素的事件处理和DOM元素的很相似，但是有几点不同，首先react事件绑定属性的命名采用驼峰写法，而不是小写，如果采用JSX的语法你需要传入一个函数作为事件处理函数，而不是一个字符串。

更新状态的时候可以传入函数，这样就可以对传入的值进行二次计算。

回调函数需要绑定this，这个是因为，其实我也没明白，就是react中的和普通的html不太一样，或者js。


有关向事件处理程序传递参数，这里面有个e是合成事件，可以使用e进行事件响应取消，如不跳转。

### 条件渲染
> 在REACT中，你可以创建不同的组件封装你需要的行为，然后还可以根据应用的状态变化只渲染其中的一部分。

可以使用js中的if逻辑判断，
```javascript
  function UserLogin(props) {
            return <h1>welcome!!!!</h1>;
        }
        function UserNotLogin() {
            return <h1>you are not login!!!</h1>;
        }
        function CheckLogin(props) {
            const isLogin = props.isLogin;
            if (isLogin){
                return <UserLogin/>;
            }
            return <UserNotLogin/>;
        }
```

箭头函数中用小括号将函数体括起来可以在调用时返回这个。

### webpack
> 现代js应用程序的模块打包器，递归构建一个依赖关系图，这个关系图包含应用程序需要的每个模块，然后将所有这些模块打包成一个或者多个bundle。

* 入口（entry）
* 输出（output）
* loader
* 插件（plugins）

#### 入口
入口起点（entry point）指示webpack应该使用哪个模块来作为构建其内部依赖图的开始。
```javascript
module.esports = {
    entry: './path/file.js'
};
```
#### 出口
> output属性告诉webpack在哪里输出它所构建的bundles，以及如何命名这些文件，你可以通过在配置中指定一个output字段，来配置。

```javascript
const path = require('path');
module.exports = {
    entry: './path/file.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'my-first-webpack.js'
    }
};
```
分离应用程序（app）和第三方（vendor）入口
```javascript
    const config = {
  entry: {
    app: './src/app.js',
    vendors: './src/vendors.js'
  }
};
```
从入口开始创建依赖图，依赖图完全是分离，相互独立的，只有一个入口起点的单页面应用。

多页面应用
配置文件如下所示
```javascript
const config = {
  entry: {
    pageOne: './src/pageOne/index.js',
    pageTwo: './src/pageTwo/index.js',
    pageThree: './src/pageThree/index.js'
  }
};
```
上面就是配置了三个独立分离的依赖图，多页面就是有多个根元素多个html，切换界面的话就需要重新加载文档，并且资源被重新下载。
#### loader
> 这个能够去处理那些非js文件（webpack本身只能理解js），loader可以将所有类型的文件转换为webpack能够处理的有效模块，然后打包处理。
loader有两个目标，识别出应该被对应的loader进行转换的那些文件。（test属性），转换这些文件，从而使其能够被添加到依赖图中（并且最终添加到bundle中）（use属性）
loader用于对模块的源代码进行转换，loader可以使你在import或加载模块时预处理文件，因此，loader类似于其它构建工具中的task。




首先你要使用npm安装依赖包，比如这里的不同的loader加载不同的格式文件。

使用loader，这里有三种使用loader的方式：
* 配置：在配置文件中指定loader
* 内联：在每个import语句中显示指定loader。
* CLI:在shell命令中指定。

#### mainfest
> 一旦你的应用程序中，形如index.html文件，一些bundle和各种资源加载到浏览器，之前的工程目录结构都不存在了，所以这就需要了mainfest，这是让webpack管理各个模块之间怎么交互。当编译器开始执行，解析和映射应用程序时，他会保留所有模块的详细要点，这个数据集合称为Mainfest，当打包完成并发送到浏览器时，会在运行时通过Mainfest来解析和加载模块。

```javascript
const path = require('path');

const config = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  }
};
// 这个全是使用的正则吧，转换以.txt结尾的文件转换。
module.exports = config;
```
“嘿，webpack 编译器，当你碰到「在 require()/import 语句中被解析为 '.txt' 的路径」时，在你对它打包之前，先使用 raw-loader 转换一下。




#### 插件
> 想要使用一个插件，你只需要require()它，然后把它添加到pulgins数组中，多数插件可以通过选项自定义，你也可以在一个配置文件中因为不同目的而多次使用一个插件。

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 用于访问内置插件
const path = require('path');

const config = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

module.exports = config;
```
### 模块热替换（hot module replacement）
> 模块热替换功能会在应用程序运行过程中替换，添加或删除模块，而无需重新加载整个页面，这就是传说中的热部署，更改完就会自己重新替换更改的部分，然后发布。就是更新webpack-dev-server的配置，使用webpack内置的HMR插件。


### webpack 作用
> 没有webpack之前，每个js都要在html中引用，这个会导致有些没用的什么的都会被下载，效率低下，自己还要把握引入的顺序，顺序不一致就会导致错误，这就需要webpack去处理这些依赖关系，最后打包成若干个bundle，这样的话，在开发环境中就将依赖关系和需要的js文件之类的全部搞定了，这样在线上的时候就会加载的比较快。

依赖是通过npm构建好的，这里也有一个配置文件，省的你去一个个安装，这个是package.json,每次手动安装的时候可以使用--save参数，这样就会在配置文件中写入，下次换个环境，直接npm install就可以实现自动扫描配置文件中的依赖，下载了。都在node_module中，然后自己在js中需要啥就引用啥就行了。

然后就是这些依赖如果有需要自己在js中引用就行了，这就是后来的react组件，最后就是使用webpack处理好这些依赖关系，打包成bundle，这样就行了。

体验了一下全部使用webpack命令打包的感觉，这样，的确不是很方便，这就引入了配置文件，取代CLI(命令接口)，然后那个配置文件呢，只是定义了入口和出口，还要运行很麻烦的webpack指令，这就引入了npm 脚本，在npm配置文件中，定义这个脚本，然后还不用指定依赖的绝对路径。
```javascript
    ./node_modules/.bin/webpack src/index.js dist/bundle.js  //直接使用命令的方式
    
 ./node_modules/.bin/webpack --config webpack.config.js  // 使用配置文件的时候
    
	"build": "webpack"  // 直接 npm run build就行
```

webpack之前只是看的处理依赖的库，现在看看webpack怎么整合其他资源，比如图像。

在js中import css文件，然后给element添加这个样式，这样webpack就会自己打包这个css到js中，然后在线解析的时候将css样式加到head中去，不过现在常使用的是将css进行分离，这样会提高生产环境的加载速度，两个bundle文件是并行加载的。

处理图像文件，同样的方式需要loader，然后

webpack通过运行一个或者多个npm脚本时，会在本地node_modules目录中查找安装的webpack


source map作用是将编译后的代码映射到原始代码，这样就是方便定位错误。

生产环境和开发环境配置文件分开，其中公共部分独立出来，然后merge，这样可以实现复用。

### 创建一个webpack工程的基本步骤

- 首先初始化npm目录结构，npm init -y
- 安装webpack（本地安装开发环境依赖）
- 构建出目录结构src/index.js dist/index.html

webpack打包可以通过CLI的方式，不过这种很麻烦，这就有了配置文件，配置文件就是定义打包的一些参数。

配置文件中有啥呢
* 常用的有入口，出口，loader，plugin等。
* 不同的loader可以让js引用不同类型文件


打包之后生成的bundle可以用HtmlWebpackPlugin插件自动的加入到html中去，就是重新生成了html。

clean-webpack-plugin插件打包之前先清理一下相应目录。

#### 一个开发工具
> 代码变化时需要自动编译代码，这里有几个工具，webpac‘s Watch Mode、webpack-dev-server、webpack-dev-middleware，现在大部分都是使用webpack-dev-server。

其中watch的话就是"watch": "webpack --watch"脚本就行，他会观察当前的文件，如果有变化的话就会重新编译。但是不会自动刷新浏览器界面。

- webpack-dev-server这个比较常见，启动一个server默认在8080端口，详细配置是HMR。
- webpack-dev-middleware

HRM只是为了不全局刷新，局部更新替换某个模块。


### CSS中的Box Model

- margin 边距
- border 边框
- padding 间隙
- content 内容

* 边距属性 (margin) 是用来设置一个元素所占空间的边缘到相邻元素之间的距离
* 边框属性 (border) 用来设定一个元素的边线
* 间隙属性 (padding) 是用来设置元素内容到元素边框的距离
* CSS 背景属性指的是 content 和 padding 区域
* CSS 属性中的 width 和 height 指的是 content 区域的宽和高


* 有关webpack-dev-server的代理，转发dev-server的请求，调试很方便。

### Jquery的deferred对象
> 为了回调函数出现的，链式写法，done，fail，可以多个回调函数，多个事件的同一个回调函数。
用处

### 有关CSS样式优先级问题
> 优先级就是分配给指定的CSS声明的一个权重，它由匹配的选择器中的每一种选择器类型的数值决定。而当优先级与多个css声明中任意一个声明的优先级相等的时候，css中最后的那个声明将会被应用到元素上。当同一个元素有多个声明的时候，优先级才会有意义，因为每一个直接作用于元素的css规则总是会接管或者覆盖改元素从祖先元素继承而来的规则。

#### 选择器类型

* 类型选择器（例如，h1）和伪元素（例如，::before）
* 类选择器（class selectors） (例如,.example)，属性选择器（attributes selectors）（例如, [type="radio"]），伪类（pseudo-classes）（例如, :hover）
* ID选择器（例如, #example）具有更高优先级

给元素添加的内联样式 (例如, style="font-weight:bold") 总会覆盖外部样式表的任何样式 ，因此可看作是具有最高的优先级

Babel是一个广泛使用的ES6转码器，可以将ES6代码转为ES5代码，从而在现有环境执行。

es6中的let声明的变量，只在当前代码块有效。这是和var的不同。

变量提升var输出的是undefined然后let直接抛出错误。


const 增加了常量


### reacjs

> 几个概念说一下，jsx是一种格式，就像xml，js等这些，在这个格式中可以把html和js混合写在一起，是javascript的语法的拓展，这个更像js，所以在里面写的一些html需要变化一些，比如html中的标签属性之类，class变成className，使用小驼峰命名法。es6是一种规范，一种标准，是js的标准或者规范，增加了一些新特性。

事件中如果回调函数使用的是箭头函数的话就可以不用绑定。

jsx只是为React.createElement()方法提供的语法糖。这样在写的时候会方便一些，下面复杂的编译通过babel编译器进行。


### react一些基础常识

其实我也觉得react会很快，总觉得这种局部刷新的，要完爆其它一切框架，但是实际上却不是这样，官方文档说过react尽量达到跟非react版本相当的性能。

这里react组件渲染分为初始化渲染和更新渲染（对应相应的生命周期方法）

首先在初始化渲染的时候回调用根组件下的所有组件的render方法进行渲染，如下图（绿色表示已渲染）

![](https://sfault-image.b0.upaiyun.com/407/896/4078962238-585142242e316_articlex)

但是当我们要更新某个子组件的时候，如下图的绿色组件（从根组件传递下来应用在绿色组件上的数据发生改变）：

![](https://sfault-image.b0.upaiyun.com/168/734/1687340203-5851427977a39_articlex)

我们的理想状态只是调用关键路径上组件的render，如下图：

![](https://sfault-image.b0.upaiyun.com/254/013/2540130692-585142a91eb77_articlex)

但是react默认做法是调用所有组件的render，再对生成的虚拟DOM进行对比，如不变则不进行更新，这样的render和虚拟DOM的对比明显是在浪费，如下图（黄色表示浪费的render和虚拟DOM对比）,这个明显做了很多无用功啊，如果这个树的深度很深，那就会导致性能上巨差。

![](https://sfault-image.b0.upaiyun.com/150/116/1501167720-5851433ff1174_articlex)


> 拆分组件是有利于复用和组件优化的，生成虚拟DOM并进行比对发生在render()后，而不是render()前


#### 更新阶段的生命周期

* componentWillReceiveProps(object nextProps)：当挂载的组件接收到新的props时被调用。此方法应该被用于比较this.props 和 nextProps以用于使用this.setState()执行状态转换。（组件内部数据有变化，使用state，但是在更新阶段又要在props改变的时候改变state，则在这个生命周期里面）
* shouldComponentUpdate(object nextProps, object nextState)： -boolean 当组件决定任何改变是否要更新到DOM时被调用。作为一个优化实现比较this.props 和 nextProps 、this.state 和 nextState ，如果React应该跳过更新，返回false。
* componentWillUpdate(object nextProps, object nextState)：在更新发生前被立即调用。你不能在此调用this.setState()。
* componentDidUpdate(object prevProps, object prevState)： 在更新发生后被立即调用。（可以在DOM更新完之后，做一些收尾的工作）

> React的优化是基于shouldComponentUpdate的，该生命周期默认返回true，所以一旦prop或state有任何变化，都会引起重新render。


所以shouldComponentUpdate就很重要，这里讲一下这个方法，react在每个组件生命周期更新的时候都会调用一个shouldComponentUpdate(nextProps, nextState)函数，它的职责就是返回true或者false，true表示需要更新，false表示不需要，默认返回为true，即便你没有显示的定义shouldComponentUpdate函数。这就不难解释上面发生的资源浪费了（默认都是true所以组件都会更新的）

为了进一步说明问题，我们再引用一张官网的图来解释，如下图（ SCU表示shouldComponentUpdate，绿色表示返回true(需要更新)，红色表示返回false(不需要更新)；vDOMEq表示虚拟DOM比对，绿色表示一致(不需要更新)，红色表示发生改变(需要更新)）：

![](https://sfault-image.b0.upaiyun.com/146/798/1467989449-5851f3865be3b_articlex)

根据渲染流程，首先会判断shouldComponentUpdate(SCU)是否需要更新。如果需要更新，则调用组件的render生成新的虚拟DOM（就是根据这个方法来判断是否更新），然后再与旧的虚拟DOM对比(vDOMEq)，如果对比一致就不更新，如果对比不同，则根据最小粒度改变去更新DOM(这也是一个优点，最小粒度更新)；如果SCU不需要更新，则直接保持不变，同时其子元素也保持不变。然后自己需要控制SCU?


每当组件的props或state改变时，react会重新创建一个虚拟DOM，并拿这个DOM与上一个作对比，如果发现两个虚拟DOM不完全相同，则React就会做reconcile（校正？），把有差异的地方更新到真实的DOM上（这当然是建立在SCU方法返回true的前提下），react中父组件更新默认触发所有子组件更新。

* C1根节点，绿色SCU (true)，表示需要更新，然后vDOMEq红色，表示虚拟DOM不一致，需要更新。
* C2节点，红色SCU (false)，表示不需要更新，所以C4,C5均不再进行检查
* C3节点同C1，需要更新
* C6节点，绿色SCU (true)，表示需要更新，然后vDOMEq红色，表示虚拟DOM不一致，更新DOM。
* C7节点同C2
* C8节点，绿色SCU (true)，表示需要更新，然后vDOMEq绿色，表示虚拟DOM一致，不更新DOM。

#### 一些带坑的写法

* {...this.props} (不要滥用，请只传递component需要的props，传得太多，或者层次传得太深，都会加重shouldComponentUpdate里面的数据比较负担，因此，请慎用spread attributes（<Component {...props} />）)。
* ::this.handleChange()。(请将方法的bind一律置于constructor)
* 复杂的页面不要在一个组件里面写完。
* 请尽量使用const element。
* map里面添加key，并且key不要使用index（可变的）具体可参考[使用Perf工具研究React Key对渲染的影响](http://levy.work/2016-08-31-debug-react-key-with-performance-tool/)
* 尽量少用setTimeOut或不可控的refs、DOM操作。
* props和state的数据尽可能简单明了，扁平化。
* 使用return null而不是CSS的display:none来控制节点的显示隐藏。保证同一时间页面的DOM节点尽可能的少。

#### 一个react优化的点

> react经常会让你给标签加一个key，不然会有警告，这里研究一下这个优化方向。React本身就非常关注性能，其提供的虚拟DOM搭配上Diff算法，实现对DOM操作最小粒度的改变也是非常的高效。然而其组件渲染机制，也决定了在对组件进行更新时还可以进行更细致的优化。

主要包括两个点：
1. 父组件更新默认触发所有子组件更新
2. 列表类型的组件默认更新方式非常复杂









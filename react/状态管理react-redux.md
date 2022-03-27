# React-Redux 基本使用

我们在使用Redux做集中式状态管理的时候会存在一些问题，例如组件无法状态公用，每一个状态组件都在通过订阅来监视，状态更新会影响到全部组件更新，面对这些问题，React官方在redux基础上提出了 `react-redux` 库

## 一、容器组件和UI组件

1.所以的UI组件都需要有一个容器组件包裹

2.容器组件来负责和Redux打交道，可以随意使用Redux的API

3.UI组件无任何Redux API

4.容器组件用于处理逻辑，UI组件只会负责渲染和交互，不处理逻辑

![alt 属性文本](./react-redux模型图.png)

我们在开发过程中，可以直接将UI组件写在容器组件的代码文件中，这样可以避免多个文件

首先，我们在src目录下，创建一个`containers`文件夹，用于存放各种容器组件，在该文件夹内创建`Count`文件夹。这个文件夹中的文件用于即将要使用状态存储的组件

要实现容器组件和UI组件的连接，我们需要借助React-Redux中的`connect`方法来实现

```js
    // 引入组件
    import CountUI from '../../components/Count'
    // 引入连接方法
    import { connect } from 'react-redux'
    // 建立连接
    export default connet()(CountUI)
```

## 二、Provider
由于我们的状态可能会被很多组件使用，所以react-redux给我我们提供了一个`Provider`组件，可以全局注入redux中的store，只需要把 `Provider` 注册在根部组件即可。

例如，当以下组件都是需要使用到`store`时，我们需要这么做，但是这样徒增了工作量，很不便利

```js
    <Count store={store} />
    <Demo1 store={store} />
    <Demo2 store={store} />
    <Demo3 store={store} />
    <Demo4 store={store} />
```

我们可以这么做：在src目录下的`index.js`文件中，引入`Provider`，直接用`Provider`标签包裹`App`组件，将`store`写在`Provider`中即可

```js
    ReactDOM.render(
        <Provider store={store}>
            <App />
        </Provider>,
        document.getElementById('root')
    )
```

这样我们在`App.js`文件中，组件无需手写指定`store`，即可使用`store`

## 三、connect

在react-redux的原型图中，我们会发现容器组件需要给UI组件传递状态和方法，并且是通过`props`来传递，看起来很简单。但是我们会发现容器组件中似乎没有我们平常传递`props`的形式。

这时候就需要继续研究下容易组件的唯一一个函数`connect`

`connect`方法是一个连接器，用于连接容器组件和UI组件，它第一次执行时候，接收4个参数。这些参数都是可选的，它执行的结果还是一个函数，第二次执行接收一个UI组件。

第一次执行时候的四个函数：`mapStateToProps`、`mapDispachToProps`、`mergeProps`、`options` 

### mapStateToProps

```js
    const mapStateToProps = state => ({ count: state })
```

它接收`state`作为参数，并且返回一个对象，这个对象标识着UI组件的同名参数

返回的对象中的key就作为传递给UI组件props的key，value就作为props的value

如上面的代码，我们可以在UI组件中直接通过props来读取`count`值

```js
    <h1>当前求和为：{this.props.count}</h1>
```
这样我们就打通了容器组件向UI组件传递状态。

### mapDispachToProps 

connect 接收的第二个参数是 `mapDisaptchToProps` 它是用于建立UI组件的参数到 `store.dispach` 方法的映射

我们可以把参数写成对象形式，在这里定义action执行的方法，例如：
```js
    {
        increment: createIncrementAction,
        subtraction: createSubtractionAction,
        incrementAsync: createIncrementAsyncAction
    }
```
这样就实现了容器组件向UI组件传递方法的形式。
当我们通过上述形式调用了函数，创建了`action`对象，但是好像并没有执行`dispatch`向`store`分发，如果不执行分发那后续的为什么能执行呢？

其实这里`react-redux`已经帮我们做了优化，当我们调用`actionCreator`的时候，会立即发送`action`给`store`而不用手动的`dispatch`。实现自动调用dispatch

## 四、react-redux完整开发流程
 
1.首先我们在 `containers `文件夹中，直接编写我们的容器组件和UI组件。

2.通过引用`connect`方法来连接UI组件和容器组件

```js
    import { connect } from 'react-redux'
    export default connect()(具体组件)
```

3.从`action`文件中暴露创建`action`的方法

```js
    export const createIncrementAction = data => ({type:'increment',data}) 
```

```js
    import { createIncrementAction } from 'xxxx'
```

4.编写reducers方法

```js
    export default function countReducer (proState = 0 , action){
        const {type,data} = action
        switch (type) {
            case 'increment':
                return proState + data*1
            case 'subtraction':
                return proState - data*1
            default:
                return proState
        }
    }
```

5.编写UI组件，通过props传递状态和方法
```js
    return(
        <div>
            <h2>当前求和：{this.props.count}</h2>
            <button onClick={this.props.add}>点我+1</button>
        </div>
    )
```

6。调用`connect`包装暴露UI组件，实现容器组件向UI组件传递状态和方法
```js
    export default connect(
        state => ({ count : state}),
        { increment: createIncrementAction }
    )(Count)
```

## 五、数据共享
当我们想要用多个数据进行存储状态中时，则需要引入 redux 中的 `combineReducers` 方法用于存储。以对象的形式传入需要存储的数据。
```js
    import { createStore, applyMiddleware, combineReducers } from 'redux'
    import countReducer from './reducers/count'
    import personReducer from './reducers/person'
    import thunk from 'redux-thunk'

    export default createStore(combineReducers({
        count: countReducer,
        person: personReducer
    }), applyMiddleware(thunk))
```


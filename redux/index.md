title: 前端状态管理机： react+redux浅析
speaker: 王之彦
url: https://github.com/zhiyan
transition: slide3
theme: moon

[slide]
# 前端状态管理机： react+redux的理解
## 分享人： 王之彦


[slide]
# 1.开发中可能会遇到的问题

[slide]
[magic data-transition="cover-circle"]
## 一般web页面对状态管理的需求和场景
----
<div class="columns-1">
    <pre>
    a.js:
    <code class="javascript">
    const data = {}
    fetch('xxxx.json')
    .then(res=>{
        data.x = res.a
        data.y = res.b
    })
    
    ...
    router.go('b', {params: data})
    </code></pre>
    <pre>
    b.js:
    <code class="javascript">
    const data = router.get('params')

    manipulate(data)

    router.go('a', {params: data})
    </code></pre>
</div>
====
## 缺点
----
* 状态在不同模块间同步需要手动维护
* 路由增多或业务逻辑增多，基本处于无法维护状态
[/magic]

[slide]
[magic data-transition="cover-circle"]
## 或许改进一下？
----
<div class="columns-1">
    <pre>
    a.js:
    <code class="javascript">
    fetch('xxxx.json')
    .then(res=>{
        window.data.x = res.a
        window.data.y = res.b
    })
    
    ...
    router.go('b')
    </code></pre>
    <pre>
    b.js:
    <code class="javascript">
    manipulate(window.data)

    router.go('a')
    </code></pre>
</div>
====
## 缺点
----
* 污染window
* 其它相似场景： 
    * 污染原生对象(Object/Array...)
    * 污染hash?
* 命名空间不严谨的情况下易变量冲突，操作window属性结果真可预料么？
====
<pre><code class="javascript">
    var tel = 123
    var name = id

    console.log(tel === name) //false

   
    // 好的明明规范，比如webpack:
    window.__WEBPACK_MODULES__
</code></pre>
[/magic]

[slide]
[magic data-transition="cover-circle"]
## 或者？
----
<div class="columns-1">
    <pre>
    a.js:
    <code class="javascript">
    import store from 'store'
    fetch('xxxx.json')
    .then(res=>{
        store.x = res.a
        store.y = res.b
    })
    
    ...
    router.go('b')
    </code></pre>
    <pre>
    b.js:
    <code class="javascript">
    import store from 'store'

    manipulate(store)

    router.go('a')
    </code></pre>
</div>
====
## 缺点
----
* 缺乏统一操作接口,比如我想在store.x赋值时做下类型检查，挨个改吧
* 只能查看到值改变，无法明确其含义
[/magic]

[slide]
## 再改进下？
----
<div class="columns-2">
    <pre>
    a.js:
    <code class="javascript">
    import store from 'store'

    fetch('xxxx.json')
    .then(res=>{
        store.set('x', res.a)
        store.set('y', res.b)
    })
    
    ...
    router.go('b')
    </code></pre>
    <pre>
    b.js:
    <code class="javascript">
    import store from 'store'

    store.get('x')
    store.get('y')
    ...
    store.set('x','other value')

    router.go('a')
    </code></pre>
</div>
操作的含义仍然无法了解 {:&.rollIn}

[slide]
[magic data-transition="cover-circle"]
## 尝试增加一层逻辑？
----
<div class="columns-1">
    <pre>
    a.js:
    <code class="javascript">
    import store from 'store'

    fetch('xxxx.json')
    .then(res=>{
        store.dispatch('UPDATE_ORDER_STATUS', {a:1, b:2})

        // UPDATE_ORDER_STATUS 是一个action
        // 一个action只操作与自己含义（业务）相关的数据
    })
    
    ...
    router.go('b')
    </code></pre>
</div>
====
## 尝试增加一层逻辑？
----
<div class="columns-1">
    <pre>
    store.js:
    <code class="javascript">
    const = state = {
        a,b,c,d,e
    }

    const store = {}

    store.dispatch = (action, newState) => {
        if(action === 'UPDATE_ORDER_STATUS'){
            store.a = newState.a
            store.b = newState.b

            // 或者使用immutable data
            // 我们不操作任何对象，我们只创建新的对象
            // 我们会有时光机的效果，后面会讲
            store = Object.assign({}, store, newState)
        }
    }

    export default store
    </code></pre>
</div>
====
## 还差点什么
----
* 数据操作和action混合在一起
* 逻辑多了dispatch函数会非常臃肿
[/magic]

[slide]
[magic data-transition="cover-circle"]
## 分层的原则： 一样东西只做本该它做的事情
----
<div class="columns-1">
    <pre>
    action.js:
    <code class="javascript">
    function updateOrderStatus(newState){
        return {
            action: 'UPDATE_ORDER_STATUS',
            // newState: newState
            payload: newState
        }
    }

    // 或者包含异步的数据获取
    function updateAsync(params){
        return {
            action: 'UPDATE_ASYNC',
            payload: fetch('xxx', params).payload().json()
        }
    }
    </code></pre>
</div>
action负责描述如何操作数据，将操作后的数据传给reducer {:&.rollIn}
====
## 分层的原则： 一样东西只做本该它做的事情
----
<div class="columns-1">
    <pre>
    reducer.js:
    <code class="javascript">
        ...

        const reducer = action => {
            switch(action, payload){
                case 'UPDATE_ORDER_STATUS':
                    return store = Object.assign({}, initState, payload)
                case 'UPDATE_ASYNC':
                    ...
                default 
                    return store
            }
        }
    </code></pre>
</div>
reducer根据action如何把数据存到store {:&.rollIn}
====
![](/images/reducer.svg)
[/magic]

[slide]
# 2.react应用的考虑?

[slide]
## react组件状态图
![](/images/react.svg)

[slide]
## 数据在组件间单向传递
----
* 数据流会非常‘漫长’ {:&.rollIn}
* 代码不易被阅读
* 拥有不同parent的子组件传递数据要通过其共有的更上层父组件
* 难以确定数据在父组件流动时会被怎样改变

[slide]
## 解决方案
![](/images/redux.svg)

[slide]
## 怎么理解
----
* store ---> 数据模型 ---> 数据库 {:&.rollIn}
* dispatch ---> 派发器 ---> 封装好的sql语句
* subscribe ---> 订阅回调 ---> 数据改变的触发器

[slide]
## 相似场景vue
----
* 1.x可以通过props/props.sync和 emit/broadcast在组件间传递
* 2.x去掉了props.sync

[slide]
## redux包含的部分
----
* dispatch
* action
* reducer
* state

[slide]
# 3.一个react-redux架构的webapp是什么样的?

[slide]
## 大致目录结构
----
![](/images/dir.png)

[slide]
[magic data-transition="cover-circle"]
## 看看reducer
![](/images/dir_reducer.png)
====
    <pre><code>
    import {
      GET_LOG_LIST_SUCCESS,
      QUERY_LOG_SUCCESS
    } from '../actions/log'

    const initialState = {
      list: {}
    }

    export default function log(state = initialState, action = {}) {
      switch (action.type) {
        case GET_LOG_LIST_SUCCESS:
          return Object.assign({}, initialState, { list: action.payload.data })
      
        case QUERY_LOG_SUCCESS:
            return Object.assign({}, initialState, { list: action.payload.data })
            
        default:
          return state
      }
    }
    </code></pre>
[/magic]

[slide]
[magic data-transition="cover-circle"]
## 再看看action
![](/images/dir_action.png)
====
    <pre><code>
    import api from '../api'

    export const GET_LOG_LIST_SUCCESS = 'GET_LOG_LIST_SUCCESS'
    export const QUERY_LOG_SUCCESS = 'QUERY_LOG_SUCCESS'

    export function getLogList(data) {
      return {
        type: 'GET_LOG_LIST',
        payload: {
          promise: api.post('/log/list', {data: data})
        }
      }
    }

    export function queryLog(data) {
      return {
        type: 'QUERY_LOG',
        payload: {
          promise: api.post('/log/query', {data: data})
        }
      }
    }
    </code></pre>
[/magic]

[slide]
[magic data-transition="cover-circle"]
## 真实的store.js里写些什么呢
    <pre><code>
    import {createStore, applyMiddleware, combineReducers} from 'redux'
    import thunkMiddleware from 'redux-thunk'

    import promiseMiddleware from '../middlewares/promiseMiddleware'

    import user from '../reducers/user'
    import menu from '../reducers/menu'
    import member from '../reducers/member'
    import content from '../reducers/content'
    import log from '../reducers/log'
    import upkeep from '../reducers/upkeep'
    import shop from '../reducers/shop'
    import consumer from '../reducers/consumer'

    const reducer = combineReducers({user, menu, member, content, log, upkeep, shop, consumer})

    const createStoreWithMiddleware = applyMiddleware(
      thunkMiddleware,
      promiseMiddleware({promiseTypeSuffixes: ['PENDING', 'SUCCESS', 'ERROR']})
    )(createStore)

    export default function configureStore(initialState) {
      return createStoreWithMiddleware(reducer, initialState)
    }

    </code></pre>

====
## 排除中间件，就干了两件事
* 将很多reducer合并成一个大reducer, 以后所有的action都会通过这个管道(reducer)
* 根据reducer里面已经设置好的初始数据，创建一个大的store
[/magic]

[slide]
[magic data-transition="cover-circle"]
## 利用react-redux将react和redux connect到一起
    <pre><code>
    import React, { PropTypes } from 'react'
    import { bindActionCreators } from 'redux'
    import { connect } from 'react-redux'
    import { Link } from 'react-router'
    import { getLogList, queryLog } from '../../actions/log'

    class LogList extends React.Component {

      constructor (props) {
        super(props)
      }

      componentWillReceiveProps(nextProps) {
        // 其他数据源被改变的subscribe
        this.pagination.total = nextProps.log.list.totalCount
      }

      someHandler(e){
        this.props.queryLog(1)
      }

      render () {
        return (
          <div></div>
        )
      }
    }

    function mapStateToProps(state) {
      const {log}  = state
      return {
        log: log
      }
    }

    function mapDispatchToProps(dispatch) {
      return {
        getLogList: bindActionCreators(getLogList, dispatch),
        queryLog: bindActionCreators(queryLog, dispatch)
      }
    }

    export default connect(mapStateToProps, mapDispatchToProps)(LogList)

    </code></pre>

====
## 排除中间件，就干了两件事
* 将很多reducer合并成一个大reducer, 以后所有的action都会通过这个管道(reducer)
* 根据reducer里面已经设置好的初始数据，创建一个大的store
[/magic]

[slide]
# 4.实际项目以及时光机

[slide]
# Fin.

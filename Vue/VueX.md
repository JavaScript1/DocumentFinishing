#### VueX 



#####解释：

​	 一个状态管理系统，一般项目中 View 是由多个组件构成的，那么这个时候就需要组件之间的通信 或者 通过一个组件的状态来执行响应式操作 这个时候组件内部的 Data 会储存两种类型的数据 一种是用来通信的（非局部状态，数据）一种是用来执行逻辑操作或者负责渲染的（局部状态，数据）	

​	VueX就是将通信的（非局部状态，数据）提出来，保存在 New VueX.Store中从而实现统一管理并且降低组件耦合度

  这样带来的另一种方便就是将组件的 渲染和逻辑分离（一定程度上）



##### 构成：

​	VueX由五部分构成分别是 State , Getter , Mutation , Action , Module



##### State

​	VueX中存储对象，State是一个对象结构，可以把它理解称为组件中的 Data 



##### 访问State

​	如果当前实例（new VueX.Store）已经挂在到 Vue 原型中，则可以直接使用this.$store.state.访问	

```javascript
const state = { 
  isBolck: false 
}; 
export default new VueX.store({
  state,
  modules:{},
  mutations:{},
  getters:{},
  actions:{},
});
// 模块的状态也保存在 state 中直接 state.模块名就可以了 
console.log( this.$store.state.isBlock );
```



##### mapState辅助函数

​	mapState是 VueX 对象的一个方法，用来批量获取 state 从而降低代码重复，臃肿

mapState可以批量获取 State 也可以单独获取 State

```javascript
import {mapState} from 'vuex';
...
computed:{
  // 获取 state 对象全部状态
 	...mapState({
			state: state => state
  })
  
  // 获取 state 
  ...mapState({
    isBlock: state.isBlock
  })
  
  // 访问 modules 内的状态 
  ...mapState(['模块名'])// 指代 state.模块名
}
```

​	

##### Getters

​	简单的说 getter 方法就是为了获取 state 状态 但不同的是 getter 方法更富有拓展性

你可以直接使用 getter 获取 state 状态 也可以在 getter 方法内部 进行一些操作 就想过滤器一样并且这个方法是全局的

```javascript
const state = {
  test1: 100,
  test2: {
    name: 'test',
    age: 18,
  }
}

const getters = {
	getTest2: state => {
    return state.demo.name + ':' + state.demo.age
  }
}

...

console.log( this.$store.getters['getTest2'] );//demo:15 

// 如果获取模块内部的 getter 则需传入路径 打印this.$store.getters可以看到全部路径
// namespaced：ture 前提下
console.log( this.$store.getters['path'] );
```



##### mapGetters 辅助函数

​	同 mapState 一样 mapGetters 也可以批量获取 getter 方法

```javascript
import { mapGretters } from 'vuex';
...
computed:{
  
  // 获取多个 getters 方法
  ...mapGetters(['getTest2','getTest3','getTest4'])
  
  // 获取 modules 中 getters
  ...mapGetters('模块名' , ['getter1'，'getter2'])
}
```



##### Mutation

​	如果你想更改 state 中的状态 那么就从这里入手 Mutation 是由 Key( 方法名 ) Value( 回调函数 ) 组成

需要 commit 来调用 并且 Mutation中的方法可以接受额外参数（payLoad） 

注意 Mutation 方法的操作必须是同步的

```javascript
const mutations = {
  testHandle( state , payload ){
		// DOTO		
  }
}
// 两种调用方式
this.$store.commit('testHandle', config )

this.$store.commit({
  type: testHandle,
  config 
})

// 调用模块内部Mutation 路径和 getters 同理
// namespaced：ture 前提下
 this.$store.commit('path' , payload )
```



##### mapMutations 辅助函数

​	同之前辅助函数一样 mapMutations 辅助函数也具备同样的功能

```javascript
import { mapMutations } from 'vuex'
methods: {
	...mapMutations(['mapMutation1','mapMutation2','mapMutation3'])
  // 调用模块内部的 Mutations
  ...mapMutations( '模块名' , ['方法名'] )
}
```

​	

##### Actions

​	Action 和 mutation 基本一样 不用的是 Action 提交的是 mutation 而不是直接修改 state 内的状态并且 Action 是可以包含异步操作的

​	Action 函数接受一个与store实例 属性，方法相同的对象 context 因此可以使用 context.commit 来提交一个Mutation 同时 context 可以使用 .state  .getters 等store拥有的方法 但注意 context 不是 store

```javascript
const mutations = {
	testHandle( state , payload ){
    // DOTO
  }  
}

const Actions = {
  testHandle( {commit} , payload ){
    commit('testHandle' , payload )
  }
}

// 调用 Action
this.$store.dispatch('testHandle' , payload );
this.$store.dispatch({
  type: testHandle,
  payload
});

// 调用模块内的Action
// namespaced：ture 前提下
this.$store.dispatch('path' , payload );
```



##### mapActions 辅助函数

```javascript
import { mapActions } from 'vuex'
methods:{
	...mapActions(['Action1','Action2','Action3'])
  // 调用模块内部的 Mutations
  ...mapActions( '模块名' , ['方法名'] )
}
```



##### Module

和 View Component 概念一样 VueX也具备 Module 的特性，这样我们VueX目录结构就可以 和Component一致

让组件和模块一一对应

注意目录结构

![image-20191105101208181](/Users/javascript/Library/Application Support/typora-user-images/image-20191105101208181.png)

Module 允许每一个模块都有用 state mutation action getter 方法

这里的 mutation , getter 接收到的第一个参数 state 是模块内部的 也是局部状态

getter 会自动接受4个参数

分别是 state getters rootState rootGetters

```javascript
const moduleA = {
  state: { count: 0 },
  mutations: {
    increment (state) {
      // 这里的 `state` 对象是模块的局部状态
      state.count++
    }
  },

  getters: {
    doubleCount (state,getters,rootState,rootGetters) {
      // 这里的 `state` 对象是模块的局部状态 
      return state.count * 2
    }
  }
}
```



而对于 action 来说 接受到的第一个参数 仍然是 context，但不同的是，局部状态是 context.state，跟节点的状态是 context.rootState

```javascript
const module = {
  actions: {
    // action 下接受的参数 
    testHandle({ state, commit, rootState , getters , rootGetters , dispatch }) {
     //DOTO
    }
  }
}
```



##### 命名空间 namespaced

​	强调模块的作用域 

如果 namespaced：true 的话 模块的方法需添加路径才可以访问

如果 namespaced：false 的话 模块的方法可以直接访问

简单说就是 true 情况下 每一个模块就想一个局部作用域 你需要从 Root 添加路径才能访问到具体模块

而 如果为 false 的话 从 Root. 的方式就可以访问到模块



##### namespaced 为 true 访问全局内容

如果你希望使用全局 state 和 getter，`rootState` 和 `rootGetters` 会作为第三和第四参数传入 getter，也会通过 `context` 对象的属性传入 action。

因为开启命名空间 dispatch commit 都已经被局部话  这种情况下可以使用传参的方式 调用全局的  dispatch commit 

```javascript
const actions = {
  testhandle( {commit,dispathch} , payload ){
    // commit 和 dispatch 的第三个参数 负责调用 跟节点方法
    commit('callback' , payload , { root: true })
    dispathch('callback' , payload , { root: true })
  }
}
```



##### 在局部模块 注册全部模块

```javascript
const actions = {
  testHandle:{
    root: true,
    handler( namesoacedContext , payload ){
      // DOTO
    }
  }
}
```




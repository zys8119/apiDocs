# store-vue 中文文档 [![npm](https://img.shields.io/badge/npm-Install-zys8119.svg?colorB=cb3837&style=flat-square)](https://www.npmjs.com/package/store-vue)  [![github](https://img.shields.io/badge/github-<Code>-zys8119.svg?colorB=000000&style=flat-square)](https://github.com/zys8119/store-vue)
这个是一个vue状态管理器、vue过滤、vue工具、MockJs的集合

## 大纲

[vuex 设计](#vuex-设计)

[目录架构](#目录架构)

[安装](#安装)

[使用及要求](#使用)

[教程](#教程)

[依赖](#依赖)


## vuex 设计

先来看一下标准的 vuex demo

> * vuex 的状态管理：<br>
  state => getters => view => action => commit => mutations => state(new) => getters => view(new)
 
### demo (from [demo](https://link.jianshu.com/?t=https://github.com/vuejs/vuex/blob/dev/examples/counter/store.js))

```javascript 1.8
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const state = {
  count: 0
}

const mutations = {
  increment (state) {
    state.count++
  },
  decrement (state) {
    state.count--
  }
}

const actions = {
  increment: ({ commit }) => commit('increment'),
  decrement: ({ commit }) => commit('decrement'),
  incrementIfOdd ({ commit, state }) {
    if ((state.count + 1) % 2 === 0) {
      commit('increment')
    }
  },
  incrementAsync ({ commit }) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        commit('increment')
        resolve()
      }, 1000)
    })
  }
}

const getters = {
  evenOrOdd: state => state.count % 2 === 0 ? 'even' : 'odd'
}

export default new Vuex.Store({
  state,
  getters,
  actions,
  mutations
})
```

### airforce 设计

以模块化抽离方式注入

```javascript 1.8
import axios from './axios'
import initState from './initState'
import Vue from 'vue'
import _ from 'lodash'
const configs = require("./configs.js").default;

const AIRFORCE_DO = 'AIRFORCE_DO'
const AIRFORCE_LEAVE = 'AIRFORCE_LEAVE'

// initial state
const state = initState

// getters
const getters = {
  airforce: state=>state
}

// actions
const actions = {
  action ({ commit }, data) {
    // if (data.loading) {
    //   VUX.loading.show({
    //     text: ''
    //   })
    // }
    const goods = data.goods;
    configs.axiosBefore(data);
    const restData = data;
    // const {goods, ...restData} = data
    if (data.method && (data.url || data.fullUrl)) {
      if(data.isFormData){
        const toFormData = data.data;
        var FormDataObj = new FormData();
        for(let k in toFormData){
            if(
                Object.prototype.toString.call(toFormData[k]) == "[object Object]" ||
                Object.prototype.toString.call(toFormData[k]) == "[object Array]"
            ){
                FormDataObj.append(k,JSON.stringify(toFormData[k]));
            }else {
                FormDataObj.append(k,toFormData[k]);
            }
        }
        restData.data = FormDataObj;
      };
      return axios(restData).then(res => {
        let result
        try {
          result = res.json()
        } catch (e) {}
        if (result) {
          return result
        }
        return res
      }).then(result => {
        configs.axiosThenBefore(result,data,commit);
        // VUX.loading.hide();
        if (data.resthen) {
            data.resthen(result)
        }
        data.goods = _.merge({}, data.goods, result.data)
        commit(AIRFORCE_DO, { data })
        if (data.callback) {
          data.callback(result.data)
        }
        if (result.status >= 200 && result.status < 300 && result.data.status !== 'ERROR') {
          return new Promise((resolve, reject) =>{
            resolve(result.data)
          })
        } else {
          return new Promise((resolve, reject) => {
            reject(result)
          })
        }
      }).then(res => {
        if (data.success) {
          data.success(res)
        };
        configs.axiosThen(res,data,commit);
        // //登录超时请重新登录
        // if(res.code === 10010){
        //     commit(AIRFORCE_LEAVE, { data:{moduleName:'login_post'}});
        //     delete localStorage.login_post;
        //     VUX.toast.text(res.message);
        //     VUX._this.$router.push("/Loogin");
        // };
        return res
      }).catch(e => {
          configs.axiosCatch(e,data,commit);
        // VUX.loading.hide();
        if (data.error) {
          data.error(e)
          return e
        }
        if (!data.noError && e.data && e.data.message) {
          // VUX.toast.text(e.data.message)
        }
        return e
      })
    } else {
      commit(AIRFORCE_DO, { data })
    }
  },
  unload ({ commit }, data) {
    commit(AIRFORCE_LEAVE, { data })
  }
}

// mutations
const mutations = {
  [AIRFORCE_DO] (state, { data }) {
    if (_.isObject(data.goods) && !_.isArray(data.goods)) {
      Vue.set(state, data.moduleName, _.merge({}, state[data.moduleName], data.goods))
    } else {
      Vue.set(state, data.moduleName, data.goods)
    }
    // console.group()
    // console.log('action', data)
    // console.log('airforce', state)
    // console.groupEnd()
  },
  [AIRFORCE_LEAVE] (state, { data }) {
    state[data.moduleName] = undefined
  }
}

export default {
  state,
  getters,
  actions,
  mutations
}

```
<hr>

## 目录架构

```
api -| -------------------------------------api模块
    index.js -------------------------------api处理
filters -| ---------------------------------过滤模块
    index.js -------------------------------过滤处理
Mock -| ------------------------------------Mock模块
    index.js -------------------------------模拟数据处理
utils -| -----------------------------------工具模块
    utils.js -------------------------------工具处理
airforce.js --------------------------------airforce模块设计
axios.js -----------------------------------axios处理
configs.js ---------------------------------配置
index.js -----------------------------------入口
initState.js -------------------------------模块初始数据
public.js ----------------------------------公共类，暂未暴露，可以局部引入
useStore.js --------------------------------插件导入
VuexStore.js -------------------------------原生vuex暴露
```

## 安装

```angular2html
npm i store-vue
```
## 使用

```angular2html
//引入store-vue
import store from "store-vue"
new Vue({
  store,
  router,
  render: h => h(App)
}).$mount('#app')
```

> 注意：使用store-vue将会要求当前项目保证有以下目录结构,便于规范项目架构

```
api -| -------------------------------------api模块
    index.js -------------------------------api处理
filters -| ---------------------------------过滤模块
    index.js -------------------------------过滤处理
Mock -| ------------------------------------Mock模块
    index.js -------------------------------模拟数据处理
utils -| -----------------------------------工具模块
    utils.js -------------------------------工具处理
store -| -----------------------------------vuex模块
    index.js -------------------------------airforce模块数据
    configs.js -------------------------------配置
    VuexStore.js -------------------------------原生vuex扩展
```

### api->index.js

> 示例

```javascript
export default {
    /**
    * apiDome
    * @param args {any} 参数
    * @param this {object}  vm对象
    * @return Promise
    * */
    apiDome(...args){
        this.action({
            moduleName:"",
            method:"get|pos|...",
            url:"url",
            params:{},
            data:{}
        }).then().catch()
    },
    apiDome2(...args){
        return this.action({
            moduleName:"",
            method:"get|pos|...",
            url:"url",
            params:{},
            data:{}
        })
    }
}
```


### Mock->index.js

```javascript
export default [
    //普通写法
    MockObj => MockObj("/url",option=>{
        return {
            code:0,
            msg:"成功",
            data:[]
        }
    }),
    //关闭写法
    MockObj2 => MockObj(false,"/url",option=>{
        return {
            code:0,
            msg:"成功",
            data:[]
        }
    })
    //...更多参考mockJs
]
```


### filters->index.js

```javascript
export default {
    /**
    * filtersDome vue过滤器
    * @param val 参数
    * @return {any} 新的value值
    */
    filtersDome(val){
        return val + "newVal";
    }
}
```


### utils->index.js

```javascript
export default {
    /**
    * utils 工具方法
    * @param val 参数
    * @return {any} 新的value值
    */
    utilsDome(val){
        //你的code
        return val + "newVal";
    }
}
```


### store->index.js

```javascript
export default {
    //airforce模块
    moudelName:{
    	fieldName:any
    }
}
```


### store->configs.js

```javascript
export default {
    //debug开发环境，默认production，开发模式，如需设置，请设置环境变量 process.env.NODE_ENV
    debug:"production",
    //是否加载过滤器模块，默认true，开启
    //说明：全局注册过滤器
    filters:true,
    //是否映射组件vuex数据及工具，默认true，开启
    //说明：该参数用于给每一个组件添加action方法和airfoece计算属性
    useStore:true,
    //是否加载MockJs，默认true，开启
    //说明：ajax请求拦截
    Mock:true,
    //是否自定义接口地址
    //说明：方便切换接口地址，默认false不开启
    $$rootUrl:false,
    //是否设置初始化vuex数据配置，默认true，开启
    //说明：设置初始化的vuex数据
    initState:true,
    //是否设置启用api功能，默认true，启用
    api:true,
    //是否设置启用原生VuexStore功能，默认true，启用
    VuexStore:true,
    //是否设置启用全局工具功能，默认true，启用
    //说明：全局工具库
    $utils:true,
    //发请求前
    //data请求参数
    axiosBefore:(data)=>{},
    //发请求成功
    //res回调数据
    //data请求参数
    //commit方法
    axiosThenBefore:(res,data,commit)=>{},
    //发请求成功
    //res回调数据
    //data请求参数
    //commit方法
    axiosThen:(res,data,commit)=>{},
    //发请求失败
    //err回调数据
    //data请求参数
    //commit方法
    axiosCatch:(err,data,commit)=>{},
}
```


### store->VuexStore.js

```javascript
export default {
    state:{
    
    },
    getters:{
    
    },
    actions:{
    
    },
    mutations:{
    
    },
    modules:{
    
    }
}
```

## 教程

>例子
```vue
<template>
    <div>
        <!--过滤器使用-->
        <div>{{dome | filtersDome}}</div>
        <!--v-model使用-->
        <div>{{airforce.moduleName.fieldName}}</div>
        <input :value="airforce.moduleName.fieldName" @input="airforce.input(value,fieldName,moduleName)"></div>
    </div>
</template>
<script>
    export default {
        name: "Dome",
        data(){
            return {
                dome:"dome"
            }
        },
        mounted(){
            // action动作
            this.action({
                moduleName:"moduleName",
                //...
            });
            // ajax请求
            this.action({
                moduleName:"moduleName",
                method:"get|pos|...",
                url:"url",
                params:{},
                data:{}
                //更多api请参考axios...
            });
            // airforce数据引用
            this.airforce.moduleName.fieldName
            // utils工具
            this.utils.DomeFun()
            // api使用
            this.api().apiFun()
        }
    }
</script>

<style scoped>

</style>
```

## 依赖

[vuex](https://vuex.vuejs.org/)
[lodash](https://www.lodashjs.com/)
[axios](http://www.axios-js.com/)
[mockJs](http://mockjs.com/)

### lodash

> 这个是一个js工具库

### axios

> ajax/http 请求库汇总

|     请求方式     |	browser |  node  | promise | 单一职责 |	标准规范  |
-----------------------------------------------------------------------
| XMLHttpRequest   |	  O     |  	X	 |    X   |  	O	  |    O      |
| Node HTTP        |	  X     |  	O	 |    X   |  	O	  |    O      |
| fetch            |	  O     |  	X	 |    O   |  	O	  |    O      |
| node-fetch       |	  X     |  	O	 |    O   |  	O	  |    O      |
| isomorphic-fetch |	  O     |  	O	 |    O   |  	O	  |    O      |
| superagent       |	  O     |  	O	 |    X   |  	O	  |    X      |
| axios            |	  O     |  	O	 |    O   |  	O	  |    X      |
| request          |	  X     |  	O	 |    X   |  	O	  |    X      |
| jQuery           |	  O     |  	X	 |    X   |  	X	  |    X      |
| reqwest          |	  O     |  	O	 |    O   |  	O	  |    X      |

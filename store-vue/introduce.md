# vuex 设计

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
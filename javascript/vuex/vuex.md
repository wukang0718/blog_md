# VUEX
### getters 
> 方法接收两个参数(state, getters)  
  第一个参数是state对象，第二个参数是getters对象  
  没有this，this指向undefined
```
    // getters.js文件
    export default {
        currentId(state) {
            if (state.details.currentProperty) {
                return state.details.currentProperty.currentSignbatchtypeid.split(",")[0];
            }
            return ""
        },
        isAdvice (state, getter) {
            let adviceStep = state.details.adviceStep
            let currentId = getter.currentId
            return adviceStep[currentId] !== undefined
        }
    }
```

### mutations
>  接收第一个参数是state，其余参数属于自定义参数  
   有this，this指向store对象
```
    // mutations.js
    export default {
        increment (state, payload) {
            state.details.id = id
            this.dispatch("getDetailsInfoAction")
        }
    }
```

### actions  
> 接收一个上下文的对象即store  
  有this， this指向store
```
    // action.js

    import {GETNEXTRECEIVELIST} from './types.js
    export default {
        incrementAsync ({state, commit, dispatch}) {
            if (!state.details.id) {
                dispatch("getDocId")
            } else {
                axios({
                    url: '/test',
                    method: "post",
                    data: {
                        id: state.details.id
                    }
                }).then(res => {
                    let {code, data} = res.data
                    if (code === "success") {
                        commit(GETNEXTRECEIVELIST, data.user.map(item => {
                            return {
                                ...item,
                                name: item.userCname
                            }
                        }))
                    }
                }).catch(err => {
                    console.error(err)
                })
            }
        }
    }
```

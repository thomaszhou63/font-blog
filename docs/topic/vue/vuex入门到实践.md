[TOC]
# vuex
>就我的直观理解 vuex类似于维护了一个全局的 Map对象。你可以往里存放 key-value。然后所有的state数据操作都方法化，保证操作的可追踪和数据的干净。

其实对于vuex的应用场景一开始我有点误区，因为我把它当做了一个从始至终类似于 localstorage的存在。后来发现一刷新页面，vuex中的state存放的数据会丢失。因为它只是在当前页面初始化生成的一个实例，你一刷新页面所有数据重新生成，数据就没了。

**所以，vuex只能用于单个页面中不同组件（例如兄弟组件）的数据流通。**

>**```Vuex应用场景```**

想必大家在想项目中啥情况会用到vuex吧。
官方是说到了时间你自然知道啥时候用了，因为小项目加入vuex，代码成本比较高，你得写各种action，mutation，dispatch交互。你自个儿都会恶心掉。

- 只有项目大了，组件多了，**你需要一个状态机来解决同一个页面内不同组件之间的数据交流**。父子通信可以通过```props 和 $emit```来实现，**简单的兄弟之间通信也可以通过将方法放在父组件上来写，这样父组建就可以用```props 和 $emit```来传递和接受，修改变量**
- 适用于项目中多个非父子，非兄弟的组件之间的通信
- 适用于多个组件（任意）还有相同的字段值或者变量组，这些变量可以用vuex保存

>**⚠️项目感悟**：我负责的璞玉商业系统大部分是 （搜索条件/导出 + 表格显示 + 分页），然后关于搜索条件有很多种，例如下面的下拉框内的值，很多页面都会复用，以前是这些值是后台传给前台，或者前台写死，但是机动性不好啊，而且数据很麻烦很乱，所以我们专门建立一个字典表数据库，专门存放这些功能的变量组，然后在我们项目最开始的加载的时候，我们就去请求字典表接口，然后将所需要的值都存入VUex进行管理，而所有的页面就从Vuex中进行获取和操作（后续会讲解代码）
>![image](https://note.youdao.com/yws/public/resource/9395a430205de5faadfc37ac789a7b79/xmlnote/CDD3E23C0F8949358DF205DC7CEB3C6C/31692)
## ```安装 vuex、配置（以点击 => 弹窗的例子进行示范） ```
首先我们在 vue.js 2.0 开发环境中安装 vuex :
```js
npm install vuex --save
```
然后 , 在 main.js 中加入 :
```js
//vuex
import store from './store' // 引入

new Vue({
  el: '#app',
  router,
  store, // 在实例化 Vue对象时加入 store 对象 :
  template: '<App/>',
  components: { App }
})
```
我们在 src 目录下 , 新建一个 store 文件夹 , 然后在里面新建一个 index.js :
## ```modules```
```js
// index.js(不好的写法)
import Vue from 'vue'
import vuex from 'vuex'
Vue.use(vuex);

export default new vuex.Store({
    state:{
        show:false
    }
})
```
那么还有一个问题是 : 虽然现在无论哪个组件都可以使用```$store.state.show ```来访问该变量 , 那组件多了之后 , 状态也多了 , 这么多状态都堆在 ```store ```文件夹下的 ```index.js ```不好维护怎么办 ?

我们可以使用 vuex 的 modules , 把 store 文件夹下的 index.js 改成 :
```
// index.js （推荐！）
import Vue from 'vue'
import vuex from 'vuex'
Vue.use(vuex);

import dialog_store from '../components/dialog_store.js'; //引入某个store对象

export default new vuex.Store({
    modules: {
        dialog: dialog_store
    }
})
```
这里我们引用了一个 ```dialog_store.js``` , 在这个 js 文件里我们就可以单独写 dialog 组件的状态了 :
```js
<script>
// dialog_store.js
export default {
    state:{
        show:false
    }
}
</script>
```
做出这样的修改之后 , 我们将之前我们使用的``` $store.state.show ```统统改为``` $store.state.dialog.show ```即可。

>**⚠️如果还有其他的组件需要使用 vuex , 就新建一个对应的状态文件 , 然后将他们加入 store 文件夹下的 index.js 文件中的 modules 中**。
```js
// 
modules: {
    dialog: dialog_store,
    other: other, //其他组件
}
```
## ```mutations```
前面我们提到的对话框例子 , 我们对vuex 的依赖仅仅只有一个 ```$store.state.dialog.show ```一个状态 , 但是如果我们要进行一个操作 , 需要依赖很多很多个状态 , 那管理起来又麻烦了 !

mutations 登场 , 问题迎刃而解 :
```js
export default {
    state:{ // state
        show:false
    },
    mutations:{
        switch_dialog(state){ // 这里的state对应着上面这个state
            state.show = state.show?false:true;
            // 你还可以在这里执行其他的操作改变state
        }
    }
}
```
那我们弹窗的父组件就这样写
- 使用``` $store.commit('switch_dialog') ```来触发``` mutations ```中的``` switch_dialog 方法```。
```js
// 弹窗的例子  --- 父组件
<template>
  <div id="app">
    <a href="javascript:;" @click="$store.commit('switch_dialog')">点击</a>
    <t-dialog></t-dialog>
  </div>
</template>

<script>
import dialog from './components/dialog.vue' // 弹窗子组件
export default {
  components:{
    "t-dialog":dialog
  }
}
</script>
```
- **这里需要注意的是**:
    - ⚠️*==```mutations``` 中的方法是不分组件的 , 假如你在 ```dialog_stroe.js ```文件中的定义了
```switch_dialog 方法 ```, 在其他文件中也有一个 ```switch_dialog 方法 ```, 那么
```$store.commit('switch_dialog')``` 会执行所有的 ```switch_dialog 方法```==。
    - ⚠️==mutations里的操作必须是同步的==**。
你一定好奇 , 如果在 mutations 里执行异步操作会发生什么事情 , 实际上并不会发生什么奇怪的事情 , 只是官方推荐 , 不要在 mutationss 里执行异步操作而已**。
## ```actions```
多个 state 的操作 , 使用 mutations 会来触发会比较好维护 ,**那么需要执行多个 mutations 就需要用 action 了**:
```js
export default {
    state:{
        show:false
    },
    mutations:{
        switch_dialog(state){
            state.show = state.show?false:true;
            //  你还可以在这里执行其他的操作改变state
        }
    },
    actions:{
        switch(context){ //  这里的context和我们使用的$store拥有相同的对象和方法(context == $store)
            context.commit('switch_dialog'); 
            // 等价于$store.commit('switch_dialog');触发mutation方法
            //  你还可以在这里触发其他的mutations方法
        },
    }
}
```
那么 , 在之前的父组件中 , 我们需要做修改 , 来触发 action 里的 switch_dialog 方法:
```js
<!-- 修改后的 父组件-->
<template>
  <div id="app">
    <a href="javascript:;" @click="$store.dispatch('switch_dialog')">点击</a>
    <t-dialog></t-dialog>
  </div>
</template>

<script>
import dialog from './components/dialog.vue'
export default {
  components:{
    "t-dialog":dialog
  }
}
</script>
```
- **==使用 $store.dispatch('switch_dialog') 来触发 action 中的 switch_dialog 方法==**。
    - ```$store.commit('switch_dialog')```是触发mutation方法
- **==官方推荐 , 将异步操作放在 action 中==**。
## ```getters（类似vue的计算属性）```
```getters 和 vue 中的 computed 类似``` , 都是用来计算 state 然后生成新的数据 ( 状态 ) 的。

还是前面的例子 , 假如我们需要一个与状态 show 刚好相反的状态 , 使用 vue 中的 computed 可以这样算出来 
```js
export default {
    state:{
        show:false
    },
    getters:{
        not_show(state){ // 这里的state对应着上面这个state
            return !state.show;
        }
    },
    mutations:{
        ... ...
    },
    actions:{
        switch_dialog(context){
            ... ...
        },
    }
}
```
我们在组件中使用``` $store.state.dialog.show ```来获得```状态 show ```, 类似的 , ==我们可以使用 ```$store.getters.not_show ```来获得状态``` not_show```==。

>⚠️**注意 : $store.getters.not_show 的值是不能直接修改的 , 需要对应的 state 发生变化才能修改（同vue的计算属性）**。
## ``` mapState、mapGetters、mapActions```
很多时候 ,``` $store.state.dialog.show 、$store.dispatch('switch_dialog') ```这种写法又长又臭 , 很不方便 , 我们没使用 vuex 的时候 , 获取一个状态只需要 this.show , 执行一个方法只需要 ```this.switch_dialog ```就行了 , 使用 vuex 使写法变复杂了 ?

>**使用 mapState、mapGetters、mapActions 就不会这么复杂了。他们的用途就是就是将vuex中的值，变成组件自身的计算属性或者方法，然后通过```this.xxx来调用```**

>==**不推荐在组件中修改state的值，要改也是在vue的mutation或者actions那些里面去改**==
- 以 mapState 为例 :
```js
<template>
  <el-dialog :visible.sync="show"></el-dialog>
  <el-button @click="test"></el-button>
</template>

<script>
import {mapState, mapGetters} from 'vuex'; // 要声明方法！！！！！！！！！！！
export default {
  computed {
    //这里的三点叫做 : 扩展运算符
    ...mapState({ 'show' }) // 在组件中可以直接通过this.show访问,相当于组件内部自己的计算属性值一样
  },
  methods: {
    test() {
      console.log(this.channelList); // 直接当作一个本地计算属性使用
    }
  }
}
</script>
```
其他的写法还有
```js
// 适用于引入单个、多个

computed: {
    ...mapState({
        isShow: 'show'  // 将state中的show 在该组件中重新命名为 isShow
     }),
 } 
 
computed: {
 ...mapGetters({
    todosALise: 'getToDo' // getToDo 不是字符串，对应的是getter里面的一个方法名字 然后将这个方法名字重新取一个别名 todosALise
 }),
}
```
 ```js
 // 适用于引入多个，不能引用单个
 computed: {
    ...mapGetters(["quarterListStart", "quarterListEnd"]) // 表示引入多个getters值
  },
 ```
>⚠️==mapGetters、mapActions 和 mapState 类似 , mapGetters 一般也写在**computed**中 , mapActions 一般写在 methods中==。
## ```Vuex（初始化和使用） ```
本项目有一些值是很多页面会共有的变量，为了避免每个页面都单独写死变量的值或者通过后台来传送该变量到前端，然后前端再赋值。部门单独建立了一个字典数据库，以后内部系统的所有共有的值全部从字典接口中去获取。

而对于项目来说，我们需要Vue在初始加载项目的时候，一次性获取这些值当中我们需要的，然后页面通过使用vuex来获取需要的字典值。

>**vuex最大优点就是：将一些公共的变量作为整个项目的全局变量，供不同页面调用**：vuex主要是是做数据交互，父子组件传值可以很容易办到，但是兄弟组件间传值，需要先将值传给父组件，再传给子组件，异常麻烦。但是vuex适合任何关系的页面

- **初始化取值**
在项目初始加载的时候就去调用字典接口去获取我们需要通过vuex管理的数据。

- ```App.vue```
```js
<script>
...
import initDict from "@/lib/initDictionary"; // 调用字典接口的函数

export default {
  name: "app",
  mounted() {
    ... ...
    initDict.call(this);// 初始加载调用字典接口
  }... ...
};
</script>
```
- ```@/lib/initDictionary```
```js
import httpRequest from './httpRequest';
import utils from './utils';
// 调用字典接口 ==》 给vuex管理

export default function () {
    httpRequest
        .get("/api/datadict/dict/tree", {
            typeId: 105 // 请求的参数
        })
        .then(res => {
            // utils.removeEmptyItems() 是一个函数，处理请求到的数据，变成我们需要的格式
            utils.removeEmptyItems(res.data, "items");
            // this指向App.vue
            // 通过 $store.commit()来修改vuex的状态state
            // 调用vuex的mutations中的setDict函数还修改变量，并且传递参数{ name: 'purchaseBudgetOptionList', value: res.data }
            this.$store.commit('setDict', { name: 'purchaseBudgetOptionList', value: res.data });
        });

    httpRequest
        .get("/api/datadict/dict/tree", {
            typeId: 104
        })
        .then(res => {
            this.$store.commit('setDict', { name: 'cinemaTypeList', value: res.data });
        });
    ...  ...
}
```
- ``` ./utils ```
```js
// 提取请求返回的数据的值
const removeEmptyItems = (arr, key) => {
    arr.forEach(e => {
        if (e[key].length === 0) {
            e[key] = null;
        } else {
            removeEmptyItems(e[key], key);
        }
    });
};
```
>```vuex的目录store/index.js```
```js
import Vuex from 'vuex';
import Vue from 'vue';
import dictionary from './dictionary';

Vue.use(Vuex);

export default new Vuex.Store({
    modules: {
        dictionary
    }
});
```
>```vuex的目录store/disctionary.js```
```js
import Vue from 'vue';
export default {
    state: {
        purchaseBudgetOptionList: [],
        cinemaTypeList: [],
        authTypeList: [],
        channelList: [],
        userTypeList: []
    },
    // mutations放的是 Vuex中修改状态state的方法
    mutations: {
        setDict(state, { name, value }) {
            state[name] = value;
        }
        // Vuex提供了commit方法来修改状态：$store.commit()
        // 里面的方法第一个参数永远是state（上面的state状态）
        // 除第一个以外的参数才是我们通过 $store.commit() 调用该方法的参数
        // 此处{ name, value }是@lib/initDictionary.js中调用$store.commit()传递的参数
    },
    getters: {    }
};

// 将state中的每个值都转成getters，为了应对接口传送数据变化，可以动态改变

Object.keys(params.state).forEach(key => {
    params.getters[key] = state => state[key];
});

export default params;
```
> **使用Vuex的页面 searchForm.vue**
```js
<template>
  <el-dialog :visible.sync="show"></el-dialog>
</template>

<script>
import {mapState} from 'vuex';
export default {
  computed: {
    ...mapGetters(["purchaseBudgetOptionList", "authTypeList", "fgtwContractType"])
  }
}
</script>
```
这样的话，在该文件中，就可以直接使用```this.purchaseBudgetOptionList、this.authTypeList、this.fgtwContractType```直接访问state的getters中的值
# `vuex-class`

> 👉🏻[详细阅读](https://github.com/ktsn/vuex-class/)

```js
import Vue from 'vue'
import Component from 'vue-class-component'
import {
  State,
  Getter,
  Action,
  Mutation,
  namespace
} from 'vuex-class'

const someModule = namespace('path/to/module')

@Component
export class MyComp extends Vue {
  @State('foo') stateFoo
  @State(state => state.bar) stateBar
  @Getter('foo') getterFoo
  @Action('foo') actionFoo
  @Mutation('foo') mutationFoo
  @someModule.Getter('foo') moduleGetterFoo


  // 如果省略了该参数，为每个 state/getter/action/mutation 类型使用属性名
  @State foo
  @Getter bar
  @Action baz
  @Mutation qux

  created () {
    this.stateFoo // -> store.state.foo
    this.stateBar // -> store.state.bar
    this.getterFoo // -> store.getters.foo
    this.actionFoo({ value: true }) // -> store.dispatch('foo', { value: true })
    this.mutationFoo({ value: true }) // -> store.commit('foo', { value: true })
    this.moduleGetterFoo // -> store.getters['path/to/module/foo']
  }
}
```

`module`用法， 列如user和person的module

```js
import { namespace } from 'vuex-class';
const user = namespace('user');
const person = namespace('person');

export default class vuexModules extends Vue {
  @person.State(state => state.name) stateName: string;
  @user.State((state) => state.token) stateToken;
  @user.Getter('getToken') GetterToken;
  @user.Mutation("changeToken") MutationToken;
  @user.Action("updateToken") ActionToken;
  created() {
    console.log(this.stateName)
    console.log(this.stateToken)
    this.GetterToken
    this.MutationToken('MutationToken')
    this.ActionToken('ActionToken')
  }
}
```


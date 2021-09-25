## watch和computed属性

### 一、watch属性

#### 1. 简单数据类型的简单监视

 * 监视data属性中数据的变化

   ```vue
   <script>
   new Vue({
     data() {
       return {
           firstname:''
       };
     },
     watch:{
         // data中指定数据的变化监听
         'firstname':function(newValue, oldValue){
   
         }
     }
   });
   </script>
   ```

   **用实例属性监听:**

   ```vue
   <script>
   vm.$watch('msg',function(newV,oldV){ 
           //这个回调将在vm.msg改变后调用
   })
   </script>
   ```

 * ***监视路由地址的改变***

   ```vue
   <script>
     watch:{
         // this.$route.path
         '$route.path':function(oldV, newV){
             
         }
     }
   </script>
   ```

#### 2. 复杂数据类型的深度监视

```vue
<script>
//deep : true,
//handler : function(new, old){ … }
  watch:{
      //监听复杂数据类型
      stus: {
          deep:true,
          handler:(newV, oldV)=>{
              
          }
      }
  }
</script>
```

#### 3. **watch高阶使用**

1. **立即执行**（watch 是在监听属性改变时才会触发，有些时候，我们希望在组件创建后 watch 能够立即执行）可能想到的的方法就是在 create 生命周期中调用一次，但这样的写法不优雅，或许我们可以使用这样的方法：**immediate: true**

   ```vue
   <script>
       export default {
           data() {
               return {
                   name: 'Joe'
               }
           },
           watch: {
               name: {
                   handler: 'sayName',
                   immediate: true
               }
           },
           methods: {
               sayName() {
                   console.log(this.name)
               }
           }
       }
   </script>
   ```

2. **深度监听**

   在监听对象时，对象内部的属性被改变时无法触发 watch ，我们可以为其设置深度监听**deep: true**

   ```vue
   <script>
       export default {
           data: {
               studen: {
                   name: 'Joe',
                   skill: {
                       run: {
                           speed: 'fast'
                       }
                   }
               }
           },
           watch: {
               studen: {
                   handler: 'sayName',
                   deep: true
               }
           },
           methods: {
               sayName() {
                   console.log(this.studen)
               }
           }
       }
   </script>
   ```

3. **触发监听执行多个方法**

   使用**数组**可以设置多项，形式包括**字符串、函数、对象**:

   ```vue
   <script>
       export default {
           data: {
               name: 'Joe'
           },
           watch: {
               name: [
                   'sayName1',
                   function(newVal, oldVal) {
                       this.sayName2()
                   },
                   {
                       handler: 'sayName3',
                       immaediate: true
                   }
               ]
           },
           methods: {
               sayName1() {
                   console.log('sayName1==>', this.name)
               },
               sayName2() {
                   console.log('sayName2==>', this.name)
               },
               sayName3() {
                   console.log('sayName3==>', this.name)
               }
           }
       }
   </script>
   ```

4. **watch监听多个变量**

   watch本身无法监听多个变量。但我们可以将需要监听的多个变量**通过计算属性返回对象**，再监听这个对象来实现“监听多个变量”

   ```vue
   <script>
       export default {
           data() {
               return {
                   msg1: 'apple',
                   msg2: 'banana'
               }
           },
           compouted: {
               msgObj() {
                   const { msg1, msg2 } = this
                   return {
                       msg1,
                       msg2
                   }
               }
           },
           watch: {
               msgObj: {
                   handler(newVal, oldVal) {
                       if (newVal.msg1 != oldVal.msg1) {
                           console.log('msg1 is change')
                       }
                       if (newVal.msg2 != oldVal.msg2) {
                           console.log('msg2 is change')
                       }
                   },
                   deep: true
               }
           }
       }
   </script>
   ```

### 	二、computed属性

> **计算属性是基于它们的依赖进行缓存的，计算属性只有在它的相关依赖发生改变时才会重新求值，不改变会立即返回之前的计算结果**

***注意：***

* 我们在使用这些计算属性的时候，是把它们的名称，直接当作属性来使用，并不会把 计算属性 当作方法去调用。计算属性在引用的时候，一定不要加()去调用

* function 内部所用到的任何data 中的数据发生了变化，就会立即重新计算 这个 计算属性的值。计算属性的求值结果，会被缓存起来，方便下次直接使用。如果计算属性方法中，所以来的任何数据，都没有发生过变化，则不会重新对 计算属性求值

  ```vue
  <script>
  new Vue({
      
      computed:{
          'fullname':function(){
              //必须在结束时return一个值
              return this.firstname + '-' + this.lastname
          }
      }
  })
  </script>
  ```

**计算属性默认只有`getter`，可使用`setter`:**

```vue
<script>
new Vue({
    computed:{
		getCurrentSong:{
            get:function(){
                return this.currentIndex;
            }
            set:function(newV){
                this.currentIndex = newV;
            }
        }
    }
})
</script>
```

### 三、watch和computed、methods的区别

- computed 属性的结果会被缓存，除非依赖的响应式属性发生变化才会重新计算。**主要当作属性**来使用。必须在结束return 一个值

- methods 方法表示一个具体的操作，**主要书写业务逻辑**
- watch 属性是一个对象，键是需要观察的表达式(data中的某个数据)，值是对应的回调函数。主要监视某些特定数据的变化，从而进行某些具体的**业务逻辑操作**；可看作是computed和methods的结合体

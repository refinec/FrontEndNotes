# functional 函数化组件

Vue.js 组件提供了一个 functional 开关，设置为 true 后，就可以让组件变为无状态、无实例的函数化组件。因为只是函数，所以渲染的开销相对来说，较小。

* 函数化的组件中的 Render 函数，提供了第二个参数 **context ** 作为上下文，**data、props、slots、children 、 parent  **都可以通过 context 来访问。

* 因为函数式没有实例，因此组件需要的一切都是通过 `context` 参数传递，它是一个包括如下字段的对象：

  - `props`：提供所有 prop 的对象
  - `children`: VNode 子节点的数组
  - `slots`: 一个函数，返回了包含所有插槽的对象
  - `scopedSlots`: (2.6.0+) 一个暴露传入的作用域插槽的对象。也以函数形式暴露普通插槽。
  - `data`：传递给组件的整个数据对象，作为 `createElement` 的第二个参数传入组件
  - `parent`：对父组件的引用
  - `listeners`: (2.3.0+) 一个包含了所有父组件为当前组件注册的事件监听器的对象。这是 `data.on` 的一个别名。
  - `injections`: (2.3.0+) 如果使用了inject选项，则该对象包含了应当被注入的属性。

  在添加 `functional: true` 之后，需要更新我们的锚点标题组件的渲染函数，为其增加 `context`参数，并将 `this.$slots.default` 更新为 `context.children`，然后将 `this.level` 更新为 `context.props.level`。

```html
<body>
    <div id="app">
        <smart-component :data="data"></smart-component>
        <button @click="change('img')">图片组件</button>
        <button @click="change('video')">视频组件</button>
        <button @click="change('text')">文本组件</button>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script>
        let imgOptions = {
            props: ['data'],
            render(createElement){
                return createElement('div', [
                    createElement('p', '图片组件'),
                    createElement('img', {
                        attrs: {
                            src: this.data.url,
                            height:300,
                            width:400
                        }
                    })
                ])
            }
        }
        let videoOptions = {
            props: ['data'],
            render(createElement){
                return createElement('div', [
                    createElement('p', '视频组件'),
                    createElement('video', {
                        attrs: {
                            src: this.data.url,
                            controls:'controls',
                            autoplay:'autoplay'
                        }
                    })
                ])
            }
        }
        let textOptions = {
            props:['data'],
            render(createElement){
                return createElement('div', [
                    createElement('p', '文本组件'),
                    createElement('p', this.data.content)
                ])
            }
        }
        Vue.component('smart-component', {
            functional:true,
            render(createElement, context){
                function get() {
                    console.log("get....")
                    let data = context.props.data;
                    switch(data.type){
                        case 'img':

                            return imgOptions;
                        case 'video':
                            return videoOptions;
                        case 'text':
                            return  textOptions
                        default:
                            console.log(data.type)
                    }
                }
                return createElement(
                    get(),
                    {
                        props:{
                            data: context.props.data
                        }
                    },
                    context.children
                )
            },
            props:{
                data:{
                    type:Object,
                    required: true
                }
            }
        })
        let app = new Vue({
            el:'#app',
            data(){
                return {
                    data: {}
                }
            },
            components:{

            },
            methods:{
                change(type) {
                    console.log("aaa",type)
                    switch (type) {
                        case 'img':
                            this.data = {
                                type: 'img',
                                url: 'http://pic-bucket.ws.126.net/photo/0001/2019-02-07/E7D8PON900AO0001NOS.jpg'
                            }
                            break;
                        case 'video':
                            this.data = {
                                type: 'video',
                                url: 'http://wxapp.cp31.ott.cibntv.net/6773887A7F94A71DF718E212C/03002002005B836E73A0C5708529E09C1952A1-1FCF-4289-875D-AEE23D77530D.mp4?ccode=0517&duration=393&expire=18000&psid=bbd36054f9dd2b21efc4121e320f05a0&ups_client_netip=65600b48&ups_ts=1549519607&ups_userid=21780671&utid=eWrCEmi2cFsCAWoLI41wnWhW&vid=XMzc5OTM0OTAyMA&vkey=A1b479ba34ca90bcc61e3d6c3b2da5a8e&iv=1&sp='
                            }
                            break;
                        case 'text':
                            this.data = {
                                type: 'text',
                                content: '《流浪地球》中的科学：太阳何时吞并地球？科学家已经给出时间表'
                            }
                            break;
                        default:
                            console.log("data 类型不合法：" + type);
                    }
                }
            },
            created(){
                this.change('video')
            }
        })

    </script>
</body>
```

1. 首先定义了图片组件设置对象、视频组件设置对象以及文本组件设置对象，它们都以 data 作为入参。
2. 函数化组件 smart-component，也以 data 作为入参。内部根据 get() 函数来判断需要渲染的组件类型。
3. 函数化组件 smart-component 的 render 函数，以 get() 作为第一个参数；以 smart-component 所传入的 data，作为第二个参数
4. 根实例 app 的 change 方法，根据输入的类型，来切换不同的组件所需要的数据

## 应用场景

函数化组件不常用，它应该应用于以下场景：

- 需要通过编程实现在多种组件中选择一种。
- children、props 或者 data 在传递给子组件之前，处理它们。
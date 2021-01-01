# vue-axios

```javascript
npm install --save axios vue-axios

import Vue from 'vue'
import axios from 'axios'
import VueAxios from 'vue-axios'
Vue.use(VueAxios, axios)

// 	绑定了axios到vue 或this
Vue.axios.get(api).then((response) => {
    console.log(response.data)
})
this.axios.get(api).then((response) => {
    console.log(response.data)
})
this.$http.get(api).then((response) => {
    console.log(response.data)
})
```


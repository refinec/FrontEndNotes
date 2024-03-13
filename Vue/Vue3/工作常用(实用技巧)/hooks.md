## 一、值改变或状态改变都显示弹窗（这能省掉很多不必要的逻辑）

```js
// Visible.js
import { ref, toValue, watchEffect } from 'vue'

export function useVisible(watchRef) {
    const visible = ref(false);
    watchEffect(() => {
        visible.value = toValue(watchRef);
    })
    return visible;
}
```

```vue
<script setup>
import { ref } from 'vue'
import { useVisible } from './Visable'

const count = ref(0);
const modalVisible = useVisible(() => count.value % 5 === 0);
</script>

<template>
  <button @click="count++" >count的值被5整除的时候，自动弹窗</button>
  <button @click="modalVisible = true">手动弹窗</button>
  <Modal v-model="modalVisible" ></Modal>
</template>
```


## 过滤器 filter

> 过滤器可被用作一些常见的文本格式化

**注意：** 过滤器只能用在2个地方：`{{ }}`和`v-bind`表达式

##### 1. 全局过滤器

```vue
<template>
    <div>
        <p>{{ msg | msgFormat}}</p>
    </div>
</template>
<script>
	Vue.filter('msgFormat', function(msg){
    	return msg.replace(/\,/g, '!');
    })
    Vue.filter('msgFormat', function(msg, arg, arg2){
        return msg.replace(/\,/g, arg + arg2);
    })
    Vue.filter('test', function(msg){
        return msg + '哈哈'
    })
</script>
```

##### 2. 私有过滤器

```vue
<script>
  filters:{
      dateFormat:function(dateStr, pattern){
          
      }
  }
</script>
```


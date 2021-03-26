# 功能函数

## bytes 转换为 KB、MB、GB……

```javascript
bytesToHuman(bytes) {
    const sizes = ['Bytes', 'KB', 'MB', 'GB', 'TB'];
    if (bytes === 0) return '0 Bytes';
    const i = parseInt(Math.floor(Math.log(bytes) / Math.log(1024)), 10);
    if (i === 0) return `${bytes} ${sizes[i]}`;
    return `${(bytes / (1024 ** i)).toFixed(1)} ${sizes[i]}`;
}
```

## throttle节流函数、防抖函数

```javascript
# 节流函数(每调用一次后在规定的时间wait内不可再次调用)
throttle(callback,wait){
    let last = Date.now();
    return function(...args){
        if((Date.now() - last) > wait){
            callback.call(this,...args);
            last = Date.now();
        }
    }
}
# 防抖函数

```


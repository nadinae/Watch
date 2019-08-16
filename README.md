# Watch侦听器的实现
Watch侦听器可以检测数据的变化，他的实现原理主要是依靠**Object.defineProperty()**方法劫持所有属性的**get**和**set**。在此之前会对所有的数据进行收集，当数据发生变化的时候就会通知

js代码：
```javascript
function observe(page) {
  let data = page.data;
  let watch = page.watch;
  Object.keys(watch).forEach(v => {
    let key = v.split('.');
    let nowData = data;
    for (let i = 0; i < key.length - 1; i++) {
      nowData = nowData[key[i]];
    }
    let lastKey = key[key.length - 1];
    let watchFun = watch[v].handler || watch[v];
    let deep = watch[v].deep;
    defineRactive(nowData, lastKey, watchFun, deep, page);
  })
}
observe(originData)
```
初始化**observe(originData)**方法将**originData**数据传入观察函数，并遍历watch对象，如果watch的key是多层嵌套的，通过
```javascript
let key = v.split('.');
let nowData = data;
for (let i = 0; i < key.length - 1; i++) {
  nowData = nowData[key[i]];
}
let lastKey = key[key.length - 1];
```
拿到每一个value的key，以及**originData**处理后的对象，调用**defineRactive**数据响应函数
```javascript
function defineRactive(obj, key, watchFun, deep, page) {
  var val = obj[key];
  let that = this;
  Object.defineProperty(obj, key, {
    set: function (value) {
      watchFun.call(page, value, val);
      val = value;
    },
    get: function () {
      return val;
    }
  })
}  
```
当数据更新时会触发**Object.defineProperty**的**set**方法，从而执行**watch**对象对应的方法。
如果是多层嵌套的值，需要进行深度检测，因为多层嵌套的值内存地址没有改变不会触发**set**方法。所以需要对**val**的值进行递归操作
```javascript
if (deep && val != null && typeof val === 'object') {
  Object.keys(val).forEach(childKey => {
    defineRactive(val, childKey, watchFun, deep, page);
  })
}
```
最后**defineRactive**函数为
```javascript
function defineRactive(obj, key, watchFun, deep, page) {
  var val = obj[key];
  if (deep && val != null && typeof val === 'object') {
    Object.keys(val).forEach(childKey => {
      defineRactive(val, childKey, watchFun, deep, page);
    })
  }
  let that = this;
  Object.defineProperty(obj, key, {
    configurable: true,
    enumerable: true,
    set: function (value) {
      watchFun.call(page, value, val);
      val = value;
    },
    get: function () {
      return val;
    }
  })
}  
```
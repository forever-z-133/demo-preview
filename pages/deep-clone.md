# 对象的深拷贝

除了深拷贝与浅拷贝的区别之外，还有拷贝的数据类型方面的问题，  
比如 `NaN` `Date` `Function` `RegRex` `Symbol` `Error` `Infinity` 等特殊对象，拷贝结果并不如意。  
所以去了解些写法和彼此利弊当然要更好咯。

### 方法一、JSON 装换
```js
function deepClone(obj) {
  return JSON.parse(JSON.stringify(obj));
}
```

### 方法二、递归
```js
function deepClone(obj) {
  var result = Array.isArray(obj) ? [] : {};
  for (var key in obj) {
    if (obj.hasOwnProperty(key)) {
      if (isObject(obj[key])) {
        result[key] = deepClone(obj[key]);
      } else {
        result[key] = obj[key];
      }
    }
  }
  return result;
}
```

### 方法三、解构递归
```js
function deepClone(obj) {
  var result = { ...obj };
  for (var key in result) {
    if (result.hasOwnProperty(key)) {
      if (isObject(result[key])) {
        result[key] = { ...result };
      }
    }
  }
  return result;
}
```

### 方法四、构造函数递归
注：只支持箭头函数，普通函数会报错。
```js
function deepClone(rawObj) {
  if (!(rawObj instanceof Object)) return rawObj;
  var objectClone;
  var Constructor = rawObj.constructor;
  switch (Constructor) {
    case RegExp:
    case Error:
    case Date:
      objectClone = new Constructor(rawObj); break;
    case Function: 
      objectClone = eval(rawObj.toString()); break;
    default:
      objectClone = new Constructor();
  }
  for (var prop in rawObj) {
    objectClone[prop] = deepClone(rawObj[prop]);
  }
  return objectClone;
}
```

<iframe height="600" style="width: 100%;" scrolling="no" title="对象深拷贝" src="//codepen.io/foreverZ133/embed/BbYNpj/?height=600&theme-id=dark&default-tab=js" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/foreverZ133/pen/BbYNpj/'>对象深拷贝</a> by 张永恒
  (<a href='https://codepen.io/foreverZ133'>@foreverZ133</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
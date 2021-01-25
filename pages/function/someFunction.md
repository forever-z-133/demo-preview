# 数据处理的公共方法

需要 jQuery 插件的会在标题后加上 `$` 字样。

- [typeOf](#-判断数据类型)（判断数据类型）
- [isEmpty](#-对象是否为空)（对象是否为空）
- [addZero](#-自动补零)（自动补零）
- [random](#-随机数)（随机数）
- [getLength](#-获取长度)（获取长度）
- [removeObjectValue](#-删除对象中为空的键值对)（删除对象中为空的键值对）
- [returnObject](#-返回非空对象)（返回非空对象）
- [returnArray](#-返回可用数组)（返回可用数组）
- [returnNumber](#-返回可用数字)（返回可用数字）
- [toFixed](#-小数的取整)（小数的取整）
- [count / countMore / countPlus](#-数字计算)（数字计算）
- [objectLength](#-获取对象的长度)（获取对象的长度）
- [unique](#-数组或数据去重)（数组或数据去重）
- [forEachDeep](#-数据的深度遍历)（数据的深度遍历）
- [forInDeep](#-对象的深度遍历)（对象的深度遍历）
- [forEachAsync](#-异步循环)（异步循环）
- [deepClone](#-深拷贝)（深拷贝）
- [filterData](#-筛选数据)（筛选数据）
- [dataToArray](#-从数据中获取数组)（从数据中获取数组）
- [dataToObject](#-数据转对象)（数据转对象）
- [arrayToData](#-数组转数据)（数组转数据）
- [objectToData](#-对象转数据)（对象转数据）
- [objectToString](#-对象转字符串)（对象转字符串）
- [stringToObject](#-字符串转对象)（字符串转对象）
- [addDataToUrl](#-给链接添加参数)（给链接添加参数）
- [getDataFromUrl](#-获取链接中的数据)（获取链接中的数据）
- [objectEqual](#-对象是否相等)（对象是否相等）
- [getObjectValue](#-获取对象的值)（获取对象的值）
- [toString](#-安全转字符串)（安全转字符串）
- [getElementData](#-获取表单数据-)（获取表单数据 `$`）
- [fillElmentWithData](#-用数据填充表单-)（用数据填充表单 `$`）
- [clearElmentAndData](#-清空表单-)（清空表单 `$`）

## \* 判断数据类型

```js
function typeOf(obj) {
  const typeStr = Object.prototype.toString.call(obj).split(' ')[1];
  return typeStr.substr(0, typeStr.length - 1).toLowerCase();
}
function isType(obj, type) {
  return typeOf(obj) === type;
}
```

## \* 对象是否为空

```js
// 判 [], {}, undefined, NaN, null, "", false；不包含 0
function isEmpty(obj) {
  if (typeof obj === 'number' && isNaN(obj)) return true;
  if (typeof obj !== 'number' && !obj) return true;
  for (const key in obj) if ({}.hasOwnProperty.call(obj, key)) return false;
  return true;
}
```

## \* 自动补零

```js
function addZero(num, len = 2) {
  let numLen = (num + '').length;
  while (numLen++ < len) num = '0' + num;
  return num + '';
}
```

## \* 随机数

```js
function random(n1, n2 = 0) {
  const min = Math.min(n1, n2);
  const max = Math.max(n1, n2);
  return min + Math.random() * (max - min);
}
```

## \* 获取长度

```js
// 用于获取 object 的长度或 utf-16 字符串的实际长度
function getLength(obj) {
  if (!obj) return 0;
  let count = 0;
  if (typeof obj === 'string') {
    for (let char of obj) count++;
  } else if (Object.keys) {
    return Object.keys(obj).length;
  } else {
    for (let key in obj) if ({}.hasOwnProperty.call(obj, key)) count++;
  }
  return count;
}
```

## \* 删除对象中为空的键值对

```js
function removeObjectValue(obj, match = undefined) {
  for (const key in obj) {
    if ({}.hasOwnProperty.call(obj, key)) {
      const val = obj[key];
      if (val === match) {
        delete obj[key];
      }
    }
  }
}
```

## \* 返回非空对象

```js
// returnObject({});  // null
function returnObject(obj) {
  if (isEmpty(obj)) return null;
  return obj;
}
```

## \* 返回可用数组

```js
// 处理 ''.split('') 为 [''] 的问题
function returnArray(obj, divide) {
  if (typeof obj === 'string' && obj) return obj.split(divide);
  if (typeOf(obj) === 'array') return obj;
  return [];
}
```

## \* 返回可用数字

```js
// returnNumber(NaN, null, 'null', [], ''); // NaN
// returnNumber('0', '', 1); // 0
function returnNumber(...args) {
  for (let i = 0; i < args.length; i++) {
    const item = args[i];
    const num = parseFloat(item);
    if (!isNaN(num)) return num;
  }
  return NaN;
}
```

## \* 小数的取整

```js
// 拓展 toFixed 结尾的取整策略，且返回的是数字类型，
// 并能解决 166.665.toFixed(2) 不等于 166.67 这类部分数值未四舍五入的问题
// ceil 向上取整， floor 向下取整，round 四舍五入
function toFixed(num, decimal = 2, mathType = 'round') {
  const pow = Math.pow(10, decimal);
  const mathFunc = Math[mathType];
  return mathFunc(num * pow) / pow;
}
```

## \* 数字计算

```js
// 解决，1. 非数字型数字的运算 2. 小数计算的精度问题
// count('+', 0.1, 0.2);  // 0.3
function count(type, n1, n2) {
  n1 = parseFloat(n1);
  n2 = parseFloat(n2);
  if (isNaN(n1) || isNaN(n2)) return NaN;
  const dot1 = (n1.toString().split('.')[1] || '').length;
  const dot2 = (n2.toString().split('.')[1] || '').length;
  const maxDotLength = Math.max(dot1, dot2, 0);
  const pow = Math.pow(10, maxDotLength);
  const num1 = Math.round(n1 * pow);
  const num2 = Math.round(n2 * pow);
  switch (type) {
    case '/':
      return num1 / num2;
    case '*':
      return (num1 * num2) / (pow * pow);
    case '-':
      return (num1 - num2) / pow;
    case '+':
    default:
      return (num1 + num2) / pow;
  }
}
```

```js
// 拓展支持多个数字的运算 countMore('+', 1, 2, 3)
function countMore(type, ...nums) {
  const startConfig = { '+': 0, '-': 0, '*': 1, '/': 1 };
  if (!(type in startConfig)) throw new Error('首位入参有误');
  let result = startConfig[type];
  for (let i = 0; i < nums.length; i++) {
    if (i === 0 && (type === '-' || type === '/')) {
      result = nums[i];
      continue;
    }
    result = count(type, result, nums[i]);
  }
  return result;
}
```

```js
// 拓展支持带符号字符串式运算 countPlus('0.1+0.2')
function countPlus(str) {
  let result = str.replace(/\s/g, '');
  // 先递归处理括号内的运算
  result = result.replace(/\(([^)]*)\)/g, (match, _str) => countPlus(_str));
  // 先乘除，后加减，用 exec 正则出来一个个计算并替换
  const _numReg = '((?=\\D|\\b)-?[0-9]+[\\.\\e]?[0-9]*)';
  ['/*', '+-'].forEach(item => {
    item = item.replace(/(?=\B)/g, '\\\\').slice(0, -2);
    const _reg = new RegExp(_numReg + '([' + item + '])' + _numReg);
    let match = _reg.exec(result);
    while (match) {
      result = result.replace(match[0], count(match[2], match[1], match[3]));
      match = _reg.exec(result);
    }
  });
  return parseFloat(result);
}
```

## \* 获取对象的长度

```js
// 请注意排除 null undefined
function objectLength(obj) {
  return getLength(obj);
}
```

## \* 数组或数据去重

```js
// unique([1,1]);
// unique([{a:1},{a:2}], 'a');
function unique(arr, keyName, options = {}) {
  const temp = {};
  const customValue = options.value;

  return arr.reduce((re, item, index) => {
    let key, value;

    if (typeof keyName === 'function') key = keyName(item, index);
    key = !isEmpty(key) ? key : keyName ? item[keyName] : item;

    if (key in temp) return re;
    temp[key] = true;

    if (typeof customValue === 'function') value = customValue(item, index);
    value = !isEmpty(value) ? value : item;

    return re.concat([value]);
  }, []);
}
```

## \* 数据的深度遍历

```js
function forEachDeep(json, childKey, func, indexs = [], parents = []) {
  json.forEach((item, index) => {
    indexs.push(index);
    parents.push(item);
    func(item, indexs, parents);
    if (item[childKey] && item[childKey].length) {
      forEachDeep(item[childKey], childKey, func, indexs, parents);
    }
  });
}
```

## \* 对象的深度遍历

```js
function forInDeep(obj, func, options, map = new WeakMap()) {
  if (typeOf(obj) === 'object' || typeOf(obj) === 'array') {
    let clone = Array.isArray(obj) ? [] : {};
    if (map.get(obj)) return map.get(obj);
    map.set(obj, clone);
    for (let key in obj) {
      let val = obj[key];
      const temp = func && func(key, val, obj, clone);
      val = func ? (temp === void 0 ? val : temp) : val;
      clone[key] = forInDeep(val, func, map);
    }
    return clone;
  } else {
    return obj;
  }
}
```

## \* 异步循环

```js
/**
 * forEachAsync([data], function(index, item, next) {
 *   var img = new Image(); img.onload = next; img.src = item.url;
 * }, {
 *   number: 5,  // 每次同时发起 5 个请求
 *   finish: function(result) {}, // 结果为 next 的入参集合
 * });
 */
function forEachAsync(data, func, options = {}) {
  const timesConfig = Math.min(options.number || 5, 8); // 最大线程数
  const total = data.length - 1;
  const result = [];

  let restQueue = timesConfig; // 剩余队列数
  let started = 0; // 已发起
  let loaded = 0; // 已完成
  (function loop(index) {
    const item = data[index];
    if (!item) return;
    func(index, item, res => {
      restQueue++;
      result[index] = res;
      if (++loaded > total) return finish(result);
      loop(++started);
    });
    if (--restQueue > 0) loop(++started);
  })(0);

  // 全部运行完成
  function finish(result) {
    options.finish && options.finish(result);
  }
}
```

## \* 深拷贝

```js
function deepClone(rawObj, map = new WeakMap()) {
  if (!(rawObj instanceof Object)) return rawObj;
  let objectClone;
  const Constructor = rawObj.constructor;
  switch (Constructor) {
    case RegExp:
    case Error:
    case Date:
      objectClone = new Constructor(rawObj);
      break;
    case Function:
      objectClone = rawObj;
      break;
    default:
      objectClone = new Constructor();
  }
  if (map.get(rawObj)) return map.get(rawObj);
  map.set(rawObj, objectClone);
  for (const prop in rawObj) {
    if ({}.hasOwnProperty.call(rawObj, prop)) {
      objectClone[prop] = deepClone(rawObj[prop]);
    }
  }
  return objectClone;
}
function cloneDeep2(obj) {
  return forInDeep(obj);
}
```

## \* 筛选数据

```js
// filterData([{id:1},{id:5}], 2, 'id', { type: '>=' }); // [{id:5}]
function filterData(data, value, key, options) {
  if (typeOf(data) !== 'array' || !data.length) return [];

  options = options || {};
  const judgeType = options.type || '=='; // 判断类型
  const invert = options.invert || false; // 取反
  const deepKey = options.deepKey || ''; // 按某 key 向下递归
  let customJudgeFunc = options.custom || undefined; // 自定义判断

  if (typeof key === 'function') customJudgeFunc = key;

  let result = [];
  forEachDeep(data, deepKey, item => {
    const val = key ? item[key] : item;
    let inner = false;
    let temp = [];

    // 判断值是否对应
    if (typeOf(customJudgeFunc) === 'function') {
      inner = customJudgeFunc(item, value);
    } else if (typeOf(val) === 'array' || typeOf(val) === 'object') {
      inner = false; // 注意 eval([1,2] = 2) 的处理是有问题的
      if (judgeType === 'indexOf') {
        inner = val.includes(value);
      }
    } else {
      // eslint-disable-next-line no-eval
      inner = eval(val + judgeType + value);
    }

    if (invert) inner = !inner;

    if (inner) temp = [item];
    result = result.concat(temp);
  });

  return result;
}
```

## \* 从数据中获取数组

```js
// [{a:1},{a:2}], 'a' => [1,2]
function dataToArray(data, key, options) {
  if (typeOf(data) !== 'array' || !data.length) return [];
  if (!key) throw new Error('第二位入参有误');

  options = options || {};
  const noEmpty = options.noEmpty || false; // 排除值为空的
  const deepKey = options.deepKey || ''; // 按某 key 向下递归

  const result = [];
  forEachDeep(data, deepKey, item => {
    const val = item[key];
    if (noEmpty && (val === undefined || val === null)) return;
    result.push(val);
  });

  return result;
}
```

## \* 数据转对象

```js
// dataToObject([{id:1,x:'a'}, {id:2,x:'b'}], 'id', 'x'); // {1:'a',2:'b'}
function dataToObject(data, keyName, valueName, options) {
  if (typeOf(data) !== 'array' || !data.length) return [];

  options = options || {};
  const deepKey = options.deepKey || ''; // 按某 key 向下递归
  const customFunc = options.custom || undefined;

  const result = {};
  forEachDeep(data, deepKey, (item, indexs) => {
    const key = keyName ? item[keyName] : index;
    const val = valueName ? item[valueName] : item;
    if (customFunc) {
      customFunc(result, key, val, index);
    } else {
      result[key] = val;
    }
  });

  return result;
}
```

## \* 数组转数据

```js
// arrayToData([1,2], {a:'$item',i:'$index'}) => [{a:1,i:0},{a:2,i:1}]
function arrayToData(arr, format, options) {
  if (typeOf(arr) !== 'array' || !arr.length) return [];
  if (typeOf(arr) !== 'object' || typeof format !== 'function') throw new Error('第二位入参有误');

  options = options || {};
  let customFormat = options.custom || undefined;

  if (typeof format === 'function') customFormat = format;

  return arr.reduce((re, item, index) => {
    let newItem = {};
    if (customFormat) {
      // 自定义处理每一项，需返回对象
      newItem = customFormat(item, index);
    } else {
      // 将 format 模板中的数据替换掉
      for (let i in format) {
        if ({}.hasOwnProperty.call(format, i)) {
          let val = format[i];
          i = i.replace('$index', index);
          i = i.replace('$item', item);
          if (typeof val === 'string') {
            val = val.replace('$index', index);
            val = val.replace('$item', item);
          } else if (typeof val === 'function') {
            val = val(item, index);
          }
          newItem[i] = val;
        }
      }
    }
    return re.concat([newItem]);
  }, []);
}
```

## \* 对象转数据

```js
// objectToData({1:'a',2:'b'}, {id:'$item',value:'$key'}) => [{id:1,value:'a'}, {id:2,value:'b'}]
function objectToData(obj, format, options) {
  if (typeOf(format) !== 'object') throw new Error('第二位入参有误');

  options = options || {};
  let customFormat = options.custom || undefined;
  const result = [];

  if (typeof format === 'function') customFormat = format;

  for (const index in obj) {
    if ({}.hasOwnProperty.call(obj, index)) {
      let temp = {};
      const item = obj[index];
      if (customFormat) {
        // 自定义处理每一项，需返回对象
        temp = customFormat(item, index);
      } else {
        // 将 format 模板中的数据替换掉
        for (let i in format) {
          if ({}.hasOwnProperty.call(format, i)) {
            let val = format[i];
            i = i.replace('$key', index);
            i = i.replace('$item', item);
            if (typeof val === 'string') {
              val = val.replace('$key', index);
              val = val.replace('$item', item);
            } else if (typeof val === 'function') {
              val = val(item, index);
            }
            temp[i] = val;
          }
        }
      }
      result.push(temp);
    }
  }

  return result;
}
```

## \* 对象转字符串

```js
// {a:1,b:2} 转为 a=1&b=2
function objectToString(obj, divide = '&', concat = '=') {
  if (!obj || typeof obj !== 'object') return '';
  let result = [];
  for (const key in obj) {
    if ({}.hasOwnProperty.call(obj, key)) {
      let val = obj[key];
      if (val === undefined || val == null) val = '';
      result.push(encodeURIComponent(key) + concat + encodeURIComponent(val));
    }
  }
  return result.join(divide);
}
```

## \* 字符串转对象

```js
// a=1&b=2 转为 {a:1,b:2}
function stringToObject(str, divide = '&', concat = '=') {
  if (!str || typeof str !== 'string') return {};
  const arr = str.split(divide);
  return arr.reduce((re, item) => {
    if (!item) return re;
    const temp = item.split(concat);
    const key = temp.shift().trim();
    let val = temp.join(concat).trim();
    if (!key) return re;
    if (['null', 'undefined'].includes(val)) val = undefined;
    if (val === 'true') val = true;
    if (val === 'false') val = false;
    re[key] = val;
    return re;
  }, {});
}
```

## \* 给链接添加参数

```js
// addDataToUrl('x.html?a=1', {b:2}) // x.html?a=1&b=2
function addDataToUrl(url, data) {
  if (!data) return url;

  const concat = /\?/.test(url) ? '&' : '?';

  if (typeof data === 'string') {
    return url + concat + data;
  } else if (typeOf(data) === 'object') {
    return url + concat + objectToString(data);
  } else {
    throw new Error('入参有误');
  }
}
```

## \* 获取链接中的数据

```js
function getDataFromUrl(name, url = location.href) {
  const obj = stringToObject(url.split('?')[1], /[#?&]/);
  return name ? obj[name] : obj;
}

function getDataFromUrl(name, url = location.href) {
  var reg = new RegExp('(^|\\?|&)' + name + '=([^&]*)');
  var r = url.match(reg);
  if (r != null) return decodeURIComponent(r[2]); return null;
}
```

## \* 对象是否相等

```js
function objectEqual(a, b) {
  if (a === b) return true;

  const aProps = Object.getOwnPropertyNames(a);
  const bProps = Object.getOwnPropertyNames(b);

  if (aProps.length !== bProps.length) return false;

  for (const key of aProps) {
    if (a[key] !== b[key]) return false;
  }

  return true;
}
```

## \* 获取对象的值

```js
// getObjectValue({a:{b:1}}, 'a.b');  // 1
// getObjectValue({a:{b:1}}, 'a.c.d');  // undefined
function getObjectValue(obj, keyStr) {
  if (!keyStr) throw new Error('入参有误');
  const keys = keyStr.split('.');
  return keys.reduce((re, key) => (re || {})[key], obj);
}
```

## \* 安全转字符串

```js
function toString(obj) {
  if (obj === null || obj === undefined || isNaN(obj)) return '';
  return typeof obj === 'object' ? JSON.stringify(obj) : String(obj);
}
```

## \* 获取表单数据 `$`

```js
function getElementData($range, tagName) {
  tagName = tagName || 'name';
  var result = {};
  $range.find('[' + tagName + ']').each(function() {
    var $this = $(this),
      keyName = $this.attr(tagName);
    if ($this.is(':radio, :checkbox')) {
      result[keyName] = result[keyName] || [];
      if ($this.is(':checked')) {
        result[keyName].push($this.val());
      }
    } else if ($this.is('img')) {
      result[keyName] = $this.attr('src');
    } else {
      result[keyName] = $this.is(':input,select,textarea') ? $this.val() : $this.text();
    }
  });
  for (var key in result) {
    // 多选生成的数组改为逗号字符串
    if (Array.isArray(result[key])) {
      result[key] = result[key].join(',').replace(/^,+|,+$|,{2}/g, '');
    }
  }
  return result;
}
```

## \* 用数据填充表单 `$`

```js
function fillElmentWithData($range, data, tagName, options) {
  options = options || {};
  $range = $range ? $range : $(document);
  $range.find('[' + tagName + ']').each(function() {
    var $this = $(this),
      key = $this.attr(tagName),
      val = data[key];

    if (!key || val == undefined) return;

    if ($this.is(':radio')) {
      $this.prop('checked', val && $this.attr('value') == val);
    } else if ($this.is(':checkbox')) {
      var vals = val ? val.split(',') : [];
      var check = vals.indexOf($this.attr('value')) > -1;
      $this.prop('checked', val && check);
    } else if ($(this).is('img')) {
      val ? $this.attr('src', val) : $this.removeAttr('src');
    } else if ($this.is('select[multiple]') && typeof val === 'string') {
      $this.val(val.split(','));
    } else if ($this.is(':input, select, textarea')) {
      $this.val(val);
    } else {
      $this.text(val);
    }
  });
}
```

## \* 清空表单 `$`

```js
function clearElmentAndData($range, tagName) {
  $range = $range ? $range : $(document);
  $range.find('[' + tagName + ']').each(function(i, elment) {
    var $this = $(this);
    if ($this.is(':radio, :checkbox')) {
      $this.prop('checked', false);
    } else if ($this.is(':input,select,textarea')) {
      $this.val('');
    } else if ($this.is('img')) {
      $this.removeAttr('src');
    } else {
      $this.text('');
    }
  });
}
```

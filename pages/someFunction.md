# 数据处理的公共方法

需要 jQuery 插件的会在标题后加上 `$` 字样。

* [typeOf](#-判断数据类型)（判断数据类型）
* [isEmpty](#-对象是否为空)（对象是否为空）
* [addZero](#-自动补零)（自动补零）
* [random](#-随机数)（随机数）
* [trim](#-字符串去空)（字符串去空）
* [returnObject](#-返回非空对象)（返回非空对象）
* [returnArray](#-返回可用数组)（返回可用数组）
* [returnNumber](#-返回可用数字)（返回可用数字）
* [toFixed](#-小数的取整)（小数的取整）
* [count](#-数字计算)（数字计算）
* [unique](#-数组或数据去重)（数组或数据去重）
* [pushToUniqueArray](#-添加到去重数组)（添加到去重数组）
* [filterData](#-筛选数据)（筛选数据）
* [deepClone](#-深拷贝)（深拷贝）
* [dataToArray](#-从数据中获取数组)（从数据中获取数组）
* [arrayToData](#-数组转数据)（数组转数据）
* [dataToObject](#-数据转对象)（数据转对象）
* [objectToData](#-对象转数据)（对象转数据）
* [objectToString](#-对象转字符串)（对象转字符串）
* [stringToObject](#-字符串转对象)（字符串转对象）
* [addDataToUrl](#-给链接添加参数)（给链接添加参数）
* [getDataFromUrl](#-获取链接中的数据)（获取链接中的数据）
* [objectEqual](#-对象是否相等)（对象是否相等）
* [getObjectValue](#-获取对象的值)（获取对象的值）
* [imageToBase64](#-图片链接转-base64)（图片链接转 base64）
* [base64ToFile](#-base64-转-File-对象)（base64 转 File 对象）
* [fileToBase64](#-File-对象转-base64)（File 对象转 base64）
* [getElementData](#-获取表单数据-)（获取表单数据 `$`）
* [fillElmentWithData](#-用数据填充表单-)（用数据填充表单 `$`）
* [clearElmentAndData](#-清空表单-)（清空表单 `$`）

## * 判断数据类型
```js
function typeOf(obj) {
  var typeStr = Object.prototype.toString.call(obj).split(" ")[1];
  return typeStr.substr(0, typeStr.length - 1).toLowerCase();
}
function isType(obj, type) {
  return typeOf(obj) === type;
}
```

## * 对象是否为空
```js
function isEmpty(obj) {
  if (isNaN(obj) || (typeof obj !== 'number' && !obj)) return true;
  for (var i in obj) return false;
  return true;
}
```

## * 自动补零
```js
function addZero(num, len) {
  len = len || 2;
  var numLen = (num + '').length;
  while (numLen++ < len) { num = '0' + num; }
  return num + '';
}
```

## * 随机数
```js
function random(n1, n2) {
  n2 = n2 || 0;
  var min = Math.min(n1, n2);
  var max = Math.max(n1, n2);
  return min + Math.random() * (max - min);
}
```

## * 字符串去空
```js
function trim(str, trimType) {
  trimType = trimType || 'ALL';
  var _config = { ALL: /^\s+|\s+$/g, LEFT: /^\s+/, RIGHT: /\s+$/ };
  var reg = _config[trimType];
  return str.replace(reg, '');
}
String.prototype.trim = function(trimType) {
  return trim(this, trimType);
}
```

## * 返回非空对象
```js
// returnObject({});  // null
function returnObject(obj) {
  if (Object.keys) {
    if (Object.keys(obj).length) return obj
  } else {
    for (var key in obj) {
      if (!obj.hasOwnProperty(key)) continue;
      return obj;
    }
  }
  return null;
}
```

## * 返回数组
```js
function returnArray(obj) {
  if (!obj) return [];
  if (typeof obj === 'string') {
    return obj.split(',');
  } else if (typeOf(obj) === 'array') {
    return obj;
  } else {
    throw new Error('入参有误');
  }
}
```

## * 返回可用的数字
```js
// returnNumber(NaN, null, 'null', [], ''); // NaN
// returnNumber('0', '', 1); // 0
function returnNumber() {
  for (var i = 0; i < arguments.length; i++) {
    var item = arguments[i];
    var _item = parseFloat(item);
    if (!isNaN(_item)) return _item;
  }
  return NaN;
}
```

## * 小数的取整
```js
// 相当于 toFixed 的拓展，且返回的是数字类型
// toFixed(1.69, 1, 'round');  // 1.7
function toFixed(num, decimal, mathType) {
  decimal = decimal != null ? decimal : 2;
  mathType = mathType || 'floor';  // ceil 向上取整， floor 向下取整，round 四舍五入

  if (isNaN(parseFloat(decimal))) throw new Error('第二位入参有误');
  if (!Math[mathType]) throw new Error('第三位入参有误');

  var pow = Math.pow(10, decimal);
  var mathFunc = Math[mathType];
  return mathFunc(num * pow) / pow;
}
```

## * 数字计算
```js
// 解决，1. 非数字型数字的运算 2. 小数计算的精度问题
// count('+', 0.1, 0.2);  // 0.3
// 或 countPlus('0.1+0.2');  // 0.3
function count(type, options) {
  var nums = [].slice.call(arguments, 2);
  var _startConfig = { '+': 0, '-': 0, '*': 1, '/': 1 };
  if (!(type in _startConfig)) throw new Error('首位入参有误');

  // 可能往后会加入些配置，但如果不是对象则不是配置
  if (typeOf(options) !== 'object') {
    nums.splice(0, 0, options);
  }

  // 小数点后面最长字符长度，比如 0.1 和 0.234 则返回 3
  var maxDotLength = nums.reduce(function (max, num) {
    return Math.max(max, ([num].toString().split('.')[1] || '').length);
  }, 0);

  // 改造成整数，并计算出结果，比如 0.1 + 0.2 改为 1+2
  var startNum = _startConfig[type];
  var pow = Math.pow(10, maxDotLength);
  var result = nums.reduce(function (re, num, index) {
    num = Number(num) * pow;
    if (index === 0 && ['-', '/'].indexOf(type) > -1) return num;
    // 注：% 求余运算也有双精度误差问题，但不适用于本函数
    switch (type) {
      case '-': return re - num;
      case '*': return re * num;
      case '/': return re / num;
      case '+':
      default: return re + num;
    }
  }, startNum);

  // 回退到原小数形态，比如 3 转为 0.3
  var _divideConfig = { '+': pow, '-': pow, '*': pow * pow, '/': 1 };
  result = result / _divideConfig[type];

  return result;
}

// 延展为 countPlus('0.1+0.2') 直接传入字符串
function countPlus(str) {
  var result = str.replace(/\s/g, '');

  // 先处理括号内的运算
  result = result.replace(/\(([^\)]*)\)/g, function(match, _str) {
    return countPlus(_str);
  });

  // 先乘除，后加减，用 exec 正则出来一个个计算并替换
  var _numReg = '(-?[0-9]+[\\.\\e]?[0-9]*)';
  ['/*', '+-'].forEach(function (item) {
    item = item.replace(/(?<=\B)/g, '\\\\').slice(0, -2);
    var match, _reg = new RegExp(_numReg + '([' + item + '])' + _numReg);
    while (match = _reg.exec(result)) {
      result = result.replace(match[0], count(match[2], match[1], match[3]))
    }
  });

  return parseFloat(result);
}
```

## * 数组或数据去重
```js
function unique(arr, key) {
  var result = [],
    temp = {};
  for (var i in arr) {
    if (!arr.hasOwnProperty(i)) continue;
    var item = arr[i],
      value = key ? item[key] : item;
    if (!(value in temp)) {
      temp[value] = true;
      result.push(item)
    }
  }
  return result;
}
```

## * 添加到去重数组
```js
// pushToUniqueArray([1, 2], 2);  // [1,2]
// pushToUniqueArray([{a:1}, {a:2}], {a:2}, 'a');  // [{a:1}, {a:2}]
// 加 true 表示相同则删除，适用于 checkbox 反选
// pushToUniqueArray([1, 2], 2, null, true);  // [1]
function pushToUniqueArray(arr, newItem, key, options) {
  if (typeOf(arr) !== 'array') return [];
  if (arr.length < 1) return [newItem];

  options = options || {};
  var remove = options.remove || false;

  return arr.reduce(function (re, item) {
    var inArray = key ? (newItem[key] === item[key]) : (newItem === item);
    return inArray && remove ? re : re.concat([item]);
  }, []);
}
```

## * 筛选数据
```js
// filterData([{id:1},{id:5}], 2, 'id', { type: '>=' }); // [{id:5}]
function filterData(data, value, key, options) {
  if (typeOf(data) !== 'array' || !data.length) return [];

  options = options || {};
  var judgeType = options.type || '=='; // 判断类型
  var customJudgeFunc = options.custom || undefined; // 自定义判断
  var reserve = options.reserve || false; // 反选

  if (typeof key === 'function') customJudgeFunc = key;

  return data.reduce(function (re, item) {
    var val = key ? item[key] : item;
    var inner = false;
    var result = [];

    // 判断值是否对应
    if (typeOf(customJudgeFunc) === 'function') {
      inner = customJudgeFunc(item, value);
    } else if (typeOf(val) === 'array' || typeOf(val) === 'object') {
      inner = false; // 注意 eval([1,2] = 2) 的处理是有问题的
      if (judgeType === 'indexOf') {
        inner = val.indexOf(value) > -1;
      }
    } else {
      inner = eval(val + judgeType + value);
    }

    if (reserve) inner = !inner;
    if (inner) result = [item];

    return re.concat(result);
  }, []);
}
```

## * 深拷贝
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
      objectClone = rawObj; break;
    default:
      objectClone = new Constructor();
  }
  for (var prop in rawObj) {
    objectClone[prop] = deepClone(rawObj[prop]);
  }
  return objectClone;
}
```

## * 从数据中获取数组
```js
// [{a:1},{a:2}], 'a' => [1,2]
function dataToArray(data, key, options) {
  if (typeOf(data) !== 'array' || !data.length) return [];
  if (!key) throw new Error('第二位入参有误');

  options = options || {};
  var noEmpty = options.noEmpty || false; // 排除值为空的
  var deepKey = options.deepKey || ''; // 按某 key 向下递归

  return data.reduce(function (re, item) {
    var value = item[key],
      deep = [];
    if (noEmpty && value == undefined) return re;
    if (deepKey) {
      var child = typeOf(item) === 'array' ? item : item[deepKey];
      deep = dataToArray(child, key, options);
    }
    return re.concat([value], deep);
  }, []);
}
```

## * 数组转数据
```js
// arrayToData([1,2], {a:'$item',i:'$index'}) => [{a:1,i:0},{a:2,i:1}]
function arrayToData(arr, format, options) {
  if (typeOf(arr) !== 'array' || !arr.length) return [];
  if (typeOf(format) !== 'object') throw new Error('第二位入参有误');

  options = options || {};
  var customFunc = options.custom || undefined;
  
  if (typeof format === 'function') customFunc = format;

  return arr.reduce(function(re, item, index) {
    var temp = {};
    if (customFunc) {
      // 自定义处理每一项，需返回对象
      temp = customFunc(item, index);
    } else {
      // 将 format 模板中的数据替换掉
      for (var i in format) {
        var val = format[i];
        i = i.replace('$index', index);
        i = i.replace('$item', item);
        if (typeof val === 'string') {
          val = val.replace('$index', index);
          val = val.replace('$item', item);
        } else if (typeof val === 'function') {
          val = val(item, index);
        }
        temp[i] = val;
      }
    }
    return re.concat([temp]);
  }, []);
}
```

## * 数据转对象
```js
// dataToObject([{id:1,x:'a'}, {id:2,x:'b'}], 'id', 'x'); // {1:'a',2:'b'}
function dataToObject(data, keyName, valueName, options) {
  if (typeOf(data) !== 'array' || !data.length) return [];
  options = options || {};
  return data.reduce(function(re, item, index) {
    var key = keyName ? item[keyName] : index;
    var value = valueName ? item[valueName] : item;
    re[key] = value;
    return re;
  }, {});
}
```

## * 对象转数据
```js
// objectToData({1:'a',2:'b'}, {id:'$item',value:'$key'}) => [{id:1,value:'a'}, {id:2,value:'b'}]
function objectToData(obj, format, options) {
  if (typeOf(format) !== 'object') throw new Error('第二位入参有误');

  options = options || {};
  var customFunc = options.custom || undefined;
  var result = [];

  if (typeof format === 'function') customFunc = format;

  for (var index in obj) {
    var temp = {};
    var item = obj[index];
    if (customFunc) {
      // 自定义处理每一项，需返回对象
      temp = customFunc(item, index);
    } else {
      // 将 format 模板中的数据替换掉
      for (var i in format) {
        var val = format[i];
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
    result.push(temp);
  }

  return result;
}
```

## * 对象转字符串
```js
// {a:1,b:2} 转为 a=1&b=2
function objectToString(obj, divide, concat) {
  divide = divide || '&';
  concat = concat || '=';
  var result = [];
  for (var key in obj) {
    if (!obj.hasOwnProperty(key)) continue;
    var value = obj[key];
    if (value == undefined) value = '';
    result.push(encodeURIComponent(key) + concat + encodeURIComponent(value));
  }
  result = result.join(divide);
  return result;
}
```

## * 字符串转对象
```js
// a=1&b=2 转为 {a:1,b:2}
function stringToObject(str, divide, concat) {
  if (!str || typeof str !== 'string') return {};
  divide = divide || '&';
  concat = concat || '=';
  var arr = str.split(divide);
  return arr.reduce(function (re, item) {
    if (!item) return re;
    var temp = item.split(concat);
    var key = temp.shift().trim();
    var value = temp.join(concat).trim();
    if (!key) return re;
    if (['null', 'undefined'].indexOf(value) > -1) value = undefined;
    if (value === 'true') value = true;
    if (value === 'false') value = false;
    re[key] = value;
    return re;
  }, {});
}
```

## * 给链接添加参数
```js
// addDataToUrl('x.html?a=1', {b:2}) // x.html?a=1&b=2
function addDataToUrl(url, data) {
  if (!data) return url;
  
  var concat = /\?/.test(url) ? '&' : '?';

  if (typeof data === 'string') {
    return url + concat + data;
  } else if (typeOf(data) === 'object') {
    return url + concat + objectToString(data);
  } else {
    throw new Error('入参有误');
  }
}
```

## * 获取链接中的数据
```js
function getDataFromUrl(name, url) {
  var obj = stringToObject((url || location.href).split('?')[1], /[#?&]/);
  return name ? obj[name] : obj;
}
```

## * 对象是否相等
```js
function objectEqual(a, b) {
  if (a === b) return true;

  var aProps = Object.getOwnPropertyNames(a);
  var bProps = Object.getOwnPropertyNames(b);

  if (aProps.length != bProps.length) return false;

  for (var i = 0; i < aProps.length; i++) {
    var key = aProps[i];
    if (a[key] !== b[key]) return false;
  }
  return true;
}
```

## * 获取对象的值
```js
// getObjectValue({a:{b:1}}, 'a.b');  // 1
// getObjectValue({a:{b:1}}, 'a.c.d');  // undefined
function getObjectValue(obj, keyStr) {
  if (!keyStr) throw new Error('入参有误');
  var keys = keyStr.split('.');
  return keys.reduce(function(re, key) {
    return (re || {})[key];
  }, obj);
}
```

## * 图片链接转 base64
```js
function imageToBase64(src, callback) {
  var img = new Image();
  img.onload = function () {
    var canvas = document.createElement('canvas');
    var imgW = canvas.width = img.width;
    var imgH = canvas.height = img.height;
    var ctx = canvas.getContext('2d');
    ctx.drawImage(img, 0, 0, imgW, imgH);
    var base64 = canvas.toDataURL('image/jpeg');
    callback && callback(base64, canvas);
  };
  img.src = src;
}
```

## * base64 转 File 对象
```js
function base64ToFile(base64, imgName, callback, options) {
  options = options || {};
  var type = options.type || 'image/jpeg';
  var byteString = atob(base64.split(',')[1]);
  var ab = new ArrayBuffer(byteString.length);
  var ia = new Uint8Array(ab);
  for (var i = 0; i < byteString.length; i++) {
    ia[i] = byteString.charCodeAt(i);
  }
  var file = new File([ia], imgName, { type: type, lastModified: Date.now() });
  callback && callback(file);
}
```

## * File 对象转 base64
```js
function fileToBase64(file, callback) {
  var reader = new FileReader();
  reader.onload = function (e) {
    var base64 = e.target.result;
    callback && callback(base64, e.target);
  };
  reader.readAsDataURL(file);
}
```

## * 获取表单数据 `$`
```js
function getElementData($range, tagName) {
  tagName = tagName || 'name';
  var result = {};
  $range.find('[' + tagName + ']').each(function () {
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
  for (var key in result) { // 多选生成的数组改为逗号字符串
    if (Array.isArray(result[key])) {
      result[key] = result[key].join(',').replace(/^,+|,+$|,{2}/g, '');
    }
  }
  return result;
}
```

## * 用数据填充表单 `$`
```js
function fillElmentWithData($range, data, tagName, options) {
  options = options || {};
  $range = $range ? $range : $(document);
  $range.find('[' + tagName + ']').each(function () {
    var $this = $(this),
      key = $this.attr(tagName),
      value = data[key];

    if (!key || value == undefined) return;

    if ($this.is(':radio')) {
      $this.prop('checked', value && $this.attr('value') == value);
    } else if ($this.is(':checkbox')) {
      var vals = value ? value.split(',') : [];
      var check = vals.indexOf($this.attr('value')) > -1;
      $this.prop('checked', value && check);
    } else if ($(this).is('img')) {
      value ? $this.attr('src', value) : $this.removeAttr('src');
    } else if ($this.is('select[multiple]') && typeof value === 'string') {
      $this.val(value.split(','))
    } else if ($this.is(':input, select, textarea')) {
      $this.val(value);
    } else {
      $this.text(value);
    }
  });
}
```

## * 清空表单 `$`
```js
function clearElmentAndData($range, tagName) {
  $range = $range ? $range : $(document);
  $range.find('[' + tagName + ']').each(function (i, elment) {
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
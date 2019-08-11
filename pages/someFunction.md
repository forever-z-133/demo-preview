# 数据处理的公共方法

需要 jQuery 插件的会在标题后加上 `$` 字样。

## * 获取数据类型
```js
function type(obj) {
  var typeStr = Object.prototype.toString.call(obj).split(" ")[1];
  return typeStr.substr(0, typeStr.length - 1).toLowerCase();
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
  return (Array(len).join(0) + num).slice(-len);
}
```

## * 获取随机数
```js
function random(min, max) {
  if (typeof max !== 'number') {
    min = 0; max = min;
  }
  min = min || 0, max = max || 1;
  return min + Math.random() * max - min;
}
```

## * 数组或数据去重
```js
function unique(arr, key) {
  var result = [], temp = {};
  for (var i in arr) {
    if (!arr.hasOwnProperty(i)) continue;
    var item = arr[i], value = key ? item[key] : item;
    if (!(value in temp)) { temp[value] = true; result.push(item) }
  }
  return result;
}
```

## * 从数据中获取数组
```js
// getArrayFromData([{id:1,x:[{id:2}]}], 'id', { deepKey: 'x' });  // [1,2]
function getArrayFromData(data, key, options) {
  if (type(data) !== 'array' || !data.length) return [];
  if (!key) throw new Error('缺少必要的入参 key');

  options = options || {};
  var noEmpty = options.noEmpty || false; // 排除值为空的
  var deepKey = options.deepKey || ''; // 按某 key 向下递归

  return data.reduce(function(re, item) {
    var value = item[key], deep = [];
    if (noEmpty && value == undefined) return re;
    if (deepKey) {
      var child = type(item) === 'array' ? item : item[deepKey];
      deep = getArrayFromData(child, key, options);
    }
    return re.concat([value], deep);
  }, []);
}
```

## * 放入去重的数组中
```js
// pushToNoSameArray([1, 2], 2);  // [1,2]
// pushToNoSameArray([{a:1}, {a:2}], {a:2}, 'a');  // [{a:1}, {a:2}]
// 加 true 表示相同则删除，适用于 checkbox 反选
// pushToNoSameArray([1, 2], 2, null, true);  // [1]
// pushToNoSameArray([{a:1}, {a:2}], {a:2}, 'a', true);  // [{a:1}]
function pushToNoSameArray(arr, newItem, key, remove) {
  if (type(arr) !== 'array') return [];
  if (arr.length < 1) return [newItem];

  return arr.reduce(function(re, item) {
    var inArray = key ? (newItem[key] === item[key]) : (newItem === item);
    return inArray && remove ? re : re.concat([item]);
  }, []);
}
```

## * 筛选数据
```js
// filterData([{id:1},{id:5}], 2, 'id', { type: '>=' }); // [{id:5}]
function filterData(data, value, key, options) {
  if (type(data) !== 'array' || !data.length) return [];

  options = options || {};
  var type = options.type || '==';  // 判断类型
  var reserve = options.reserve || false; // 反选
  var deepKey = options.deepKey || '';  // 向下递归

  return data.reduce(function(re, item) {
    var val = key ? item[key] : item, result = [], deep = [];

    if (deepKey) {
      var child = type(item) === 'array' ? item : item[deepKey];
      deep = filterData(child, value, key, options);
    }

    if (type(val) === 'array' || type(val) === 'object') {
      inner = false;
    } else {
      inner = eval(val + judgeType + value);
    }

    var inner = eval(val + type + value);
    if (reserve) inner = !inner;
    if (inner) result = [item];

    return re.concat(result, deep);
  }, []);
}
```

## * 折算成金额
```js
// numberToMoney(12345.6789);  // 12,345.6789
function numberToMoney(num) {
  if (!num) return '0.00';
  num = num.toString().replace(/\$|\,/g, '');
  num = parseFloat(num);
  if (isNaN(num)) num = "0";
  return [num].toString()
    .split('.')  // 小数点前的加逗号，小数点后的不管
    .map(function(s,i) { return i ? s : s.replace(/\B(?=(\d{3})+\b)/g, ','); })
    .join('.');
}
```

## * 返回非空对象
```js
// returnObject({});  // null
function returnObject(obj) {
  if (Object.keys) {
    if (Object.keys(obj).length) return obj
  } else {
    for (var i in obj) {
      if (!obj.hasOwnProperty(key)) continue;
      return obj;
    }
  }
  return null;
}
```

## * 返回可用的数字
```js
// returnNumber(NaN, null, 'null', [], ''); // NaN
// returnNumber('0', '', 1); // 0
function returnNumber() {
  var args = [].slice.call(arguments);
  for (var i=0; i<args.length; i++) {
    var item = args[i];
    var _item = parseFloat(item);
    if (!isNaN(_item)) return _item;
  }
  return NaN;
}
```

## * 对象转为字符串
```js
// {a:1,b:2} 转为 a=1&b=2
function objectToString(obj, concat) {
  concat = concat || '&';
  var result = [];
  for (var key in obj) {
    if (!obj.hasOwnProperty(key)) continue;
    var value = obj[key];
    if (value == undefined) value = '';
    result.push(encodeURIComponent(key) + '=' + encodeURIComponent(value));
  }
  result = result.join(concat);
  return result;
}
```

## * 字符串转对象
```js
// a=1&b=2 转为 {a:1,b:2}
function stringToObject(str, divide) {
  if (!str || typeof str !== 'string') return {};
  divide = divide || '&';
  var arr = str.split(divide);
  return arr.reduce(function (re, item) {
    if (!item) return re;
    var temp = item.split('=');
    var key = temp.shift();
    var value = temp.join('=');
    key = decodeURIComponent(key);
    value = decodeURIComponent(value);
    if (!key) return re;
    if (['null', 'undefined'].some(function(x) { return x === value })) value = undefined;
    if (value === 'true') value = true;
    if (value === 'false') value = false;
    re[key] = value;
    return re;
  }, {});
}
```

## * 给链接添加参数
```js
// addParamsToUrl('x.html?a=1', {b:2}) // x.html?a=1&b=2
function addParamsToUrl(url, params) {
  var concat = /\?/.test(url) ? '&' : '?';
  if (!params) {
    return url;
  } else if (typeof params === 'string') {
    return url + concat + params;
  } else if (type(params) === 'object') {
    return url + concat + objectToString(params);
  } else {
    throw new Error('入参有误');
  }
}
```

## * 获取链接中的数据
```js
function getDataFromUrl(key, url) {
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

## * 数字计算公共方法
```js
// 解决，1. 非数字型数字的运算 2. 小数计算的精度问题
// count('+', 0.1, 0.2);  // 0.3
function count(type, options) {
  var nums = [].slice.call(arguments, 2);
  var _startConfig = { '+': 0, '-': 0, '*': 1, '/': 1 };
  if (!(type in _startConfig)) return new Error('首位入参有误');

  // 可能往后会加入些配置，但如果不是对象则不是配置
  if (type(options) !== 'object') {
    nums.splice(0, 0, options);
  }

  // 小数点后面最长字符长度，比如 0.1 和 0.234 则返回 3
  var maxDotLength = nums.reduce(function(max, num) {
    return Math.max(max, ([num].toString().split('.')[1] || '').length);
  }, 0);

  // 改造成整数，并计算出结果，比如 0.1 + 0.2 改为 1+2
  var startNum = _startConfig[type];
  var pow = Math.pow(10, maxDotLength);
  var result = nums.reduce(function(re, num, index) {
    num = Number(num) * pow;
    if (index === 0 && ['-', '/'].indexOf(type) > -1) return num;
    switch (type) {
      case '-': return re - num;
      case '*': return re * num;
      case '/': return re / num;
      // 注：% 求余运算也有双精度误差问题，但不适用本函数
      case '+': default: return re + num;
    }
  }, startNum);

  // 回退到原小数形态，比如 3 转为 0.3
  var _divideConfig = { '+': pow, '-': pow, '*': pow * pow, '/': 1 };
  result = result / _divideConfig[type];

  return result;
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
function fileObjectTobase64(file, callback) {
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
function tranDivToJson($range, tagName) {
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
    if (type(result[key]) === 'array') {
      result[key] = result[key].filter(function (item) { return !!item; }).toString();
    }
  }
  return result;
}
```

## * 用数据填充表单 `$`
```js
function fillElmentValueByTag($range, data, tagName, options) {
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
      $this.val(value.split(','));
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
function clearElmentValueByTag($range, tagName) {
  $range = $range ? $range : $(document);
  $range.find('[' + tagName + ']').each(function (i, elment) {
    var $this = $(this);
    if ($this.is(':radio, :checkbox')) {
      $this.prop('checked', false);
    } else if ($this.is(':input,select,textarea')) {
      $this.val("");
    } else if ($this.is('img')) {
      $this.removeAttr('src');
    } else {
      $this.text('');
    }
  });
}
```
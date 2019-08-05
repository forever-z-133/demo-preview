# 日常的公共方法

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
## * 自动补零
```js
function addZero(num, ) {
  len = len || 2;
  return (Array(len).join(0) + num).slice(-len);
}
```

## * 折算成金额
```js
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

## * 数字计算公共方法
```js
function count(type, options) {
  var nums = [].slice.call(arguments, 2);
  var _startConfig = { '+': 0, '-': 0, '*': 1, '/': 1 };
  if (!(type in _startConfig)) return new Error('首位入参有误');

  // 可能往后会加入些配置，但如果不是对象则不是配置
  if (Object.prototype.toString.call(options) !== '[object Object]') {
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

## * 去抖
```js
// 不断操作无效，只有停止操作 delta 秒后才触发
function debounce(fn, delta, context) {
  var timeoutID = null;
  return function() {
    var args = arguments;
    clearTimeout(timeoutID);
    timeoutID = setTimeout(function() {
      fn.apply(context, args);
    }, delta);
    return timeoutID;
  };
}
```

## * 节流
```js
// 每隔 delta 秒时触发
function throttle(fn, delta, context) {
  var timeoutID, lastDate = 0;
  return function() {
    var args = arguments;
    var nowDate = +new Date();
    if (nowDate - lastDate >= delta) {
      lastDate = nowDate;
      fn.apply(context, args);
    }
    clearTimeout(timeoutID);
    timeoutID = setTimeout(function() {
      fn.apply(context, args);
    }, delta);
    return timeoutID;
  };
}
```

## * 获取表单数据
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
    if ($.type(result[key]) === 'array') {
      result[key] = result[key].filter(function (item) { return !!item; }).toString();
    }
  }
  return result;
}
```

## * 用数据填充表单
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
      var check = vals.some(function (item) {
        return $this.attr('value') == item;
      });
      $this.prop('checked', value && check);
    } else if ($this.is('select[multiple]') && typeof value === 'string') {
      $this.val(value.split(','));
    } else if ($this.is('select') && options.fillSelect) {
      if ($this.attr('auto-init-tag') || $this.attr('auto-re-tag')) {
        $this.attr('data-auto-selected', value);
      } else {
        $this.attr('data-value', value);
      }
    } else if ($(this).is('img')) {
      value ? $this.attr('src', value) : $this.removeAttr('src');
    } else if ($this.is(':input,select,textarea')) {
      $this.val(value);
    } else {
      $this.text(value);
    }
  });
}
```

## * 清空表单
```js
function clearElmentValueByTag($range, tagName) {
  $range = $range ? $range : $(document);
  $range.find('[' + tagName + ']').each(function (i, elment) {
    var $this = $(this);
    if ($this.is('input:radio,input:checkbox')) {
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

## * 滚动加载数据
```js
function divideDataForScroll($scroller, data, page, callback, options) {
  options = options || {};
  var size = options.size || 20;
  var nowSize = page*size;
  var nowData = data.slice(0, nowSize);
  var eventId = '.scroller';

  // 数据全部加载完毕
  if (data.slice(nowSize-size, nowSize).length < 1) {
    options.finish && options.finish(data, page, size); // 全部完成
    return $scroller.off(eventId);
  }

  // 每页的回调
  callback && callback(nowData, page, size);

  var winH = $scroller.height();
  var threshold = options.threshold || 50;
  $scroller.off(eventId).on('scroll' + eventId, function(e){
    var elemH = $scroller[0].scrollHeight;
    var sTop = e.target.scrollTop;
    if (sTop + winH + threshold > elemH) {
      divideDataForScroll($scroller, data, ++page, callback, options);
    }
  });
}
```

## * 返回非空对象
```js
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

## * 异步跑批
```js
/**
 * forEachAsync([data], {
 *   number: 5,  // 每次同时发起 5 个请求
 *   customAjax: function(item, next) { // 自定义请求，next 表示运行下一个
 *     var img = new Image(); img.onload = next; img.src = item.url;
 *   },
 *   api: 'http://', // 没有 customAjax 则走接口
 *   ajaxMethod: 'ajax',  // 默认使用 Util.ajax，还可传入 get post
 *   params: function(item){ return { x: item.x } }, // 自定义接口入参，不写则默认 item
 *   afterAjax: function(response, item) {}, // 每个请求完成的回调
 *   callback: function(result) {},  // 全部跑完的回调，result 现暂是无序的，以后可能会改吧
 * });
 */
function forEachAsync(data, options) {
  options = options || {};

  var timesConfig = options.number || 5; // 每次请求 5 条
  timesConfig = Math.min(timesConfig, 8); // 每次不得同时发送太多

  // 每次几个数据，都走完则回调
  function _total_ajax(data, next) {
    var total = data.length, result = [];
    for (var i=0; i<total; i++) {
      var item = data[i];
      _ajax(item, (function(i) {
        return function(res) {
          result[i] = res;
          if (--total > 0) return;
          next && next(result);
        };
      })(i));
    }
  }

  // 每次一个数据，走完则回调
  function _ajax(item, next) {
    if (options.customAjax) {
      return options.customAjax(item, _next);
    }
    var method = options.ajaxMethod;
    var ajaxMethod = method === 'ajax' ? Util.ajax : (method ? $[method] : $.post);
    var api = options.api;
    var params = options.params ? options.params(item) : item;
    ajaxMethod(api, params, function(res){
      if(res.resultCode == 0){
        _next && _next(res);
      }
    });
    function _next(res) {
      options.afterAjax && options.afterAjax(res, item);
      next && next(res, item);
    }
  }

  // 循环数据，每 5 条一波走完再下一波
  (function loop(data, result) {
    var thisData = data.splice(0, timesConfig); // 本次请求的一波数据
    if (thisData.length < 1) {
      options.callback && options.callback(result);
      return;  // 走完了
    }
    _total_ajax(thisData, function(res) {
      result = result.concat(res);
      loop(data, result);
    });
  })(data, []);
}
```

## * 下载
```js
// download('base64', 'xx.jpg', 'image/jpeg')
// download('words', 'xx.txt', 'text/plain')
function download() {
  var scriptId = 'downloadScript';
  var args = [].slice.call(arguments);

  if (!document.getElementById(scriptId)) {
    var script = document.createElement('script');
    script.id = scriptId;
    script.src = 'http://danml.com/js/download.js';
    script.onload = scriptReady;
    document.body.appendChild(script);
  } else scriptReady();

  function scriptReady() {
    download.apply(this, args);
  }
}
```

## * 对象转为字符串
```js
// a=1&b=2 转为 {a:1,b:2}
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
// {a:1,b:2} 转为 a=1&b=2
function stringToObject(str, divide) {
  if (!str || typeof str !== 'string') return {};
  divide = divide || '&';
  var arr = str.split(divide);
  return arr.reduce(function (re, item) {
    if (!item) return re;
    var mark = item.indexOf('=');
    mark = mark < 0 ? item.length : mark;
    var key = item.slice(0, mark), value = item.slice(mark + 1);
    if (!key) return re;
    if (['null', 'undefined', ''].some(function(x) { return x === value })) value = undefined;
    if (value === 'true') value = true;
    if (value === 'false') value = false;
    re[key] = value;
    return re;
  }, {});
}
```

## * 给链接添加参数
```js
// addParamsToUrl('x.html', {a,:1}) // x.html?a=1
function addParamsToUrl(url, params) {
  var concat = /\?/.test(url) ? '&' : '?';
  if (!params) {
    return url;
  } else if ($.type(params) === 'string') {
    return url + concat + params;
  } else if ($.type(params) === 'object') {
    return url + concat + objectToString(params);
  } else {
    throw new Error('入参有误');
  }
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
  var file new File([ia], imgName, { type: type, lastModified: Date.now() });
  callback && callback(file);
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
    callback && callback(base64);
  };
  img.src = src;
}
```

## * File 对象转 base64
```js
function fileObjectTobase64(file, callback) {
  var reader = new FileReader();
  reader.onload = function (e) {
    var base64 = e.target.result;
    callback && callback(base64);
  };
  reader.readAsDataURL(file);
}
```

## * 生成二维码
```js
function createQrCode(text, callback, options) {
  options = options || {};
  var scriptId = 'qrCodeScript';
  var tempDivId = 'qrCodeTempDiv';
  var tempDiv;

  tempDiv = $('#' + tempDivId);
  if (tempDiv.length < 1) {
    var div = document.createElement('div');
    div.id = tempDivId;
    div.style.position = 'absolute';
    div.style.left = '-99999em';
    tempDiv = $(div).appendTo('body');
  }

  tempDiv.empty();
  
  if (!document.getElementById(scriptId)) {
    var script = document.createElement('script');
    script.id = scriptId;
    script.src = 'https://larsjung.de/jquery-qrcode/latest/jquery-qrcode-0.17.0.js';
    script.onload = scriptReady;
    document.body.appendChild(script);
  } else scriptReady();

  function scriptReady() {
    if (options.image) {
      var img = new Image();
      img.onload = function () {
        render(img);
      };
      img.src = options.image;
      delete options.image;
    } else {
      render();
    }
  }
  
  function render(image) {
    var opts = $.extend({
      text: text,
      size: options.width || 400, // width height 竟然没效
      mode: 4,
      image: image,
      mSize: 0.2,  // 内部图尺寸
      render: 'image'
      // ecLevel: 'H',//识别度
      // fill: '#000',//二维码颜色
      // background: '#ffffff',//背景颜色
      // quiet: 2,//边距
      // mPosX: 50 * 0.01,
      // mPosY: 50 * 0.01,
      // label: '',
      // fontname: '黑体',
      // fontcolor: '#ff9818'
    }, options || {});
    tempDiv.empty().qrcode(opts);
    setTimeout(function() {
      finish();
    }, 50);
  }
  
  function finish() {
    var $img = tempDiv.find('img').eq(0);
    if (!$img) {
      var canvas = tempDiv.find('canvas')[0];
      var base64 = canvas.toDataURL('image/jpeg');
      var img = new Image();
      img.onload = function () {
        callback && callback($(this));
      };
      img.src = base64;
      return false;
    }
    callback && callback($img);
  }
}
```

## * 动画类
```js
function Animation() {
  var animTimer = null;
  function start(start, to, duration, callback) {
    var time = Date.now();
    stop();
    (function run() {
      var per = Math.min(1, (Date.now() - time) / duration);
      if (per >= 1) return callback && callback(to, 1);
      var now = start + (to - start) * per;
      callback && callback(now, per);
      animTimer = window.requestAnimationFrame(run);
    })();
  }
  function stop() {
    animTimer && window.cancelAnimationFrame(animTimer);
  }
  return { start: start, stop: stop }
}
```
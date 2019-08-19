# 不常用的公共方法

需要 jQuery 插件的会在标题后加上 `$` 字样。

* [numberToMoney](#-折算成金额)（折算成金额）
* [returnChineseNumber](#-转中文数字)（转中文数字）
* [toAngle](#-弧度转角度)（弧度转角度）
* [toRadian](#-角度转弧度)（角度转弧度）
* [toFirstUpperCase](#-首字母大写)（首字母大写）
* [htmlToString](#-html-转义)（html 转义）
* [stringToHtml](#-html-字符串反转义)（html 字符串反转义）
* [debounce](#-去抖)（去抖）
* [throttle](#-节流)（节流）
* [download](#-下载)（下载）
* [forEachAsync](#-异步循环)（异步循环）
* [InterceptManage](#-次数拦截器)（次数拦截器）
* [Animation](#-动画类)（动画类）
* [divideDataForScroll](#-滚动加载数据)（滚动加载数据）
* [createQrCode](#-生成二维码-)（生成二维码 `$`）
* [copyText](#-复制文本)（复制文本）
* [useCache](#-使用函数结果缓存)（使用函数结果缓存）
* [PhoneShake](#-手机摇一摇)（手机摇一摇）
* [forceReflow](#-强制重排)（强制重排）
* [toHashCode](#-转为-Hash-数字)（转为 Hash 数字）
* [createCurry](#-函数科里化)（函数科里化）
* [Singleton](#-单例模式)（单例模式）

## * 折算成金额
```js
// numberToMoney(12345.6789);  // 12,345.6789
function numberToMoney(num) {
  if (!num) return '0.00';
  var numStr = [num].toString().replace(/\$|\,/g, '');
  if (isNaN(parseFloat(numStr))) numStr = "0";
  return numStr
    .split('.') // 小数点前的加逗号，小数点后的不管
    .map(function (s, i) {
      return i ? s : s.replace(/\B(?=(\d{3})+\b)/g, ',');
    })
    .join('.');
}
```

## * 转中文数字
```js
// 1001 => 一千零一
function returnChineseNumber(num, chineseType) {
  if (!num) return '';

  var numArr = [num].toString().split('');
  var _numConfig = '零一二三四五六七八九'.split('');
  var _unitConfig = ' 十百千'.split('');
  var _sizeConfig = ' 万亿兆'.split('');
  if (chineseType) {
    _numConfig = '零壹贰叁肆伍陆柒捌玖'.split('');
    _unitConfig = ' 拾佰仟'.split('');
    // _sizeConfig = ' 萬億兆'.split('');
  }

  var result = '';
  for (var i = 0, len = numArr.length; i < len; i++) {
    var n = Number(numArr[len - i - 1]);
    var cn = _numConfig[n];
    var unit = i % 4 !== 0 ? _unitConfig[i % 4] : '';
    var size = i % 4 === 0 && i !== 0 ? _sizeConfig[i / 4] : '';

    // 现在将返回 1001 => 一千零百零十一，不符合情况，则进行以下处理
    if (cn === '零') unit = '';
    if (cn === '零' && (result === '' || result.slice(0, 1) === '零')) cn = '';

    result = cn + unit + size + result;
  }

  return result;
}
```

## * 弧度转角度
```js
function toAngle(radian) {
  return radian / Math.PI * 180;
}
```

## * 角度转弧度
```js
function toRadian(angle) {
  return angle / 180 * Math.PI;
}
```

## * 首字母大写
```js
function toFirstUpperCase(str) {
  return str.slice(0, 1).toUpperCase() + str.toLowerCase().slice(1);
}
```

## * html 转义
```js
// htmlToString('<p>x</p>'); // &lt;p&gt;x&lt;/p&gt;
function htmlToString(html) {
  if (!html) return '';
  var ele = document.createElement('span');
  ele.appendChild( document.createTextNode( html ) );
  var result = ele.innerHTML;
  ele = null;
  return result;
}
```

## * html 字符串反转义
```js
// htmlToString('&lt;p&gt;x&lt;/p&gt;'); // <p>x</p>
// htmlToString('<p>x</p>'); // x 注意此方法对次运行可能不是预期结果
function stringToHtml (str) {
  if (!str) return '';
  var ele = document.createElement('span');
  ele.innerHTML = str;
  var result = ele.innerText || ele.textContent;
  ele = null;
  return result;
}
```

## * 去抖
```js
// 不断操作无效，只有停止操作 delta 秒后才触发
function debounce(fn, delta, context) {
  var timeoutID = null;
  return function () {
    var args = arguments;
    clearTimeout(timeoutID);
    timeoutID = setTimeout(function () {
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
  var safe = true;
  return function () {
    if (!safe) return;

    var args = arguments;
    fn.call(context, args);

    safe = false;
    setTimeout(function () {
      safe = true;
    }, delta);
  };
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

## * 异步循环
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
    var total = data.length,
      result = [];
    for (var i = 0; i < total; i++) {
      var item = data[i];
      _ajax(item, (function (i) {
        return function (res) {
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
    ajaxMethod(api, params, function (res) {
      if (res.resultCode == 0) {
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
      return; // 走完了
    }
    _total_ajax(thisData, function (res) {
      result = result.concat(res);
      loop(data, result);
    });
  })(data, []);
}
```

## * 次数拦截器
```js
// var im = new Util.InterceptManage();
// im.set('listOk', 2, () => console.log('ok'));
// if (!im.ok('listOk')) return;
// if (!im.ok('listOk')) return; // 两次跑完后才走 finish
function InterceptManage() {
  var temp = {};
  return {
    data: temp,
    set: function (key, times, finish) {
      temp[key] = {
        times: times,
        finish: finish
      };
      return temp[key];
    },
    ok: function (key) {
      var item = temp[key];
      if (!item) return false;
      if (--item.times < 1) {
        item.finish && item.finish();
        return true;
      } else return false;
    },
    remove: function (key) {
      delete temp[key];
    }
  }
}
```

## * 动画类
```js
function Animation() {
  var animTimer = 0;

  function start(start, to, duration, callback) {
    var time = Date.now();
    stop();
    (function run() {
      var per = Math.min(1, (Date.now() - time) / duration);
      if (per >= 1) return callback && callback(to, 1);
      var now = start + (to - start) * per;
      callback && callback(now, per);
      animTimer = requestAnimationFrame(run);
    })();
  }

  function stop() {
    cancelAnimationFrame(animTimer);
  }
  return {
    start: start,
    stop: stop
  }
}
```

## * 滚动加载数据
```js
function divideDataForScroll($scroller, data, callback, options) {
  options = options || {};
  var size = options.size || 20;
  var winH = $scroller.height();
  var threshold = options.threshold || 50;

  $scroller.removeEventListener('scroll', _scroll);
  $scroller.addEventListener('scroll', _scroll);

  var page = 1;
  _go();

  function _scroll() {
    var elemH = $scroller.scrollHeight;
    var sTop = $scroller.scrollTop;
    if (sTop + winH + threshold > elemH) _go();
  }

  function _go() {
    var nowSize = page * size;
    var nowData = data.slice(0, nowSize);

    // 数据全部加载完毕
    if (data.slice(nowSize - size, nowSize).length < 1) {
      options.finish && options.finish(data, page, size); // 全部完成
      return $scroller.removeEventListener('scroll', _scroll);
    }

    // 每页的回调
    callback && callback(nowData, page, size);
  }
}
```

## * 生成二维码 `$`
```js
// createQrCode('xx', ($img) => download($img.attr('src'), '文字.jpg', 'image/jpeg'))
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
    }, options.config || {});
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

## * 复制文本
```js
function copyText(text) {
  var $input = document.createElement('textarea');
  document.body.appendChild($input);
  $input.value = text;
  $input.select();
  document.execCommand("Copy");
  document.body.removeChild($input);
  $input = null;
}
```

## * 使用函数结果缓存
```js
// add = userCache((a,b) => a+b);
// add(1,2); add(1,2); // 第二次会直接取缓存
function useCache(fn) {
  var cache = {};
  return function(){
    var key = arguments.length + Array.prototype.join.call(arguments, ",");
    if (key in cache) return cache[key];
    else return cache[key] = fn.apply(this, arguments);
  }
}
```

## * 手机摇一摇
```js
function PhoneShake(func, options) {
  if (typeOf(func) === 'object') {
    options = func; func = undefined;
  }

  options = options || {};
  var SHAKE_THRESHOLD = options.threshold || 300;
  var last_update = 0;
  var x, y, z, last_x, last_y, last_z;
  var shakecount = 0;

  function deviceMotionHandler(e) {
    var acceleration = e.accelerationIncludingGravity;
    var curTime = new Date().getTime();
    if ((curTime - last_update) > 300) {
      var diffTime = curTime - last_update;
      last_update = curTime;
      x = acceleration.x, y = acceleration.y, z = acceleration.z;
      var speed = Math.abs(x + y + z - last_x - last_y - last_z) / diffTime * 10000;
      if (speed > SHAKE_THRESHOLD) {
        if (func) func(++shakecount);
      }
      last_x = x, last_y = y, last_z = z;
    }
  }

  function start() {
    if (!window.DeviceMotionEvent) return alert("很抱歉，您的手机设备不支持摇一摇！");
    shakecount = 0;
    window.removeEventListener('devicemotion', deviceMotionHandler, false);
    window.addEventListener('devicemotion', deviceMotionHandler, false);
    return this;
  }

  function stop() {
    window.removeEventListener('devicemotion', deviceMotionHandler, false);
    return this;
  }

  function callback(newFunc) {
    if (newFunc) func = newFunc;
    return this;
  }
  return {
    start: start,
    stop: stop,
    callback: callback
  }
}
```

## * 强制重排
```js
function forceReflow() {
  var tempDivID = "reflowDivBlock";
  var tempDiv = document.getElementById(tempDivID);
  if (!tempDiv) {
    tempDiv = document.createElement("div");
    tempDiv.id = tempDivID;
    document.body.appendChild(tempDiv);
  }
  var parentNode = tempDiv.parentNode;
  var nextSibling = tempDiv.nextSibling;
  parentNode.removeChild(tempDiv);
  parentNode.insertBefore(tempDiv, nextSibling);
}
```

## * 转为 Hash 数字
```js
function toHashCode(str){
  var hash = 0;
  if (str.length == 0) return hash;
  for (i = 0; i < str.length; i++) {
    char = str.charCodeAt(i);
    hash = ((hash << 5) - hash) + char;
    hash = hash & hash;
  }
  return hash;
}
```

## * 函数科里化
```js
// 此为科里化中的一种流派，其他的待整理
// var add = createCurry((...args) => args.reduce((re,x) => re+x, 0));
// add(1,2)(3)(4); // 10
function createCurry(fn){
  var _args = [];
  var _temp = function() {
    _args = _args.concat([].slice.call(arguments));
    return _temp;
  }
  _temp.toString = _temp.valueOf =function(){
    return fn.apply(fn, _args);
  }
  return _temp;
}
```

## * 单例模式
```js
// A = Singleton(A); var a = new A(x); var b = new A(y);
function Singleton(func) {
  var result;
  return function () {
    if (result) return result;
    var args = [].slice.call(arguments);
    result = new (func.bind.apply(func, [null].concat(args)));
    return result;
  }
}
```
# 不常用的公共方法

需要 jQuery 插件的会在标题后加上 `$` 字样。

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
  var safe = true;
  return function() {
    if (!safe) return;

    var args = arguments;
    fn.call(context, args);

    safe = false;
    setTimeout(function() {
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
      temp[key] = { times: times, finish: finish };
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
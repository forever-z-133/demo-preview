# 不常用的公共方法

需要 jQuery 插件的会在标题后加上 `$` 字样。

- [trim](#-字符串去空)（字符串去空）
- [reverseString](#-颠倒字符串)（颠倒字符串）
- [sliceString](#-截取字符串)（截取字符串）
- [toFirstUpperCase](#-首字母大写)（首字母大写）
- [numberToMoney](#-折算成金额)（折算成金额）
- [returnChineseNumber](#-转中文数字)（转中文数字）
- [toAngle / toRadian](#-角度弧度转化)（角度弧度转化）
- [distence](#-两点间距离)（两点间距离）
- [fitWidth / fitHeight](#-尺寸适配)（尺寸适配）
- [camelize / hyphenate](#-驼峰或连字符)（驼峰或连字符）
- [htmlToString](#-html-转义)（html 转义）
- [stringToHtml](#-html-字符串反转义)（html 字符串反转义）
- [jsonStringify / jsonParse](#-JSON-处理)（JSON 处理）
- [urlParse](#-返回链接对象)（返回链接对象）
- [imageToBase64](#-图片链接转-base64)（图片链接转 base64）
- [base64ToFile](#-base64-转-File-对象)（base64 转 File 对象）
- [fileToBase64](#-File-对象转-base64)（File 对象转 base64）
- [debounce](#-去抖)（去抖）
- [throttle](#-节流)（节流）
- [useCache](#-使用函数结果缓存)（使用函数结果缓存）
- [flexible](#-px-转-rem-自适应)(px 转 rem 自适应)
- [singleton](#-单例模式)（单例模式）
- [promisify](#-转-Promise)（转-Promise）
- [download](#-下载)（下载）
- [downloadExcel](#-下载-Excel)（下载 Excel）
- [copyText](#-复制文本)（复制文本）
- [getLocationData](#-获取链接信息)（获取链接信息）
- [divideDataForScroll](#-滚动加载数据)（滚动加载数据）
- [InterceptManage](#-次数拦截器)（次数拦截器）
- [Animation](#-动画类)（动画类）
- [createQrCode](#-生成二维码-)（生成二维码 `$`）
- [PhoneShake](#-手机摇一摇)（手机摇一摇）
- [toHashCode](#-转为-Hash-数字)（转为 Hash 数字）
- [partial](#-偏函数)（偏函数）
- [currying](#-函数科里化)（函数科里化）

## \* 字符串去空

```js
function trim(str, trimType) {
  trimType = trimType || 'ALL';
  const config = { ALL: /^\s+|\s+$/g, LEFT: /^\s+/, RIGHT: /\s+$/ };
  const reg = config[trimType];
  return str.replace(reg, '');
}
if (!String.prototype.trim) {
  String.prototype.trim = trimType => trim(this, trimType);
}
```

## \* 颠倒字符串

```js
// 用 for-of 避免 utf-16 字符的拆分
function reverseString(str) {
  let result = '';
  for (let char of str) {
    result = char + result;
  }
  return result;
}
```

## \* 截取字符串

```js
// 用 for-of 避免 utf-16 字符的拆分
// UTF8 字符长度为实际长度，比如 😀 算 2 个，太多表情会超字数，常用于 textarea 判断截断
// UTF16 字符长度为显示长度，比如看上去是 5 个表情，实际长度是 10 个会被多截，常用于显示超出判断
function sliceString(str, start, end = str.length, type = 'UTF8') {
  if (end < 0) end = Math.max(start, str.length + end);
  let charIndex = 0;
  let result = '';
  for (let char of str) {
    charIndex += type === 'UTF8' ? char.length : 1;
    if (charIndex >= start && charIndex <= end) {
      result += char;
    }
  }
  return result;
}
```

## \* 首字母大写

```js
function toFirstUpperCase(str) {
  return str.slice(0, 1).toUpperCase() + str.slice(1).toLowerCase();
}
function toFirstUpperCase2(str) {
  return str.replace(/\b[a-z]/g, char => char.toUpperCase());
}
```

## \* 折算成金额

```js
// numberToMoney(12345.6789);  // 12,345.6789
function numberToMoney(num) {
  if (!num) return '0.00';
  if (typeof num === 'string') {
    // 要处理 '$1,234' 和 '1e3' 的情况
    num = parseFloat(num.replace(/\$|\,/g, ''));
  }
  if (isNaN(num)) return '0.00';
  return num.toString().replace(/\d+(?=$|\.)/, digits => {
    // 仅正则整数部分
    return digits.replace(/\B(?=(\d{3})+\b)/g, ',');
  });
}
```

## \* 转中文数字

```js
// 1001 => 一千零一
function returnChineseNumber(num, chineseType) {
  if (!num) return '';

  const numArr = [num].toString().split('');
  let numConfig = '零一二三四五六七八九'.split('');
  let unitConfig = ' 十百千'.split('');
  let sizeConfig = ' 万亿兆'.split('');
  if (chineseType) {
    numConfig = '零壹贰叁肆伍陆柒捌玖'.split('');
    unitConfig = ' 拾佰仟'.split('');
    sizeConfig = ' 萬億兆'.split('');
  }

  let result = '';
  for (let i = 0, len = numArr.length; i < len; i++) {
    const n = Number(numArr[len - i - 1]);
    let cn = numConfig[n];
    let unit = i % 4 !== 0 ? unitConfig[i % 4] : '';
    const size = i % 4 === 0 && i !== 0 ? sizeConfig[i / 4] : '';

    // 现在将返回 1001 => 一千零百零十一，不符合情况，则进行以下处理
    if (cn === '零') unit = '';
    if (cn === '零' && (result === '' || result.slice(0, 1) === '零')) cn = '';

    result = cn + unit + size + result;
  }

  return result;
}
```

## \* 角度弧度转化

```js
// 弧度转角度
function toAngle(radian) {
  return (radian / Math.PI) * 180;
}
// 角度转弧度
function toRadian(angle) {
  return (angle / 180) * Math.PI;
}
```

## \* 两点间距离

```js
function distence(x1, y1, x2, y2) {
  if (x2 == undefined && y2 == undefined) {
    return Math.abs(x1 - y1);
  }
  return Math.sqrt(Math.pow(x1 - x2, 2) + Math.pow(y1 - y2, 2));
}
```

## \* 尺寸适配

```js
/// 按新宽度计算新尺寸
function fitWidth(maxWidth, imgWidth, imgHeight) {
  const newWidth = maxWidth;
  const newHeight = newWidth / imgWidth * imgHeight;
  return newHeight;
}
/// 按新高度计算新尺寸
function fitHeight(maxHeight, imgWidth, imgHeight) {
  const newHeight = maxHeight;
  const newWidth = newHeight / imgHeight / imgWidth;
  return newWidth;
}
```

## \* 驼峰或连字符

```js
function hyphenate(str) {
  return str.replace(/\B([A-Z])/g, '-$1').toLowerCase();
}
function camelize(str) {
  return str.toLowerCase().replace(/-(\w)/g, (_, s) => (s ? s.toUpperCase() : ''));
}
```

## \* html 转义

```js
// htmlToString('<p>x</p>'); // &lt;p&gt;x&lt;/p&gt;
function htmlToString(html) {
  if (!html) return '';
  let ele = document.createElement('span');
  ele.appendChild(document.createTextNode(html));
  const result = ele.innerHTML;
  ele = null;
  return result;
}
```

## \* html 字符串反转义

```js
// htmlToString('&lt;p&gt;x&lt;/p&gt;'); // <p>x</p>
// htmlToString('<p>x</p>'); // x 注意此方法对次运行可能不是预期结果
function stringToHtml(str) {
  if (!str) return '';
  let ele = document.createElement('span');
  ele.innerHTML = str;
  const result = ele.innerText || ele.textContent;
  ele = null;
  return result;
}
```

## \* JSON 处理

```js
function jsonStringify(obj) {
  // 有时会希望就是存 'null' 之类的，但个人不推荐，容易混淆
  if (obj === null || obj === undefined) obj = '';
  if (obj === 'null' || obj === 'undefined') obj = '';
  // 注意 Date Function 等对象会转不成功的哟
  // 注意 {a:null} 和 {a:undefined} 的结果不同哟
  return JSON.stringify(obj);
}
function jsonParse(str) {
  if (typeof str !== 'string') return str;
  if (!str) return ''; // 传空字符会报错
  if (str === 'null') return null;
  if (str === 'undefined') return undefined;
  let value;
  try {
    value = JSON.parse(str);
  } catch (err) {
    // JSON.parse 不成功
  }
  return value;
}
```

## \* 返回链接对象

```js
function urlParse(url = location.href) {
  let a = document.createElement('a');
  a.href = url;
  const DEFAULT_PORT = /^:?(0|80|443)$/;
  const port = a.port.replace(DEFAULT_PORT, '');
  const host = a.host.replace(DEFAULT_PORT, '');
  const origin = a.origin ? a.origin : a.protocol + '//' + host;
  const pathname = a.pathname.charAt(0) == '/' ? a.pathname : '/' + a.pathname;
  const { hash, hostname, href, protocol, search } = a;
  return { hash, host, hostname, href, origin, pathname, port, protocol, search };
}
```

## \* 图片链接转 base64

```js
function imageToBase64(src, callback) {
  const img = new Image();
  img.onload = () => {
    const canvas = document.createElement('canvas');
    const imgW = (canvas.width = img.width);
    const imgH = (canvas.height = img.height);
    const ctx = canvas.getContext('2d');
    ctx.drawImage(img, 0, 0, imgW, imgH);
    const base64 = canvas.toDataURL('image/jpeg');
    if (callback) callback(base64, canvas);
  };
  img.src = src;
}
```

## \* base64 转 File 对象

```js
function base64ToFile(base64, imgName, callback, options) {
  options = options || {};
  const type = options.type || 'image/jpeg';
  const byteString = window.atob(base64.split(',')[1]);
  const ab = new ArrayBuffer(byteString.length);
  const ia = new Uint8Array(ab);
  for (let i = 0; i < byteString.length; i++) {
    ia[i] = byteString.charCodeAt(i);
  }
  const file = new File([ia], imgName, { type, lastModified: Date.now() });
  if (callback) callback(file);
}
```

## \* File 对象转 base64

```js
function fileToBase64(file, callback) {
  const reader = new FileReader();
  reader.onload = e => {
    const base64 = e.target.result;
    if (callback) callback(base64, e.target);
  };
  reader.readAsDataURL(file);
}
```

## \* 去抖

```js
// 不断操作无效，只有停止操作 delta 秒后才触发
function debounce(fn, delta, context) {
  let timeoutID = null;
  return (...args) => {
    clearTimeout(timeoutID);
    timeoutID = setTimeout(() => {
      fn.apply(context, args);
    }, delta);
    return timeoutID;
  };
}
```

## \* 节流

```js
// 每隔 delta 秒时触发
function throttle(fn, delta, context) {
  let safe = true;
  return (...args) => {
    if (!safe) return;
    fn.apply(context, args);
    safe = false;
    setTimeout(() => {
      safe = true;
    }, delta);
  };
}
```

## \* 使用函数结果缓存

```js
// add = userCache((a,b) => a+b);
// add(1,2); add(1,2); // 第二次会直接取缓存
function useCache(fn) {
  const cache = {};
  return function(...args) {
    const key = args.length + JSON.stringify(args);
    if (key in cache) return cache[key];
    else return (cache[key] = fn.apply(this, args));
  };
}
```

## \* px 转 rem 自适应

```js
// 750rem = 100vw
function flexible(remRatio = 750) {
  function setRem() {
    const winW = docEl.getBoundingClientRect().width;
    const fontSize = winW / remRatio;
    docEl.style.fontSize = fontSize;
    $style.innerText = `html{font-size:${fontSize}px !important;}`;
  }
  const win = window,
    doc = document,
    docEl = doc.documentElement,
    $style = doc.createElement('style');
  doc.head.appendChild($style);
  setRem();
  win.addEventListener('resize', setRem, !1);
}
```

## \* 单例模式

```js
// A = singleton(A); var a = new A(x); var b = new A(y);
function singleton(func) {
  let result = undefined;
  return function(...args) {
    if (result) return result;
    result = new (func.bind.apply(func, [null].concat(args)))();
    return result;
  };
}
```

## \* 转 Promise

```js
// const sleep = (delay) => new Promise(resolve => setTimeout(resolve, delay));
// 可改为以下方式 const sleep = promisify((delay, next) => setTimeout(next, delay));
function promisify(fn) {
  return function(...args) {
    const that = this;
    return new Promise(resolve => {
      args[args.length] = resolve;
      args.length += 1;
      fn.apply(that, args);
    });
  };
}
```

## \* 下载

```js
// download('base64', 'xx.jpg', 'image/jpeg')
// download('words', 'xx.txt', 'text/plain')
function download(...args) {
  const scriptId = 'downloadScript';

  if (!document.getElementById(scriptId)) {
    const script = document.createElement('script');
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

## \* 下载 Excel

```js
function downloadExcel(url, callback, options = {}) {
  onError = options.error;
  const xhr = new XMLHttpRequest();
  xhr.open('GET', url, true);
  xhr.responseType = 'blob';
  xhr.onload = function() {
    const res = this.response;
    const isBlob = typeOf(res) === 'Blob';
    if (+this.status === 200 && isBlob) {
      let filename = options.filename;
      if (!filename) {
        const dis = (filename = xhr.getResponseHeader('Content-Disposition'));
        filename = dis.match(/filename=(.*)/i)[1];
        filename = filename ? decodeURIComponent(filename) : '新建文件.xlsx';
      }
      download(res, filename, 'application/octet-stream');
      callback && callback();
    } else {
      onError && onError(res, this);
    }
  };
  xhr.onerror = onError && onError;
  xhr.send();
}
```

## \* 复制文本

```js
function copyText(text) {
  let $input = document.createElement('textarea');
  document.body.appendChild($input);
  $input.value = text;
  $input.select();
  document.execCommand('Copy');
  document.body.removeChild($input);
  $input = null;
}
```

## \* 获取链接信息

```js
function getLocationData(url) {
  let obj = { href: url };
  if (/^https?/.test(url)) {
    url.replace(/(https?:)\/\/([^\/]*?)(\/.*)/, function(match, protocol, host, pathname) {
      const [hostname, port] = host.split(':');
      const origin = protocol + '//' + host;
      obj = { ...obj, protocol, host, hostname, port, origin, pathname };
    });
  } else {
    obj = { ...obj, protocol: null, host: null, hostname: null, prot: null, origin: null, pathname: url };
  }
  obj.pathname = obj.pathname.replace(/\?([^#]*)/, function(match, search) {
    obj.search = search || '';
    return '';
  });
  obj.pathname = obj.pathname.replace(/#([^\?]*)/, function(match, hash) {
    obj.hash = hash || '';
    return '';
  });
  return obj;
}
```

## \* 滚动加载数据

```js
function divideDataForScroll($scroller, data, callback, options) {
  options = options || {};
  const size = options.size || 20;
  const winH = $scroller.offsetHeight;
  const threshold = options.threshold || 50;

  $scroller.removeEventListener('scroll', myScroll);
  $scroller.addEventListener('scroll', myScroll);

  let page = 1;
  myScroll();

  function myScroll() {
    const elemH = $scroller.scrollHeight;
    const sTop = $scroller.scrollTop;
    if (sTop + winH + threshold > elemH) touchBottom();
  }

  function touchBottom() {
    const nowSize = page * size;
    const nowData = data.slice(0, nowSize);

    // 数据全部加载完毕
    if (data.slice(nowSize - size, nowSize).length < 1) {
      options.finish && options.finish(data, page, size); // 全部完成
      return $scroller.removeEventListener('scroll', myScroll);
    }

    // 每页的回调
    callback && callback(nowData, page, size);
  }
}
```

## \* 次数拦截器

```js
// var im = new Util.InterceptManage();
// im.set('listOk', 2, () => console.log('ok'));
// if (!im.ok('listOk')) return;
// if (!im.ok('listOk')) return; // 两次跑完后才走 finish
function InterceptManage() {
  var temp = {};
  return {
    data: temp,
    set: function(key, times, finish) {
      temp[key] = {
        times: times,
        finish: finish
      };
      return temp[key];
    },
    ok: function(key) {
      var item = temp[key];
      if (!item) return false;
      if (--item.times < 1) {
        item.finish && item.finish();
        return true;
      } else return false;
    },
    remove: function(key) {
      delete temp[key];
    }
  };
}
```

## \* 动画类

```js
function Animation() {
  let animTimer = 0;
  function start(start, to, duration, callback) {
    const time = Date.now();
    stop();
    (function run() {
      const per = Math.min(1, (Date.now() - time) / duration);
      if (per >= 1) return callback && callback(to, 1);
      const now = start + (to - start) * per;
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
  };
}
```

## \* 生成二维码 `$`

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
      img.onload = function() {
        render(img);
      };
      img.src = options.image;
      delete options.image;
    } else {
      render();
    }
  }

  function render(image) {
    var opts = $.extend(
      {
        text: text,
        size: options.width || 400, // width height 竟然没效
        mode: 4,
        image: image,
        mSize: 0.2, // 内部图尺寸
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
      },
      options.config || {}
    );
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
      img.onload = function() {
        callback && callback($(this));
      };
      img.src = base64;
      return false;
    }
    callback && callback($img);
  }
}
```

## \* 手机摇一摇

```js
function PhoneShake(func, options) {
  if (typeOf(func) === 'object') {
    options = func;
    func = undefined;
  }

  options = options || {};
  var SHAKE_THRESHOLD = options.threshold || 300;
  var last_update = 0;
  var x, y, z, last_x, last_y, last_z;
  var shakecount = 0;

  function deviceMotionHandler(e) {
    var acceleration = e.accelerationIncludingGravity;
    var curTime = new Date().getTime();
    if (curTime - last_update > 300) {
      var diffTime = curTime - last_update;
      last_update = curTime;
      (x = acceleration.x), (y = acceleration.y), (z = acceleration.z);
      var speed = (Math.abs(x + y + z - last_x - last_y - last_z) / diffTime) * 10000;
      if (speed > SHAKE_THRESHOLD) {
        if (func) func(++shakecount);
      }
      (last_x = x), (last_y = y), (last_z = z);
    }
  }

  function start() {
    if (!window.DeviceMotionEvent) return alert('很抱歉，您的手机设备不支持摇一摇！');
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
  };
}
```

## \* 转为 Hash 数字

```js
function toHashCode(str) {
  var hash = 0;
  if (str.length == 0) return hash;
  for (i = 0; i < str.length; i++) {
    char = str.charCodeAt(i);
    hash = (hash << 5) - hash + char;
    hash = hash & hash;
  }
  return hash;
}
```

## \* 偏函数

```js
// function log(time, type, message) {};  todayLog(new Date(), 'error', '报错')
// var todayLog = partial(log, new Date()); todayLog('error', '报错')
// var todayErrorLog = partial(todayLog, 'error'); todayErrorLog('报错')
function partial(fn, ...rawArgs) {
  return function(...args) {
    return func.call(this, ...rawArgs, ...args);
  };
}
```

## \* 函数科里化

```js
// 和偏函数类似，但将入参分批传入，只要实参数量与形参数量一致
// var todayErrorLog = currying(log)(new Date())('error'); todayErrorLog('报错')
function currying(func, ...rawArgs) {
  return function curried(...args) {
    if (args.length >= func.length) {
      return func.call(this, ...rawArgs, ...args);
    } else {
      return function(...args2) {
        return curried.call(this, ...rawArgs, ...args, ...args2);
      };
    }
  };
}
```

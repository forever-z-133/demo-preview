# 各种判断函数

## * 数字相关
```js
// 是合法数字
const isNumber = (num) => !isNaN(parseFloat(num);

// 是完整的数字
// 比如输入了 1. 或 - 还尚不完整
const isValidNumber = (num) => /\d$/.test(num);

// 是正数
const isPositive = (num) => num > 0;

// 是整数
const isInteger = (num) => Math.floor(num) === num;

// 是小数
const isDecimal = (num) => !isInteger(num);

// 不大于几位小数的小数
// 比如 underDecimalLength(1.234, 2); // false
const underDecimalLength = (num, len = 2) => {
  const numStr = String(num);
  const total = numStr.length;
  const integer = numStr.indexOf('.') + 1;
  return total - integer <= len;
}
```

## * 设备相关
```js
const ua = navigator.userAgent;

// 微信浏览器
const isWeChat = /MicroMessenger/i.test(ua);
// 微信开发者工具
const isWeChat = isWeChat() && /wechatdevtools/i.test(ua);

// 支付宝浏览器
const isAliPay = /AlipayClient/i.test(ua);

// 小于 IE9
const ltIE9 = !+"\v1";
```

## * 其他常用
```js
// 字符串长度不大于
const underLength = (str, len) => [str].toString().length <= len;
```

## * 常见正则
```js
// 手机号
const phone_reg = /^1\d{10}$/;
// 邮箱
const email_reg = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(.[a-zA-Z0-9_-])+/;
// 密码，必须包含数字和字母，长度 6-30
const password_reg = /(?=.*[0-9]|.*[a-zA-Z]).{6,30}/;
// 带 css 单位的数字
const css_number_reg = /^(-?\d*?\.?\d+)(px|%|r?em|vw|vh|pt)$/;
// /**/ 注释
const comment_reg = /\/\/*(.|\n)*?\*\//;
```
# 不常用的判断函数

## * 是合法数字
```js
function isNumber(num) {
  return !isNaN(parseFloat(num));
}
```

## * 是正数
```js
function isPositive(num) {
  return isNumber() && num > 0;
}
```

## * 是整数
```js
function isInteger(num) {
  return num >> 0 == num;
}
```

## * 是什么设备
```js
function isWeChat() { // 微信浏览器
  return /MicroMessenger/i.test(navigator.userAgent);
}
function isWeChatDevTool() { // 微信开发者工具
  return isWeChat() && /wechatdevtools/i.test(navigator.userAgent);
}
function isAliPay() { // 支付宝浏览器
  return /AlipayClient/i.test(navigator.userAgent);
}
```

## * 是手机号
```js
function isPhoneNumber(num) {
  return /^1\d{10}$/.test(num);
}
```

## * 是邮箱
```js
function isEmail(str) {
  return /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(.[a-zA-Z0-9_-])+/.test(str);
}
```
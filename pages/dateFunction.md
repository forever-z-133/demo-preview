# 时间相关的公共方法

## * 日期转为字符串
```js
function dateToString(date, format, noZero) {
  format = format || 'yyyy-MM-dd';
  noZero = noZero || false;
  var d = new Date(date);
  var result = format;
  var _config = {
    'M+': d.getMonth() + 1, // 月
    'd+': d.getDate(), // 日
    'h+': d.getHours(), // 小时
    'm+': d.getMinutes(), // 分
    's+': d.getSeconds(), // 秒
  };

  if (/(y+)/.test(result)) { // 年
    var value = d.getFullYear() + '';
    value = value.slice(-RegExp.$1.length);
    result = result.replace(RegExp.$1, value);
  }
  for (var reg in _config) {
    if (!(new RegExp("(" + reg + ")").test(result))) continue;
    var value = _config[reg] + '';
    value = noZero ? value : ('00' + value).slice(-value.length);
    result = result.replace(RegExp.$1, value);
  }

  return result;
}
```

## * 时间加减
```js
// addDate(new Date(2019,5,19,10,40,0), 1); // Thu Jun 20 2019 10:40:00
function addDate(date, time, dateType) {
  dateType = dateType || 'date';
  var d = new Date(date);
  var _config = {
    'year': 'FullYear',
    'month': 'Month',
    'date': 'Date',
    'day': 'Date',
    'hour': 'Hours',
    'minute': 'Minutes',
    'second': 'Seconds'
  }
  var _type = _config[dateType];
  var setFunc = date['set' + _type].bind(d);
  var getFunc = date['get' + _type].bind(d);
  return new Date(setFunc(getFunc() + time));
}
```

## * 清除不需要的时间
```js
// 有时，时分秒会影响计算结果，故有此方法
// 也可用作获取本年首日，本月首日，本天初始的功能
function getSimpleDate(date, dateType) {
  var d = new Date(date);
  dateType = dateType || 'date';
  var _config = {
    'year': '1000000',
    'month': '1100000',
    'date': '1110000',
    'day': '1110000',
    'hour': '1111000',
    'minute': '1111100',
    'second': '1111110'
  }
  _config[dateType].split('').forEach(function (item, index) {
    if (item === '1') return;
    index === 1 && d.setMonth(0);
    index === 2 && d.setDate(1);
    index === 3 && d.setHours(0);
    index === 4 && d.setMinutes(0);
    index === 5 && d.setSeconds(0);
    index === 6 && d.setMilliseconds(0);
  });
  return new Date(d);
}
```

## * 本月有多少天
```js
function getDayNumberInThisMonth(date, month, year) {
  month = (month || date.getMonth()) % 11;
  var tempYear = month / 11 >> 0; // month 可能会超出 11 则再加 1 年
  year = (year || date.getFullYear()) + tempYear;
  var m = [31, null, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
  m[1] = (year % 4 === 0 && year % 100 !== 0 || year % 400 === 0) ? 29 : 28;
  return m[month];
}
```

## * 获取某阶段的首日
```js
// 如，每周首日，每月首日
function getFirstDate(date, dateType, offset) {
  offset = offset || 0;
  if (dateType !== 'week') {
    var d = getSimpleDate(date, dateType);
    return addDate(d, offset, 'date');
  } else { // 注，每周首日为周一，可用 offset 调节
    var d = new Date(date);
    var over = -1 * d.getDay() + 1 + offset;
    return addDate(d, over, 'date');
  }
}
```

## * 返回两日期间所有日期的数组
```js
// 前开后闭区间，注意时分秒会影响计算结果
// getArrayFromTwoDate(new Date(2019,5,19,11), new Date(2019,5,21,10)) // [20,21]
function getArrayFromTwoDate(a, b) {
  var result = [];
  var daySecond = 24 * 60 * 60 * 1000;
  var start = a < b ? a : b;
  var target = a < b ? b : a;

  while (start < target) {
    start = new Date(+start + daySecond);
    result.push(start);
  }

  return result;
}
```
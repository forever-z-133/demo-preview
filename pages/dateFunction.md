# 日期相关的公共方法

* [dateToString](#-日期转为字符串)（日期转为字符串）
* [addDate](#-日期加减)（日期加减）
* [getSimpleDate](#-返回初始日期)（返回初始日期）
* [getDayNumberInThisMonth](#-本月多少天)（本月多少天）
* [getFirstDate](#-获取首日)（获取首日）
* [getArrayFromTwoDate](#-两日期间所有日期的数组)（两日期间所有日期的数组）
* [getPastDateString](#-返回已过去时间)（返回已过去时间）
* [getWeekName](#-获取星期几)（获取星期几）

## * 日期转为字符串
```js
function dateToString(date, format) {
  format = format || 'yyyy-MM-dd';
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
    value = value.slice(-RegExp.$1.length); // 非常不推荐只显示两位的年数
    result = result.replace(RegExp.$1, value);
  }
  for (var reg in _config) {
    if (!(new RegExp("(" + reg + ")").test(result))) continue;
    var value = _config[reg] + '';
    value = RegExp.$1.length < 2 ? value : ('00' + value).slice(-value.length);
    result = result.replace(RegExp.$1, value);
  }

  return result;
}
```

## * 日期字符串转日期
```js
function stringToDate(str, format) {

}
```

## * 日期转简单对象
```js
// 实为生活理解上的日期，不推荐用此数据去计算
function dateToObject(date) {
  return {
    year: date.getFullYear(),
    month: date.getMonth() + 1, // 一月为 1
    date: date.getDate(),
    hour: date.getHours(),
    minute: date.getMinutes(),
    second: date.getSeconds(),
    day: (date.getDay() + 7) % 7, // 周一为 1，周日为 7
    quarter: 1 + date.getMonth() / 3 >> 0, // 季度
    week: 2 + (date.getDate() - date.getDay()) / 7 >> 0,  // 本月第几周
  }
}
```

## * 日期加减
```js
// addDate(new Date(2019,5,19,10,40,0), 1); // Thu Jun 20 2019 10:40:00
function addDate(date, offset, dateType) {
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
  return new Date(setFunc(getFunc() + offset));
}
```

## * 返回初始日期
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

## * 本月多少天
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

## * 获取首日
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

## * 两日期间所有日期的数组
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

## * 返回已过去时间
```js
// 此方法并不标准，需按业务情景进行调整
// getPastDateString(new Date()); // '刚刚'
function getPastDateString(date, options) {
  options = options || {};
  var time = +new Date(date);
  var now = +new Date();
  var diff = (now - time) / 1000;

  if (diff < 0) return '';  // 未知时间

  if (diff < 60) {
    return '刚刚'
  } else if (diff < 60*60) {
    return (diff/60>>0) + '分钟前';
  } else if (diff < 24*60*60) {
    return (diff/3600>>0) + '小时前';
  } else if (diff < 2*24*60*60) {
    return '一天前';
  } else {
    return dateToString(time, 'yyyy-MM-dd');
  }
}
```

## * 获取星期几
```js
// getWeekName(0, null, 1);  offset 可让 0 为周一
function getWeekName(date, strType, offset) {
  var _config = '日,一,二,三,四,五,六'.split(',');
  if (strType === 1) _config = '周日,周一,周二,周三,周四,周五,周六'.split(',');
  if (strType === 2) _config = '星期日,星期一,星期二,星期三,星期四,星期五,星期六'.split(',');
  if (strType === 3) _config = 'Sunday,Monday,Tuesday,Wednesday,Thursday,Friday,Saturday'.split(',');
  if (strType === 4) _config = 'Sun,Mon,Tue,Wed,Thu,Fri,Sat'.split(',');
  if (typeOf(strType) === 'array') _config = strType;

  offset = offset || 0;
  var day = typeOf(date) === 'date' ? date.getDay() : date;
  day = (day + offset) % 6;

  return _config[day];
}
```
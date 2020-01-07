# 颜色相关的公共方法

* [typeOfColor](#-颜色类型)（颜色类型）

## * 公共的正则
```js
const color_num_unit_reg = '(?![3-9]..|2[6-9].|25[6-9])\d{1,3}'; // 0-255
const color_hex_unit_reg = '[a-fA-f0-9]'; // 0-F
const hex_3_reg = new RegExp(`#(${color_hex_unit_reg}{3,4})`);
const hex_6_reg = new RegExp(`#(${color_hex_unit_reg}{6})(${color_hex_unit_reg}{2})?`);
const rgb_reg = new RegExp(`rgba?\(\s*([+-]?\d+)\s*,\s*([+-]?\d+)\s*,\s*([+-]?\d+)\s*(?:,\s*([+-]?[\d\.]+)\s*)?\)`);
```

## * 颜色类型
```js
function typeOfColor(str) {
  const prefix = str.slice(0, 4);
  
}
```

## 借鉴
https://github.com/Qix-/color-string  
https://github.com/Qix-/color-convert  
https://github.com/colorjs/color-name  
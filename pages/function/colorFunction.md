# 颜色相关的公共方法

* [rgb2hex](#RGB-转-HEX)（RGB 转 HEX）
* [hex2rgb](#HEX-转-RGB)（HEX 转 RGB）

## * 公共的正则
```js
const color_num_unit_reg = '(?![3-9]..|2[6-9].|25[6-9])\d{1,3}'; // 0-255
const color_hex_unit_reg = '[a-fA-f0-9]'; // 0-F
const hex_3_reg = new RegExp(`#(${color_hex_unit_reg}{3,4})`);
const hex_6_reg = new RegExp(`#(${color_hex_unit_reg}{6})(${color_hex_unit_reg}{2})?`);
const rgb_reg = new RegExp(`rgba?\(\s*([+-]?\d+)\s*,\s*([+-]?\d+)\s*,\s*([+-]?\d+)\s*(?:,\s*([+-]?[\d\.]+)\s*)?\)`);
```

## RGB 转 HEX
```js
function rgb2hex(r, g, b) {
  return ((1 << 24) + (r << 16) + (g << 8) + b).toString(16).slice(1);
}
```

## HEX 转 RGB
```js
function hex2rgb(hex) {
  var result = /^(#|0x)?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
  return result ? {
    r: parseInt(result[1], 16),
    g: parseInt(result[2], 16),
    b: parseInt(result[3], 16)
  } : null;
}
```

## 借鉴
https://github.com/Qix-/color-string<br />
https://github.com/Qix-/color-convert<br />
https://github.com/colorjs/color-name<br />
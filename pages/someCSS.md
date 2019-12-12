# 常用的 CSS 原子类

可直接查看下方链接：
https://github.com/forever-z-133/rebuild-panda/tree/master/src/themes/common

```css
.scroller {
  max-height: 100%;
  overflow: auto;
  -webkit-overflow-scrolling: touch;
}
```

```less
/* less 语法 */
/* 结果： .gap-right-10 > :not(:last-child) { margin-right: 10px; } */
.margin-gap-loop(@n, @i:1, @dir:right) when (@i <= @n) {
  @gap: @i * 5;
  .gap-@{dir}-@{gap} > :not(:last-child) {
    margin-@{dir}: @gap * 1px;
  }
  .margin-gap-loop(@n, (@i + 1), @dir);
}
.margin-gap-loop(6, 1, right);
.margin-gap-loop(6, 1, bottom);
```

```scss
.disabled {
  pointer-events: none;
  user-select: none;
  &.gray { filter: grayscale(1); }
}
```

```scss
.flex-row {
  display: flex;
  align-items: center;
  &.t { align-items: flex-start; }
  &.s { align-items: stretch; }
  &.b { align-items: flex-end; }
  &.c { justify-content: center; }
  &.r { justify-content: flex-end; }
  &.w { flex-wrap: wrap; }
  & > .grow { flex-grow: 1; overflow: hidden; }
  & > :not(.grow) { flex-shrink: 0; }
}
.flex-col {
  display: flex;
  flex-direction: column;
  & > .grow { flex-grow: 1; overflow: hidden; }
  & > :not(.grow) { flex-shrink: 0; }
}
.flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}
```

```scss
.inblock-row {
  letter-spacing: -1em;
  & > * {
    display: inline-block;
    letter-spacing: 0;
  }
}
.float-row {
  overflow: hidden;
  & > * {
    float: left;
  }
}
```

```scss
.cover { position: absolute; top: 0; left: 0; right: 0; bottom: 0; }
.fitCover { position: absolute; top: 0; left: 0; width: 100%; height: 100%; object-fit: cover; }
.pos-center { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); }
```

```scss
.ratio { position: relative; }
.ratio::before { content: ''; float: left; padding-top: 100%; }
.ratio::after { content: ''; display: block; clear: both; }

.imgRatio { position: relative; }
.imgRatio::before { content: ''; float: left; padding-top: 100%; }
.imgRatio > * { .fitCover; }
```

```scss
label.form-file {
  position: relative;
  vertical-align: middle;
  cursor: pointer;
  & > input[type="file"] { position: absolute; left: -999em; }
}
```

```scss
@mixin textOverflow($line) {
  @if $line==1 {
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
  } @else {
    overflow: hidden;
    text-overflow: ellipsis;
    word-break: break-all;
    display: -webkit-box;
    -webkit-line-clamp: $line;
    -webkit-box-orient: vertical;
  }
}
```

```css
table {
  background-color: transparent;
  empty-cells: show;
  border-spacing: 0;
  border-collapse: collapse;
}
td, th { padding: 0; }
.list {
  margin-top: 0;
  margin-bottom: 0;
  padding-left: 0;
  list-style: none;
}
```

```scss
@mixin this($sel) {
  @at-root #{if(not &, $sel, selector-append(&, $sel))} {
    @content;
  }
}
```

```scss
// 三角函数系列
@function PI() {
  @return 3.14159265359;
}
@function sin($angle) {
  $sin: 0;
  $angle: rad($angle);
  @for $i from 0 through 10 {
    $sin: $sin + pow(-1, $i) * pow($angle, (2 * $i + 1)) / fact(2 * $i + 1);
  }
  @return $sin;
}
@function cos($angle) {
  $cos: 0;
  $angle: rad($angle);
  @for $i from 0 through 10 {
    $cos: $cos + pow(-1, $i) * pow($angle, 2 * $i) / fact(2 * $i);
  }
  @return $cos;
}
@function tan($angle) {
  @return sin($angle) / cos($angle);
}
```

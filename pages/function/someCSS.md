# 常用的 CSS 原子类

有兴趣可查看下方链接的源码：<br />
https://www.npmjs.com/package/common-scss-mixin<br />
https://www.npmjs.com/package/common-less-mixin

```css
.scroller {
  max-height: 100%;
  overflow: auto;
  -webkit-overflow-scrolling: touch;
}
```

```scss
.disabled {
  pointer-events: none;
  user-select: none;
  &.gray {
    filter: grayscale(1);
  }
}
```

```scss
.flex-row {
  display: flex;
  align-items: center;
  &.t {
    align-items: flex-start;
  }
  &.m {
    align-items: center;
  }
  &.b {
    align-items: flex-end;
  }
  &.s {
    align-items: stretch;
  }
  &.l {
    justify-content: flex-start;
  }
  &.c {
    justify-content: center;
  }
  &.r {
    justify-content: flex-end;
  }
  &.sb {
    justify-content: space-between;
  }
  &.w {
    flex-wrap: wrap;
  }
  & > .grow {
    flex-grow: 1;
    overflow: hidden;
  }
  & > :not(.grow) {
    flex-shrink: 0;
  }
}
.flex-col {
  display: flex;
  flex-direction: column;
  &.t {
    justify-content: flex-start;
  }
  &.m {
    justify-content: center;
  }
  &.b {
    justify-content: flex-end;
  }
  &.sb {
    justify-content: space-between;
  }
  &.l {
    align-items: flex-start;
  }
  &.c {
    align-items: center;
  }
  &.r {
    align-items: flex-end;
  }
  & > .grow {
    flex-grow: 1;
  }
  & > :not(.grow) {
    flex-shrink: 0;
  }
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

```css
.cover {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}
.fit-cover {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.pos-center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.pos-hide {
  position: absolute;
  left: -9999em;
}
```

```less
.ratio {
  position: relative;
}
.ratio::before {
  content: '';
  float: left;
  padding-top: 100%;
}
.ratio::after {
  content: '';
  display: block;
  clear: both;
}

.image-ratio {
  position: relative;
}
.image-ratio::before {
  content: '';
  float: left;
  padding-top: 100%;
}
.image-ratio > * {
  .fit-cover();
}
```

```less
label.form-file {
  position: relative;
  vertical-align: middle;
  cursor: pointer;
  & > input[type='file'] {
    .pos-hide();
  }
}
```

```scss
.text-overflow {
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}
.text-overflow-2 {
  overflow: hidden;
  text-overflow: ellipsis;
  word-break: break-all;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}
```

```css
table {
  background-color: transparent;
  empty-cells: show;
  border-spacing: 0;
  border-collapse: collapse;
}
td,
th {
  padding: 0;
}
.list {
  margin-top: 0;
  margin-bottom: 0;
  padding-left: 0;
  list-style: none;
}
```

```css
@font-face {
  font-family: 'color-emoji';
  src: local('Apple Color Emoji'), local('Segoe UI Emoji'), local('Segoe UI Symbol'), local('Noto Color Emoji');
  unicode-range: U+1F000-1F644, U+203C-3299; /* 需根据实际情况进行调整 */
}
```

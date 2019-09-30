# 常用的 CSS 原子类

```css
.scroller {
  height: 100%;
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
  &.gray {
    filter: grayscale(1);
  }
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
```

```scss
.cover { position: absolute; top: 0; left: 0; right: 0; bottom: 0; }
.fitCover { position: absolute; top: 0; left: 0; width: 100%; height: 100%; object-fit: cover; }
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
  overflow: hidden;
  cursor: pointer;
  & > input[type="file"] { position: absolute; left: -999em; }
}
```

```scss
.textOverflow { text-overflow: ellipsis; overflow: hidden; white-space: nowrap; }

/* scss 语法 */
@mixin line-clamp($line) {
  overflow: hidden;
  text-overflow: ellipsis;
  word-break: break-all;
  display: -webkit-box;
  -webkit-line-clamp: @line;
  -webkit-box-orient: vertical;
}
.textOverflow2 { @include line-clamp(2); };
.textOverflow3 { @include line-clamp(3); };
.textOverflow4 { @include line-clamp(4); };
```
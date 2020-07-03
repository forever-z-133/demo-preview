常见而公用的 less 的 mixin 方法，且只有 mixin 方法，不会产生不必要的 css 导出。

## 1、如何使用

```
npm i -D common-less-mixin
```
https://www.npmjs.com/package/common-less-mixin

### 使用 scss

```less
// xxx.less
@import "~common-less-mixin";

.someClass {
  .pos-top(); /* 顶部定位 */
  .flex-row(); /* flex 行且水平居中 */
  .padding-row(); /* 两边加上 15px 的内边距 */
  .child-gap-right(); /* 子元素之间距离为 5px */
  .border-bottom(); /* 底部毛细线 */
}
```

### 使用 css

```less
// 也有打包好的 css 可供使用，但类名会有所不同
// 如果喜欢本项目的话，可通过源码进一步尝试
@import "~common-less-mixin/dist.min.css";
```

```html
<div class="post-top flex-row padding-row gap-right-5 border-bottom"></div>
```

### 使用单个 scss

```less
// 如果担心编译太多 mixin 会影响性能，可分别引入
@import "~common-less-mixin/src/border.scss";
@import "~common-less-mixin/src/flex.scss";
@import "~common-less-mixin/src/form.scss";
@import "~common-less-mixin/src/gap.scss";
@import "~common-less-mixin/src/layout.scss";
@import "~common-less-mixin/src/others.scss";
@import "~common-less-mixin/src/position.scss";
@import "~common-less-mixin/src/text.scss";
@import "~common-less-mixin/src/utils.scss";
@import "~common-less-mixin/src/var.scss";

// 单个 css 则可以自己打包
@import "~common-less-mixin/src/flex.scss";
@import "~common-less-mixin/src/gap.scss";
.flex-build();
.gap-build();
```

## 2、如何自定义

```less
// common.less
@import "~common-less-mixin";
@px: 1rem;

// 使用时
@import "common.less";
```

可配置项一览：

```less
@px: 1px;

@gap-xs: 5 * @px;
@gap-sm: 10 * @px;
@gap-md: 15 * @px;
@gap-xl: 30 * @px;
@row-gap: @gap-md;
@child-gap: @gap-xs;

@radius-md: 4 * @px;
@radius-round: 1000em;
@radius-circle: 50%;

@border-width: 1PX;
@border-color: #e8e8e8;

// 当在小程序上使用时不支持 > :not() 和 *，故而可通过以下配置改为 view
@flex-grow-children: ~".grow";
@flex-shrink-children: ~"";
@flex-column-grow-children: ~".item", ~".col";
@float-row-children: ~"*";
@inblock-row-children: ~"*";
@child-gap-children: ~"";
@child-gap-children-not: ~":last-child";
@input-file-children: ~'[type="file"]';
@image-ratio-children: ~"*";
```

## 3、总览

```less
/* border.scss */
.border(@color, @width);
.border-top(@color, @width);
.border-left(@color, @width);
.border-right(@color, @width);
.border-bottom(@color, @width);
.raduis(@rdius, @overflow);
.radius-round(@overflow);
.radius-circle(@overflow);

/* gap.scss */
.padding-row(@gap);
.padding-col(@gap);
.margin-row(@gap);
.margin-col(@gap);
.margin-center();
.child-gap-right(@gap);
.child-gap-bottom(@gap);

/* flex.scss */
.flex-row-normal();
.flex-row(@children);
.flex-row-top();
.flex-row-middle();
.flex-row-bottom();
.flex-row-stretch();
.flex-row-left();
.flex-row-center();
.flex-row-right();
.flex-row-space();
.flex-row-wrap();

.flex-col-normal();
.flex-col(@children);
.flex-col-top();
.flex-col-middle();
.flex-col-bottom();
.flex-col-space();
.flex-col-left();
.flex-col-center();
.flex-col-right();

.flex-center();
.flex-column(@column, @children);

/* layout.scss */
.inblock-row();
.float-row();
.scroller(@dir);
.scroller-x();
.scroller-y();

/* form.scss */
.input-file(@children);
.pure();

/* position.scss */
.cover(@type);
.fit-cover(@fit);
.pos-center(@type);
.pos-top(@type);
.pos-bottom(@type);
.pos-left(@type);
.pos-right(@type);
.pos-top-left(@type, @offset);
.pos-top-right(@type, @offset);
.pos-bottom-left(@type, @offset);
.pos-bottom-right(@type, @offset);
.pos-hide();

/* text.scss */
.text-overflow(@line);
.text-last-justify(@height);

/* others.scss */
.reset();
.normal-list();
.clearfix();
.rect-box(@size);
.disabled(@grey);
.ratio(@ratio);
.image-ratio(@ratio, @fit, @children);
.seo-only();
```

常见而公用的 scss 的 mixin 方法，且只有 mixin 方法，避免无端增加打包体积。

## 1、如何使用

```
npm i -D common-scss-mixin
```
https://www.npmjs.com/package/common-scss-mixin

### 使用 scss

```scss
// xxx.scss
@import 'common-scss-mixin';

.someClass {
  @include pos-top(fixed); /* 顶部定位 */
  @include flex-row(); /* flex 行且垂直居中 */
  @include padding-row(); /* 两边加上 15px 的内边距 */
  @include child-gap-right(); /* 子元素之间间隙为 10px */
  @include border-bottom(); /* 底部毛细线 */
}
```

### 使用 css

```scss
// 也有打包好的 css 可供使用，但类名会有所不同
// 如果喜欢本项目的话，可通过源码进一步尝试
@import 'common-scss-mixin/dist.min.css';
```

```html
<div class="fixed-top flex-row padding-row gap-right-5 border-bottom"></div>
```

### 使用单个 scss

```scss
// 如果担心编译太多 mixin 会影响性能，可分别引入
@import 'common-scss-mixin/src/var.scss';
@import 'common-scss-mixin/src/utils.scss';
@import 'common-scss-mixin/src/border.scss';
@import 'common-scss-mixin/src/flex.scss';
@import 'common-scss-mixin/src/form.scss';
@import 'common-scss-mixin/src/gap.scss';
@import 'common-scss-mixin/src/layout.scss';
@import 'common-scss-mixin/src/others.scss';
@import 'common-scss-mixin/src/position.scss';
@import 'common-scss-mixin/src/text.scss';

// 单个 css 则可以自己打包
@import 'common-scss-mixin/src/flex.scss';
@import 'common-scss-mixin/src/position.scss';
@include flex-build();
@include position-build();
```

### 全局使用 scss

不想每个地方都去 @import，可使用 sass-loader 配置,
让其加在编译前达到全局使用的效果，只管 @include 即可。

```js
// 以 vue-cli 的 vue.config.js 举例:
module.exports = {
  css: {
    loaderOptions: {
      sass: {
        // 当 sass-loader 版本低于 7.0 时 prependData 需改为 data
        prependData: "@import 'common-scss-mixin';"
      }
    }
  }
};
```

## 2、如何自定义

```scss
// some/common.scss;
@import 'common-scss-mixin';
@function px($n) {
  @return $n * 1rem;
}

// 使用
@import 'some/common.scss';
```

可配置项一览：

```scss
@function px($n) { @return $n * 1px; }

$row-gap: px(15);
$col-gap: px(10);
$child-gap: px(10);

$radius-md: px(4);
$radius-round: 1000em;
$radius-circle: 50%;

$border-width: 1PX;
$border-color: #e8e8e8;

// 譬如小程序不支持 '*' 则可改为 'view', 'text' 等
// 且 children 代表可设配多个子类
$flex-grow-children: '.grow';
$flex-shrink-children: '';
$flex-column-grow-children: '.item', '.col';
$float-row-children: '*';
$inblock-row-children: '*';
$child-gap-children: '';
$child-gap-children-not: ':last-of-type';
$input-file-children: '[type="file"]';
$image-ratio-children: '*';
```

## 3、总览

```scss
// 注意 $children 想传入数组类型时需用 #{}，比如 @include xx(#{'view', 'text'});

/* border.scss */
@include border($color, $width);
@include border-top($color, $width);
@include border-left($color, $width);
@include border-right($color, $width);
@include border-bottom($color, $width);
@include raduis($rdius, $overflow);
@include radius-round($overflow);
@include radius-circle($overflow);

/* gap.scss */
@include padding-row($gap);
@include padding-col($gap);
@include margin-row($gap);
@include margin-col($gap);
@include margin-center();
@include child-gap-right($gap, $children);
@include child-gap-bottom($gap, $children);

/* flex.scss */
@include flex-row-normal();
@include flex-row($children);
@include flex-row-top();
@include flex-row-middle();
@include flex-row-bottom();
@include flex-row-stretch();
@include flex-row-left();
@include flex-row-center();
@include flex-row-right();
@include flex-row-space(); // space-between 贴边
@include flex-row-wrap();

@include flex-col-normal();
@include flex-col($children);
@include flex-col-top();
@include flex-col-middle();
@include flex-col-bottom();
@include flex-col-space();
@include flex-col-left();
@include flex-col-center();
@include flex-col-right();

@include flex-center();
@include flex-column($column, $children);

/* layout.scss */
@include inblock-row($children);
@include float-row($children);
@include scroller($dir);
@include scroller-x();
@include scroller-y();

/* form.scss */
@include input-file();
@include pure();

/* position.scss */
@include cover($type);
@include fit-cover($fit);
@include pos-center($type);
@include pos-top($type);
@include pos-bottom($type);
@include pos-left($type);
@include pos-right($type);
@include pos-top-left($type, $offset);
@include pos-top-right($type, $offset);
@include pos-bottom-left($type, $offset);
@include pos-bottom-right($type, $offset);
@include pos-hide();

/* text.scss */
@include text-overflow($line);
@include text-last-justify($height);

/* others.scss */
@include reset();
@include normal-list();
@include clearfix();
@include rect-box($size);
@include disabled($grey);
@include ratio($ratio);
@include image-ratio($ratio, $fit, $children);
@include seo-only();

/* utils.scss */
@mixin child-map($child, $not) { $content }
sin($angle);
cos($angle);
tan($angle);
```

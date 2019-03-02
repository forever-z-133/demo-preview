# 三种子级均匀留隙的实现

## 一、flex + space-between
```css
.wrap {
  display: flex;
  justify-content: space-between
}
```

## 二、flex + margin: auto
```css
.wrap { display: flex; }
.wrap > :not:(:last-child) {
  margin-right: auto;
}
```

## 三、grid
```css
```

## 其他、inline-block + text-align

类似文字的做法，可看 [案例](./pages/text-align-justify)
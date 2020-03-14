# DOM 相关的公共方法

```js
function matchesSelector(element, selector) {
  if (typeof selector != 'string') return false;
  const nativeMatches = window.Element.prototype.matches;
  if (nativeMatches) return nativeMatches.call(element, selector);
  const nodes = element.parentNode.querySelectorAll(selector);
  for (let i = 0, node; (node = nodes[i]); i++) {
    if (node == element) return true;
  }
  return false;
}
```

```js
// matches(dom, '.x');
// matches(dom, ['.x', '.y']);
// matches(dom, dom2);
// matches(dom, [dom2, dom3]);
function matches(element, test) {
  if (element && element.nodeType == 1 && test) {
    if (typeof test == 'string' || test.nodeType == 1) {
      return element == test || matchesSelector(element, test);
    } else if ('length' in test) {
      for (let i = 0, item; (item = test[i]); i++) {
        if (element == item || matchesSelector(element, item)) return true;
      }
    }
  }
  return false;
}
```

```js
function parents(element) {
  const list = [];
  while (element && element.parentNode && element.parentNode.nodeType == 1) {
    element = element.parentNode;
    list.push(element);
  }
  return list;
}
```

```js
function closest(element, selector, checkSelf = false) {
  if (!(element && element.nodeType == 1 && selector)) return;
  const parentElements = (checkSelf ? [element] : []).concat(parents(element));
  for (let i = 0, parent; (parent = parentElements[i]); i++) {
    if (matches(parent, selector)) return parent;
  }
}
```

```js
// 原方法获得的是 map 结构，改为 {attr:value} 简单对象结构
function getAttributes(element) {
  const attrs = {};
  if (!(element && element.nodeType == 1)) return attrs;
  const map = element.attributes;
  if (map.length === 0) return {};
  for (let i = 0, attr; (attr = map[i]); i++) {
    attrs[attr.name] = attr.value;
  }
  return attrs;
}
```

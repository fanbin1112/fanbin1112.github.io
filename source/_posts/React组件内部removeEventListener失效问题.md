---
title: React组件内部removeEventListener失效问题
date: 2024-06-03 17:15:44
tags:
---

# 问题复现(示例代码)

```js
const judeScrollIsEnd = useCallback(
  debounce(
    () => {
      // 控制页面滚动时，icon的显隐动画
      if (isScroll.current) {
        floatShrunkRef.current.classList.remove(styles.hidden);
        floatShrunkRef.current.classList.add(styles.visible);
      } else {
        floatShrunkRef.current.classList.remove(styles.visible);
        floatShrunkRef.current.classList.add(styles.hidden);
      }
      isScroll.current = !isScroll.current;
    },
    scrollWaitTime,
    true,
    true,
  ),
  [],
);

useEffect(() => {
  window.addEventListener('scroll', judeScrollIsEnd);
}, []);

useEffect(() => {
  if (!isShowIcon) {
    window.removeEventListener('scroll', judeScrollIsEnd);
  }
}, [isShowIcon]);
```

当 isShowIcon 为 false 时，移除监听事件，但是事件依然会触发。

# 问题解析

通过以下示例代码可以发现，当页面没有 props 或者 state 变化时，点击按钮是可以正常解绑 click 事件的。但当我们使用了 useState,触发了页面渲染时，解绑会失效。

这是因为每当页面渲染时，`clickFunc`作为组件内的方法，会一同重新创建，导致绑定时和解绑时的`clickFunc`内存引用地址不一致。所以 removeEventListener 无法解除绑定, 再次 addEventListener 则会绑定一个新方法。

```js
import React, { useState } from 'react';

const App = () => {
  // const [state, setState] = useState(0)

  const handleAddListening = () => {
    window.addEventListener('click', clickFunc);
    // setState(state + 1)
  };

  const clickFunc = () => {
    console.log('clicking');
  };

  const handleRemoveListening = () => {
    window.removeEventListener('click', clickFunc);
  };

  return (
    <div className="App">
      <div onClick={handleAddListening}>Click to Console</div>
      <div onClick={handleRemoveListening}>Remove Click to Console</div>
    </div>
  );
};
```

# 解决方法

使用无依赖的`useCallback`包裹事件处理函数,解决重渲染问题。

```js
const clickFunc = useCallback(() => {
    console.log("clicking");
  }, []);
```

最近不少国内网站app都置灰了，所以聊一下前端怎么置灰。

置灰主要分三种：全局置灰，首屏置灰和部分置灰。全局置灰比较多；首屏置灰就是第一屏是灰色的，往下滚动又是彩色，比如B站App；在一些场景中国徽或一些重要人物的照片是不能置灰的，因此只能部分置灰。

点击[查看效果](https://gray-page.vercel.app/)。

### 全局置灰

首先要知道大部分页面的置灰是通过CSS中的[filter](https://developer.mozilla.org/zh-CN/docs/Web/CSS/filter)属性实现的。通过设置`filter: grayscale(100%);`可以实现对元素的置灰。为了兼容性，一般这么写：

```css
.gray {
  -webkit-filter: grayscale(100%);
  -moz-filter: grayscale(100%);
  -ms-filter: grayscale(100%);
  -o-filter: grayscale(100%);
  filter: grayscale(100%);
  -webkit-filter: gray;
  filter: gray;
  -webkit-filter: progid:dximagetransform.microsoft.basicimage(grayscale=1);
  filter: progid:dximagetransform.microsoft.basicimage(grayscale=1);
}
```

然后通过js给html标签加上`.gray`。

```js
/**
 * 切换全部置灰
 */
function toggleAllGray() {
  const htmlElement = document.documentElement;
  if (htmlElement.classList.contains("global-gray")) {
    htmlElement.classList.remove("global-gray");
  } else {
    htmlElement.classList.add("global-gray");
  }
}
```

### 首屏置灰

`filter`属性存在性能问题，在一些老机子上有滑动卡顿的情况；虽然可以通过[transform开启硬件加速](https://juejin.cn/post/6844903649974435854)，不过如果只置灰首屏也能符合需求的话终究更好。

[backdrop-filter](https://developer.mozilla.org/zh-CN/docs/Web/CSS/backdrop-filter)属性可以让你为一个元素后面区域添加图形效果（如模糊或颜色偏移）。因为它适用于元素背后的所有元素，为了看到效果，必须使元素或其背景至少部分透明。

思路就是给首屏添加一层遮罩，给该遮罩设置`backdrop-filter`。

```css
.first-gray {
  position: relative;
  width: 100%;
  height: 100%;
}
.first-gray::before {
  content: "";
  position: absolute;
  top: 0;
  width: 100%;
  height: 100%;
  backdrop-filter: grayscale(100%);
  pointer-events: none;
  inset: 0;
  z-index: 100;
}
```

可以看到上面还加了个`pointer-events: none;`，原因在于如果叠加了一层遮罩效果在其上，那这层遮罩下方的所有交互时间都将失效，譬如 hover、click 等。我们给[pointer-events](https://developer.mozilla.org/zh-CN/docs/Web/CSS/pointer-events)设置为`none`，可以让这层遮罩不阻挡事件的点击交互。

### 部分置灰

一开始的想法是给不能置灰的元素添加属性`filter: none;`，但是发现不生效。即父元素设置了`filter: grayscale(100%);`，那么子孙元素怎么设置都无法突破该效果。于是想到用js来手动处理，只对需要置灰的元素置灰。

对于DOM树，首先标记不能置灰的节点，然后从节点到根节点的路径上的所有节点都不能置灰，其余节点可以。于是问题就变成了一道向上遍历树的问题了。

```js
/**
 * 切换部分置灰
 */
function togglePartialGray() {
  const htmlElement = document.documentElement;
  if (htmlElement.classList.contains("partial-gray")) {
    htmlElement.classList.remove("partial-gray");
    return;
  } else {
    htmlElement.classList.add("partial-gray");
  }
  const noGrayElements = document.querySelectorAll(".no-gray");
  // 给从根节点到noGray节点路径上的所有节点添加noGray标记
  for (const el of noGrayElements) {
    // 添加totalNoGray标记，表示在遍历到该节点时不再往下遍历
    el.classList.add("total-no-gray");
    let node = el;
    while (node) {
      node.classList.add("no-gray");
      node = node.parentElement;
    }
  }
  // 从根节点开始往下遍历，对于没做noGray标记的
  dfs(htmlElement);

  /**
   * 递归遍历给节点做标记
   * @param {HTMLElement} node
   * @returns
   */
  function dfs(node) {
    if (!node || node.classList.contains("total-no-gray")) return;
    if (node.classList.contains("no-gray")) {
      for (const child of node.children) {
        dfs(child);
      }
    } else {
      node.classList.add("gray");
    }
  }
}
```
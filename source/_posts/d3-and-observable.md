---
title: d3 and observable
date: 2021-09-11 12:44:09
tags:
---

# Embedding a cell on a web page
有几种嵌入方式，本文只关注通过 JavaScript 方式嵌入 cell.

## 方式一：嵌入 Cell
1. 在 cell 左侧，点击更多按钮（纵向的三个点）
2. 在弹出菜单中，点击 Emebed
3. 在弹出窗口中选择 cell 名称
4. 在弹出窗口中选择 Runtime with JavaScript
5. 复制显示的 JavaScript 到项目中
- 源码在远端，无法修改

## 方式二：下载插件源码
1. 在页面右上方，点击更多按钮（横向的三个点）
2. 在弹出菜单中，点击 Export
3. Download Code
- 插件源码在本地，可以修改

# 渲染 Cell
最显而易见的方式是直接在页面中显示嵌入的 notebook 的内容。Observable runtime 包含一个标准的 inspector，它从 notebook 获取内容并显示在 HTML 中。
```js
// Load the Observable runtime and inspector.
import {Runtime, Inspector} from "https://cdn.jsdelivr.net/npm/@observablehq/runtime@4/dist/runtime.js";

// Your notebook, compiled as an ES module.
import notebook from "https://api.observablehq.com/@jashkenas/my-neat-notebook.js?v=3";

// Load the notebook, observing its cells with a default Inspector
// that simply renders the value of each cell into the provided DOM node.
new Runtime().module(notebook, Inspector.into(document.body));
```

如果不想渲染整个 notebook，可以自定义一个方法选择只渲染哪些 cell，使用 name 来匹配的，如下：
```js
new Runtime().module(notebook, name => {
  if (name === "chart") {
    return new Inspector(document.querySelector("#chart"));
  }
});
```

如果想只加载提供能力，不渲染，当匹配对应的 cell name 时返回 true，如下：
```js
(new Runtime).module(define, name => {
  if (name === "chart") return Inspector.into(".chart")();
  // update 会被 chart 调用
  if (name === "update") return true;
});
```

# 获取 cell 内容
```js
const module = new Runtime().module(notebook);
const value = await module.value("chart");
```

还有一种方式，通过 observer 来获取，例如：
```js
new Runtime().module(notebook, name => {
  return {
    pending() { console.log(`${name} is running…`); },
    fulfilled(value) { console.log(name, value); },
    rejected(error) { console.error(error); }
  };
});
```

# 更新 cell 的内容
通过 [module.redefine](https://github.com/observablehq/runtime/blob/main/README.md#module_redefine) 可以修改 cell 中定义的变量，比如图标的 data/width/height/x/y/margin。
```js
const main = new Runtime().module(define, name => {
  if (name === "chart") return new Inspector(document.querySelector("#observablehq-chart-3ff9aa20"));
});
main.redefine("data", [
	{'name':'apple', value:100},
	{'name':'orange', value:100},	
	]);
main.redefine('height', 100)
```

# Links
1. [Downloading and Embedding](https://observablehq.com/@observablehq/downloading-and-embedding-notebooks)
2. [Observable Examples](https://github.com/observablehq/examples)

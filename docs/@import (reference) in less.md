`@import (reference)` 是 **Less** 预处理器中一个非常强大且实用的指令。

简单来说，它的核心作用是：**静默导入。**

当你在文件 `a.less` 中使用 `(reference)` 标志导入 `b.less` 文件后，`a.less` 文件可以读取并使用 `b.less` 文件里的内容，如果不显式使用，则 `a.less` 经过编译生成的 css 文件中不会包含任何 `b.less` 文件中的内容。

下面详细介绍它的工作原理和核心应用场景。

### 1. 核心工作原理对比

假设我们有一个基础样式文件 `base.less`：

```less
/* base.less */
@primary-color: #1890ff;

.btn-base {
  padding: 10px 20px;
  border-radius: 4px;
  text-align: center;
}
```

#### 常规导入：`@import "base.less";`
如果你在 `main.less` 中常规导入：
```less
/* main.less */
@import "base.less";

.my-button {
  color: @primary-color;
}
```
**编译后的 CSS 输出**会包含 `base.less` 里的所有内容：
```css
/* 编译输出 */
.btn-base {
  padding: 10px 20px;
  border-radius: 4px;
  text-align: center;
}
.my-button {
  color: #1890ff;
}
```

#### 引用导入：`@import (reference) "base.less";`
如果你使用 reference 标志：
```less
/* main.less */
@import (reference) "base.less";

.my-button {
  color: @primary-color;
}
```
**编译后的 CSS 输出**非常干净，只保留你实际写在 `main.less` 里的样式：
```css
/* 编译输出 */
.my-button {
  color: #1890ff;
}
```
`btn-base` 的样式被“吃掉”了，没有造成多余的 CSS 代码冗余。

---

### 2. 核心应用场景

既然不输出代码，那它有什么用呢？对于前端工程化和样式优化来说，这非常关键。

#### 场景一：只使用第三方库的变量和 Mixin
比如你在项目中使用了第三方库的 Less 源码，你可能只想借用里面定义的变量或者某个Mixin，而不希望把整个庞大的 UI 库的基础样式全部打包进你当前的组件代码里。
```less
@import (reference) "path.less";

.my-custom-modal {
  // 直接使用第三方库的变量，但不会引入全量 CSS
  background-color: @gray-lighter; 
}
```

#### 场景二：配合 `:extend` 实现高效的样式复用
这是 `(reference)` 最经典的高阶用法。你可以把公共的 CSS 写在一个文件里，作为 reference 引入，然后通过 Less 的 `&:extend()` 伪类来继承样式。

**举个例子：**
```less
/* tools.less */
.flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

```less
/* component.less */
@import (reference) "tools.less";

.header-container {
  &:extend(.flex-center);
  height: 60px;
}

.footer-container {
  &:extend(.flex-center);
  height: 100px;
}
```

**编译后的极简 CSS：**
```css
/* Less 聪明地将选择器合并了，极大地缩小了 CSS 体积 */
.header-container,
.footer-container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.header-container {
  height: 60px;
}

.footer-container {
  height: 100px;
}
```

### 3. 注意事项
* `@import (reference)` 只能用于 Less 文件，不能用于原生的 `.css` 文件。
* 它可以与其他导入标志组合使用，比如 `@import (reference, multiple) "foo.less";`。

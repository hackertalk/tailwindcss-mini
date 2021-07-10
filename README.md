## 工具优先
tailwindcss 的理念就是 Utility-First（工具优先），其本身提供了很好的 cli 操作，但是由于小程序端限制过多，比如样式不能包含小数点、斜杠、大量 css 选择器不能用等原因， tailwindcss 很难通过配置适配小程序端样式需求，由于 tailwindcss 常用类代码不超过 2000 行，所以我们通过手动提取样式 + scss 工具累类实现 tailwindcss 类似的功能。

## 为什么使用这个方案：
对比xx: 不支持自定义、不支持 divide 等样式，报错，可删减（按需使用），支持不同设计稿（375px or 750px）

## 小程序限制点
- 样式不能包含小数点：使用字母 `d` 替代，如：`pt-1.5` => `pt-1d5`，这里 `d` 是 dot 的意思
- 样式不能包含斜杠：使用字母 `d` 替代，如：`top-1/2` => `top-1d2`，这里 `d` 是 divide 的意思
- 不支持 `*` 通配符选择器

## 剔除的样式
- hover 样式：小程序端不需要
- focus 样式：小程序端不需要
- dark 样式：样式体积翻倍，不适合小程序场景（2M 包大小限制）
- responsive 样式：小程序端不需要响应式相关样式，如：sm、lg、xl 前缀
- currentColor：tailwindcss 的 currentColor 不支持使用

## 小程序特定样式
base-styles.scss 已经增加样式重置，但还需额外增加部分样式，修复小程序场景下常见样式问题
- divide：小程序不支持 `> :not([hidden]) ~ :not([hidden])` 选择器，因此实现上只能用【直接子代组合器 + 一般兄弟组合器】即 `> view ~ view`，使用需要注意列表项目需要用 view 封装
- border-none: 修复小程序按钮 border-none 无效的问题
- placeholder: 小程序没有 `::placeholder` 伪类，直接使用类似 `placeholder-gray-200` 可能无效，请使用 `text-gray-200` 替代
- box-shadow：代码实现直接使用 px 单位，设计稿尺寸不同时需要考虑是否改动

## 代码风格说明
- 可读性：为了增强代码可读性，可以写成单行的样式就写成单行，并通过大括号对齐
- 继承性：为了保持和 tailwindcss 良好的继承关系，代码注释使用 tailwindcss 标题顺序，方便开发者定位和增删。
- 行字符限制：一般不超过 72 字符

例如：
```scss
.static   { position: static; }
.fixed    { position: fixed; }
.absolute { position: absolute; }
.relative { position: relative; }
.sticky   { position: sticky; }
```

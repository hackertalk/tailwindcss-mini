## 工具优先
[tailwindcss](https://tailwindcss.com) 的理念就是 Utility-First（工具优先），提供了很好的配置，但是由于小程序端限制过多，比如样式不能包含小数点、斜杠、大量 css 选择器不能用等原因， tailwindcss 很难通过配置适配小程序端样式需求，由于 tailwindcss 常用类代码不超过 2000 行，所以我们通过手动提取样式 + scss 工具累类实现 tailwindcss 类似的功能。

## 为什么使用这个方案
对比 [taro-tailwind](https://github.com/windedge/taro-tailwind) 等其他配置方案，此方案具有以下优点：
- 框架无关：无论你使用 uni-app、taro 还是 wepy 都可以使用
- 支持自定义：可以通过修改 scss 变量（颜色、尺寸等）配置自定义样式
- 支持全量样式： 支持 divide 等其他方案不支持的样式
- 按需使用可裁剪：可以根据自己使用情况，开启不同变量，自动生成体积更小的 css 代码
- 支持不同设计稿：无论你是 375px 还是 750px 设计稿还是使用 rpx、 rem 单位，都可以通过修改一个 scss 变量实现配置
- 无依赖无需安装：代码甚至不依赖 tailwindcss 的逻辑，没有一行 js 代码，避免 js 和 css 交互问题，直接拷贝 scss 文件即可使用（不到2000行）

## 如何使用
直接 copy 项目 scss 文件即可，不到 2000 行代码，无任何 js 依赖，也可直接使用生成的 css 文件：index.css

修改构建命令：
```shell script
npm run build
```
### 自定义（customization.scss 文件）
- `$color-palette`：调色板，可以自定义全局颜色
- `$spacing`：基本尺寸，可以适配不同设计稿
- `$sizing`：百分比尺寸，出现在 width、height 样式中
- `$placement`：百分比尺寸，出现在 top、left、right、bottom 等定位样式中
- `$ring-width`：边框样式
- `$divide-width`：分割线样式

### 设计稿尺寸适配
默认是 375px 设计稿，如需适配其他设计稿，可通过 `$base-unit` 变量开启：
```scss
// $base-unit: 1rem; // 使用 rem 样式时开启
// $base-unit: 32px; // 750px 设计稿开启
$base-unit: 16px; // 375px 设计稿开启
```
如果你配合 [taro](http://taro-docs.jd.com/taro/docs/README) 框架使用，请阅读相关文档：[设计稿及尺寸单位](http://taro-docs.jd.com/taro/docs/size/)

### 按需使用
由于小程序端开发环境混乱，且一般项目启动后电脑就很卡，很难像 tailwindcss 一样开启全局 watch，动态生成最小化的 css 体积，这里我们可以通过手动配置 scss 变量实现按需使用，最小化 css 体积。

scss 中使用了 `@each` 批量生成 css 代码，比如小程序端并不需要 `w-96`、`w-80` 这样的大尺寸，直接注释 `$spacing` 变量相应的值即可减少代码生成，同样，很多颜色不需要也可以直接注释，减少 css 体积。原理如下：
```scss
@import "styles/customization";

@each $name, $size in $spacing {
  .w-#{$name} { width: $size; }
}
```

## 剔除的样式
- hover 样式：小程序端不需要
- focus 样式：小程序端不需要
- dark 样式：样式体积翻倍，不适合小程序场景（2M 包大小限制）
- responsive 样式：小程序端不需要响应式相关样式，如：sm、lg、xl 前缀
- currentColor：tailwindcss 的 currentColor 功能不支持使用

## 小程序限制点
- 样式不能包含小数点：使用字母 `d` 替代，如：`pt-1.5` => `pt-1d5`，这里 `d` 是 dot 的意思
- 样式不能包含斜杠：使用字母 `d` 替代，如：`top-1/2` => `top-1d2`，这里 `d` 是 divide 的意思
- 不支持 `*` 通配符选择器：通过列举标签覆盖解决，参考 base-styles.scss 文件

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

## 黑客说小程序
[黑客说](https://hackertalk.net/)：一个有趣的程序员交流平台

![](hackertalk-mini.jpg)

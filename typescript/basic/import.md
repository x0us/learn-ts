# ☕ TypeScript 中的模块导入

---

TypeScript 中的模块导入方法比较多。我下面列举一下我在实际中用到的一些方法：

#### type alias with import()
用于导入目标模块中的类型

```js
type Dirent = import('fs').Dirent
type StatsBase<T> = import('fs').StatsBase<T>
```

这种做法好处是不会生成 js 代码，还有个优点是只影响类型声明空间，不污染值声明空间。缺点是泛型参数列表都得自己重新写一遍。

#### type alias with typeof import()
用于导入目标模块中的值的类型，比如 querystring 的 escape 的类型是 (str: string) => string, 我们想提取其为 Escape，则可以用：
```js
type Escape = typeof import('querystring').escape
```

#### import type
```js
import type { StatsBase } from 'fs';
type Stats = StatsBase<number>
```
这种写法的解决了 type alias with import() 写法必须手写泛型参数的问题。

#### import type by import
```js
import { StatsBase } from 'fs';
type Stats = StatsBase<number>
```
这种导入类型的方法最常见。不过在某些情况下，可能没有擦除干净，比如配置了 --importsNotUsedAsValues preserve。所以为了兼容性，以后尽量少用它。

#### ES Module import
这种 import 是最常见的，比如
```js
import defaultExport from "module-name";
import * as name from "module-name";
import { export1 } from "module-name";
import { export1 as alias1 } from "module-name";
import { export1 , export2 } from "module-name";
```
但是配合 commonjs 模块使用时候，需要注意两个编译选项 esModuleInterop 和 allowSyntheticDefaultImports。前者会影响运行时后者不会。

#### commonjs import
我们在 commonjs 中可能使用的是
```js
const fs = require('fs');
```
这种代码在 ts 中对应的是
```js
import fs = require('fs');
```
值得注意的是，这种代码只能在 "module": "CommonJS" 才能使用

#### require with other function
这种不算是 ts 语法，但是是一种工程里的实践，比如我们用 create-react-app 开发 electron ，默认情况下，require 被 webpack 劫持了，我们不想改 webpack 配置情况下，如果我们想直接 require 某些库，不想被 webpack 打进 bundle，可以用：
```js
const fs = window.require('fs') as typeof import('fs');
```
当然最好的方法还是改 webpack 配置。
如果遇到了commonjs 和 esm 交互，想要写 esModuleInterop 无论开启与否，都兼容的代码，可以考虑：
```js
const assert = require('assert') as typeof import('assert');
```
或者在 iframe 里，想用外部 electron renderer 的 require
```js
const fs = (window.parent as any).require('fs') as typeof import('fs');
```
当然，配合本文开头的几种纯导入类型的方法一起食用，味道更佳。


#### 三斜线指令 类型声明
引入外部定义的扩展类型声明，使用三斜线 比如:
```js
// 引入 nagivator.connection 的类型声明：
/// <reference types="network-information-types" />
navigator.connection?.type //  "bluetooth" | "cellular" | "ethernet" | "mixed" | "none" | "other" | "unknown" | "wifi" | "wimax" | undefined

// 引入 webpack 运行环境定义的类型，比如 require.context
/// <reference types="@types/webpack-env" />

// 更常见的是引入 node 的类型声明
/// <reference types="node" />
// 这个类型声明引入，使得 import * as fs from 'fs' 等原生模块都有了类型声明
```
大家是不是好奇，为什么我们没写 /// <reference types="node" />，平时也可以用。

原因是： tsconfig 有一项是 "types",其中大家经常是写了["node"]或者没写。如果没写，则默认把 typeRoots 选项里的所有内容加入 "types"。加入 types 的内容，相当于自动帮你写了三斜线指令导入类型声明


---
source: https://zh-hans.react.dev/learn/importing-and-exporting-components
tags:
  - JS
---
## 默认导出 vs 具名导出

这是 JavaScript 里两个主要用来导出值的方式：默认导出和具名导出。到目前为止，我们的示例中只用到了默认导出。但你可以在一个文件中，选择使用其中一种，或者两种都使用。 **一个文件里有且仅有一个 *默认* 导出，但是可以有任意多个 *具名* 导出。**

![Default and named exports](https://zh-hans.react.dev/images/docs/illustrations/i_import-export.svg)

组件的导出方式决定了其导入方式。当你用默认导入的方式，导入具名导出的组件时，就会报错。如下表格可以帮你更好地理解它们：

| 语法 | 导出语句 | 导入语句 |
| --- | --- | --- |
| 默认 | `export default function Button() {}` | `import Button from './Button.js';` |
| 具名 | `export function Button() {}` | `import { Button } from './Button.js';` |

当使用默认导入时，你可以在 `import` 语句后面进行任意命名。比如 `import Banana from './Button.js'` ，如此你能获得与默认导出一致的内容。相反，对于具名导入，导入和导出的名字必须一致。这也是称其为 **具名** 导入的原因！

**通常，文件中仅包含一个组件时，人们会选择默认导出，而当文件中包含多个组件或某个值需要导出时，则会选择具名导出。** 无论选择哪种方式，请记得给你的组件和相应的文件命名一个有意义的名字。我们不建议创建未命名的组件，比如 `export default () => {}` ，因为这样会使得调试变得异常困难。
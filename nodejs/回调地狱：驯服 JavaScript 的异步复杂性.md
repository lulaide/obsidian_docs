---
source: "https://callbackhell.com/"
tags:
  - "JS"
---
## 回调地狱

### 什么是“ 回调地狱 ”？

异步 JavaScript，或者说使用回调的 JavaScript，直观上很难正确理解。很多代码最终看起来像这样：

```javascript
fs.readdir(source, function (err, files) {
  if (err) {
    console.log('Error finding files: ' + err)
  } else {
    files.forEach(function (filename, fileIndex) {
      console.log(filename)
      gm(source + filename).size(function (err, values) {
        if (err) {
          console.log('Error identifying file size: ' + err)
        } else {
          console.log(filename + ' : ' + values)
          aspect = (values.width / values.height)
          widths.forEach(function (width, widthIndex) {
            height = Math.round(width / aspect)
            console.log('resizing ' + filename + 'to ' + height + 'x' + height)
            this.resize(width, height).write(dest + 'w' + width + '_' + filename, function(err) {
              if (err) console.log('Error writing file: ' + err)
            })
          }.bind(this))
        }
      })
    })
  }
})
```
看到金字塔形状和最后的所有 `})` 了吗？哎呀！这被亲切地称为 **回调地狱** 。 回调地狱的原因在于人们试图以一种从上到下的方式编写 JavaScript，使得执行过程在视觉上是线性的。很多人都会犯这个错误！在 C、Ruby 或 Python 等其他语言中，通常期望第 1 行的代码会在第 2 行代码开始运行之前完成，依此类推。然而，正如你将要学习的，JavaScript 是不同的。

## 什么是回调？

回调只是使用 JavaScript 函数的一种约定名称。在 JavaScript 语言中并没有一个特别的东西叫做“回调”，这只是一个约定。与大多数函数立即返回某个结果不同，使用回调的函数需要一些时间来产生结果。“异步”这个词，也就是“async”，只是意味着“需要一些时间”或“发生在未来，而不是现在”。通常，回调仅在进行 I/O 时使用，例如下载东西、读取文件、与数据库交互等。当你调用一个普通函数时，你可以使用它的返回值：

```javascript
var result = multiplyTwoNumbers(5, 10)
console.log(result)
// 50 gets printed out
```
然而，异步函数和使用回调的函数不会立即返回任何东西。
```javascript
var photo = downloadPhoto('http://coolcats.com/cat.gif')
// photo is 'undefined'!
```
在这种情况下，gif 可能需要很长时间才能下载完成，而你不希望你的程序在等待下载完成时暂停（即“阻塞”）。 相反，你将下载完成后应该运行的代码存储在一个函数中。这就是回调！你将它传递给 `downloadPhoto` 函数，当下载完成时，它会运行你的回调（例如“稍后给你回电”），并传入照片（如果出现问题则传入错误）。
```javascript
downloadPhoto('http://coolcats.com/cat.gif', handlePhoto)

function handlePhoto (error, photo) {
  if (error) console.error('Download error!', error)
  else console.log('Download finished', photo)
}

console.log('Download started')
```
人们在尝试理解回调时遇到的最大障碍是理解程序运行时事情执行的顺序。在这个例子中发生了三件主要的事情。首先声明了 `handlePhoto` 函数，然后调用 `downloadPhoto` 函数并将 `handlePhoto` 作为其回调传递，最后打印出 `’Download started’` 。请注意， `handlePhoto` 还没有被调用，它只是被创建并作为回调传递给 `downloadPhoto` 。但它不会运行，直到 `downloadPhoto` 完成其任务，这可能需要很长时间，具体取决于互联网连接的速度。这个例子旨在说明两个重要概念：– `handlePhoto` 回调只是存储一些稍后要做的事情的一种方式 – 事情发生的顺序并不是从上到下读取的，而是根据事情完成的时间跳跃

## 我该如何解决回调地狱？

回调地狱是由于糟糕的编码实践造成的。幸运的是，编写更好的代码并不难！你只需要遵循 **三条规则** ：

## 1\. 保持代码简洁

这里有一些混乱的浏览器 JavaScript，它使用 [browser-request](https://github.com/iriscouch/browser-request) 向服务器发起 AJAX 请求：

```javascript
var form = document.querySelector('form')
form.onsubmit = function (submitEvent) {
  var name = document.querySelector('input').value
  request({
    uri: "http://example.com/upload",
    body: name,
    method: "POST"
  }, function (err, response, body) {
    var statusMessage = document.querySelector('.status')
    if (err) return statusMessage.value = err
    statusMessage.value = body
  })
}
```
这段代码有两个匿名函数。让我们给它们起个名字！
```javascript
var form = document.querySelector('form')
form.onsubmit = function formSubmit (submitEvent) {
  var name = document.querySelector('input').value
  request({
    uri: "http://example.com/upload",
    body: name,
    method: "POST"
  }, function postResponse (err, response, body) {
    var statusMessage = document.querySelector('.status')
    if (err) return statusMessage.value = err
    statusMessage.value = body
  })
}
```
如你所见，命名函数非常简单，并且有一些直接的好处：– 使代码更易于阅读，因为函数名称具有描述性 – 当发生异常时，你将获得引用实际函数名称的堆栈跟踪，而不是“匿名” – 允许你将函数移动到程序的顶层并通过名称引用它们 现在我们可以将函数移动到程序的顶层：
```javascript
document.querySelector('form').onsubmit = formSubmit

function formSubmit (submitEvent) {
  var name = document.querySelector('input').value
  request({
    uri: "http://example.com/upload",
    body: name,
    method: "POST"
  }, postResponse)
}

function postResponse (err, response, body) {
  var statusMessage = document.querySelector('.status')
  if (err) return statusMessage.value = err
  statusMessage.value = body
}
```
请注意，这里的 `function` 声明是在文件的底部定义的。这要归功于 [函数提升](https://gist.github.com/maxogden/4bed247d9852de93c94c) 。

## 2\. 模块化

这是最重要的部分： **任何人都能够创建模块** （即库）。引用 [Isaac Schlueter](http://twitter.com/izs) （node.js 项目的创始人）的话： *“编写小模块，每个模块只做一件事，然后将它们组合成做更大事情的其他模块。如果你不去那里，就无法陷入回调地狱。”* 让我们去掉上面的样板代码，并通过将其拆分成几个文件来将其转变为一个模块。我将展示一个适用于浏览器代码或服务器代码（或两者都适用的代码）的模块模式： 这里是一个名为 `formuploader.js` 的新文件，包含我们之前的两个函数：

```javascript
module.exports.submit = formSubmit

function formSubmit (submitEvent) {
  var name = document.querySelector('input').value
  request({
    uri: "http://example.com/upload",
    body: name,
    method: "POST"
  }, postResponse)
}

function postResponse (err, response, body) {
  var statusMessage = document.querySelector('.status')
  if (err) return statusMessage.value = err
  statusMessage.value = body
}
```
`module.exports` 部分是 node.js 模块系统的一个示例，该系统在 node、Electron 和使用 [browserify](https://github.com/substack/node-browserify) 的浏览器中工作。我非常喜欢这种模块风格，因为它在任何地方都能工作，易于理解，并且不需要复杂的配置文件或脚本。现在我们有了 `formuploader.js` （并且它在被 browserify 处理后作为脚本标签加载到页面中），我们只需引入并使用它！以下是我们应用程序特定代码的样子：
```javascript
var formUploader = require('formuploader')
document.querySelector('form').onsubmit = formUploader.submit
```
现在我们的应用程序只有两行代码，并具有以下好处： – 对于新开发者来说更容易理解——他们不会因为需要阅读所有的 `formuploader` 函数而感到困惑—— `formuploader` 可以在其他地方使用而无需重复代码，并且可以轻松地在 github 或 npm 上共享

## 3\. 处理每一个错误

错误有不同类型：由程序员引起的语法错误（通常在你第一次运行程序时被捕获），由程序员引起的运行时错误（代码运行了但有一个错误导致某些事情出错），以及由无效文件权限、硬盘故障、无网络连接等引起的平台错误。本节仅旨在解决最后一种错误。前两条规则主要是关于使你的代码可读，但这一条是关于使你的代码稳定。当处理回调时，你本质上是在处理被调度的任务，这些任务在后台执行，然后成功完成或因失败而中止。任何有经验的开发者都会告诉你，你永远无法知道这些错误何时发生，因此你必须计划它们总是会发生。在回调中，处理错误的最流行方式是 Node.js 风格，其中回调的第一个参数始终保留用于错误。

```javascript
var fs = require('fs')

 fs.readFile('/Does/not/exist', handleFile)

 function handleFile (error, file) {
   if (error) return console.error('Uhoh, there was an error', error)
   // otherwise, continue on and use file in your code
 }
```
将第一个参数设为 `error` 是一个简单的约定，鼓励你记得处理错误。如果它是第二个参数，你可能会写出像 `function handleFile (file) { }` 这样的代码，更容易忽略错误。代码检查工具也可以配置为帮助你记得处理回调错误。最简单的一个叫做 [standard](http://standardjs.com/) 。你只需在代码文件夹中运行 `$ standard` ，它会显示你代码中每个未处理错误的回调。

### 摘要

1. 不要嵌套函数。给它们命名并将它们放在程序的顶层
2. 利用 [函数提升](https://gist.github.com/maxogden/4bed247d9852de93c94c) 的优势，将函数移动到“折叠”下方
3. 在每一个回调中处理 **每一个错误** 。使用像 [standard](http://standardjs.com/) 这样的代码检查工具来帮助你。
4. 创建可重用的函数并将其放入模块中，以减少理解代码所需的认知负担。将代码拆分成这样的小块也有助于处理错误、编写测试，迫使你为代码创建一个稳定且有文档的公共 API，并有助于重构。 避免回调地狱最重要的方面是 **将函数移开** ，以便程序的流程更容易理解，而不必让新手在所有函数的细节中挣扎，以了解程序试图做的事情。你可以先将函数移动到文件的底部，然后逐步将它们移动到另一个文件中，通过相对的 require 加载，例如 `require(‘./photo-helpers.js’)` ，最后将它们移动到一个独立的模块中，如 `require(‘image-resize’)` 。创建模块时的一些经验法则是：– 首先将重复使用的代码移入一个函数 – 当你的函数（或一组与同一主题相关的函数）变得足够大时，将它们移入另一个文件，并使用 `module.exports` 暴露它们。你可以通过相对 require 加载这个文件 – 如果你有一些可以在多个项目中使用的代码，给它一个自己的 readme、测试和 `package.json` ，并将其发布到 github 和 npm。 这种特定方法有太多令人惊叹的好处，无法一一列举！- 一个好的模块应该小巧，并专注于一个问题 - 模块中的单个文件不应超过大约 150 行 JavaScript - 一个模块不应有超过一层的嵌套文件夹，里面满是 JavaScript 文件。如果有，那可能做了太多事情 - 向你认识的更有经验的编码者请教，让他们给你展示好的模块示例，直到你对它们的样子有一个清晰的了解。如果理解所需时间超过几分钟，那可能不是一个很好的模块。

### 更多阅读
尝试阅读我的 [更长的回调介绍](https://github.com/maxogden/art-of-node#callbacks) ，或者试试一些 [nodeschool](http://nodeschool.io/) 的教程。还可以查看 [browserify-handbook](https://github.com/substack/browserify-handbook) ，了解编写模块化代码的示例。
### 那关于 promises/generators/ES6 等呢？
在查看更高级的解决方案之前，请记住回调是 JavaScript 的基本部分（因为它们只是函数），在学习更高级的语言特性之前，您应该学会如何阅读和编写回调，因为所有这些特性都依赖于对回调的理解。如果您还不能编写可维护的回调代码，请继续努力！ 如果您 *真的* 希望您的异步代码从上到下可读，可以尝试一些高级技巧。请注意 **这些可能会引入性能和/或跨平台运行时兼容性问题** ，因此请确保做好研究。 **Promise** 是一种编写异步代码的方法，它看起来仍然像是自上而下执行，并且由于鼓励使用 `try/catch` 风格的错误处理，能够处理更多类型的错误。 **生成器** 让你可以“暂停”单个函数，而不暂停整个程序的状态，这虽然使得代码稍微复杂一些，但让你的异步代码看起来像是自上而下执行的。查看 [watt](https://github.com/mappum/watt) 以获取这种方法的示例。 **异步函数** 是一个提议的 ES7 特性，它将进一步用更高级的语法包装生成器和承诺。如果这听起来有趣，可以去看看。就我个人而言，我在编写的 90%的异步代码中使用回调，当事情变得复杂时，我会引入像 [run-parallel](https://github.com/feross/run-parallel) 或 [run-series](https://github.com/feross/run-series) 这样的东西。我认为回调与承诺或其他任何东西对我来说并没有太大区别，最大的影响来自于保持代码简单，不嵌套，并拆分成小模块。无论你选择哪种方法，始终 **处理每一个错误** 并 **保持代码简单** 。
### 记住，只有你能防止回调地狱和森林火灾
你可以在 [github](http://github.com/maxogden/callback-hell) 上找到这个的源代码。
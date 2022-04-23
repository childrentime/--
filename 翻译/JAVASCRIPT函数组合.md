# JAVASCRIPT函数组合：有什么大不了的？

## 你可以学到什么？

- 使用函数组合的好处
- 使用正则表达式解析 `MARKDOWN`

原文链接: [https://jrsinclair.com/articles/2022/javascript-function-composition-whats-the-big-deal/](https://jrsinclair.com/articles/2022/javascript-function-composition-whats-the-big-deal/);

原文作者: [James Sinclair](https://jrsinclair.com/about.html);

一些人会说，函数组合是某种神圣的真理。一个你应该沉思时卑微地跪下并上香的神圣原则。但函数组合其实并不复杂。不过你是否意识到，你可能一直在使用它。那么为什么函数式编程程序员会对此烦恼呢？它有什么大不了的？

## 什么是函数组合

函数组合是当我们将两个函数合并成一个的过程。也就是，我们的新函数调用其中一个函数，并且将结果传递给另外一个函数。就这么简单！比如下面的代码：

```js
// 我们将函数命名为 c2，是 compose two functions together 的缩写
const c2 = (funcA, funcB) => x => funcA(funcB(x));
```

一个不好理解的地方是（如果有的话），我们从一个函数中返回了另外一个函数。这就是为什么这里有两个箭头函数的原因。

我们如何把它运用到现实世界的问题呢？很容易，想象一下我们正在处理某种评论系统。我们希望允许评论中嵌入图片和链接，但是不允许任何旧的 `HTML`。为了实现这个，我们将创造一个裁剪过的 `MARKDOWN` 版本。在我们的版本中，链接就像下面这样：

```markdown
[link text goes here](http://example.com/example-url)
```

图片像这样：

```markdown
![alt text goes here](/link/to/image/location.png)
```

现在，使用正则表达式 ，我们可以对每个表达式编写一个函数。我们取一个字符串并且使用适当的 `HTML` 对其中的内容进行替换：

```js
const imagify = str => str.replace(
    /!\[([^\]"<]*)\]\(([^)<"]*)\)/g,
    '<img src="$2" alt="$1" />'
);
const linkify = str => str.replace(
    /\[([^\]"<]*)\]\(([^)<"]*)\)/g,
    '<a href="$2" rel="noopener nowfollow">$1</a>'
);
```

为了创建一个可以既转换图片又可以转换链接的函数，我们可以使用之前定义的 `c2()`：

```js
const linkifyAndImagify = c2(linkify, imagify);
```

尽管如此，使用 `c2` 并不比我们直接组合函数更简洁

```js
const linkifyAndImagify = str => linkify(imagify(str));
```

我们的 `c2()` 函数只节省了8个字符。当我们添加更多函数的时候，它能节省的字符更少。比如，假设我们想要添加对下划线的支持。

```js
const emphasize = str => str.replace(
    /_([^_]*)_/g,
    '<em>$1</em>'
);
```

然后我们将它加入之前的函数中

```js
const processComment = c2(linkify, c2(imagify, emphasize));
```

与我们直接进行组合对比：

```js
const processComment = str => linkify(imagify(emphasize(str)));
```

使用 `c2` 函数，它依然更简短，但也差不多。要是我们能定义自己的操作符就好了，比如我们可以定义一个点操作符（•）来将一个函数的右侧与一个函数的左侧组合，那么我们的 `processComment` 函数会像这样：

```js
const processComment = linkify • imagify • emphasize;
```

不幸的是，`JavaScript`[现在](https://github.com/tc39/proposal-operator-overloading)还并不支持我们自定义操作符。相反，我们将编写一个多元的组合函数。

## 组合

我们想要组合许多函数更加简单，为了达到这个目的，我们将使用[剩余参数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)来将参数列表转换为数组。一旦我们有一个数组，我们就可以使用 `reduceRight`方法去轮流调用每个函数。使用这个的话，我们的代码将如下：

```js
const compose = (...fns) => x0 => fns.reduceRight(
    (x, f) => f(x),
    x0
);
```

为了说明 `compose` 函数如何工作，我们继续在我们的评论系统中添加一点新功能，现在我们允许用户在一行的开头放置三个哈希（###）来添加 `<h3>`元素。

```js
const headalize = str => str.replace(
    /^###\s+([^\n<"]*)/mg,
    '<h3>$1</h3>'
);
```

我们可以使用我们的函数来构建评论系统：

```js
const processComment = compose(linkify, imagify, emphasize, headalize);
```

如果空间不足，我们可以将每个函数单独放一行

```js
const processComment = compose(
    linkify,
    imagify,
    emphasize,
    headalize
)
```

尽管如此，这里还有点小问题。`headalize()`函数是第一个被执行的，却被放在列举的函数的最后。如果我们从上到下阅读，这些函数是逆序执行。这是因为 `compose` 函数模仿了我们直接组合函数的布局。

```js
const processComment = str => linkify(imagify(emphasize(headalize(str))));
```

这就是为什么 `compose` 函数使用 `reduceRight` 方法而不是 `reduce` 方法。次序很重要！如果我们在 `linkify` 函数之前执行 `imagify` 函数，我们的代码将不会工作。我们的图像好像都变成了链接。

![process img](https://jrsinclair.com/assets/processing-comments-with-compose.svg)

如果我们要在垂直列表中编写函数的话，为什么要颠倒顺序呢？我们可以编写一个函数来从另一个方向组合函数。这样，数据将从上到下流动。

## 流动

想要创建 `compose` 函数的相反版本，我们只需要将 `reduceRight` 函数更换为 `reduce` 函数就可以了。代码如下：

```js
// 我们将其命名为 flow，因为现在值从左到右流动
const flow = (...fns) => x0 => fns.reduce(
    (x, f) => f(x),
    x0
);
```

为了展示它如何工作，我们在评论系统中添加了另外一个功能。这次，我们添加了反引号之间的代码格式。

```js
const codify = str => str.replace(/`([^`<"]*)`/g, '<code>$1</code>');
```

将它丢进 `flow` 函数中，我们得到：

```js
const processComment = flow(
    headalize,
    emphasize,
    imagify,
    linkify,
    codify
);
```

![flow process image](https://jrsinclair.com/assets/processing-comments-with-flow.svg)

现在看起来要比我们直接组合好不少了。

```js
const processComment = str => codify(
    linkify(
        imagify(
            emphasize(
                headalize(str)
            )
        )
    )
);
```

确实，`flow` 函数更加整齐。因为它使用起来非常愉快，我们可能会发现自己经常使用它来构建功能。但是如果我们只使用一次函数，我们可能会变得懒惰并且立即调用它。比如：

```js
const processedComment = flow(
    headalize,
    emphasize,
    imagify,
    linkify,
    codify
)(commentStr);
```

这种结构有时候会很尴尬。一些 `JavaScript` 开发者觉得立即调用函数让人不安。即使我们的同事可以接受它，这对双括号依然很丑陋。

不要害怕，我们可以使用另外一种组合函数来解决它。

## 管道

我们将创建一个新的函数，`pipe()`，它和 `flow` 函数使用剩余参数的方式有一些不同：

```js
const pipe = (x0, ...fns) => fns.reduce(
    (x, f) => f(x),
    x0
);
```

我们的 `pipe` 函数和 `flow` 函数主要在两个地方不同：

1. 它返回一个值，而不是函数。`flow` 函数总是返回一个函数，而 `pipe` 函数总是返回一个值。
2. 它使用第一个参数作为值。使用 `flow` 的时候，所有的参数都必须是函数。

结果是我们的组合函数将会立即执行。这意味着我们不能重用组合函数，但通常我们也并不需要。

为了说明 `pipe` 函数如何有用，来对我们的例子做一点点小的改变。假设我们有一个评论数组需要处理，我们可能定义一些关于数组的实用函数。

```js
const c2 = (funcA, funcB) => x => funcA(funcB(x));
const map    = f => arr =>arr.map(f);
const filter = p => arr => arr.filter(p);
const take   = n => arr => arr.slice(0, n);
const join   = s => arr => arr.join(s);
```

然后一些关于字符串的实用函数

```js
const itemize        = str => `<li>${str}</li>`;
const orderedListify = str => `<ol>${str}</ol>`;
const chaoticListify = str => `<ul>${str}</ul>`;
const mentionsNazi   = str => (/\bnazi\b/i).test(str);
```

我们将它们与 `pipe` 函数组合

```js
const comments = pipe(commentStrs,
    filter(noNazi),
    take(10),
    map(emphasize),
    map(itemize),
    join('\n'),
);
```

如果我们眯起眼睛，我们会发现我们的管道与数组的调用链没什么区别

```js
const comments = commentStrs
    .filter(noNazi)
    .slice(0, 10)
    .map(emphasize)
    .map(itemize)
    .join('\n');
```

现在，一些人会感觉数组的调用链更加简洁。这可能是对的。所以一些人会好奇我们为什么要浪费时间在 `pipe` 函数和其他实用函数上面。所有的实用函数仅仅是调用数组方法而已。为什么不直接调用他们呢？因为 `pipe` 函数有个额外的优点。它可以使用其他的函数来保持流动，即使这个值上面本来没有这个函数可以调用。比如，我们可以添加一个 `chaoticListify` 函数到我们的管道中。

```js
const comments = pipe(commentStrs,
    filter(noNazi),
    take(10),
    map(emphasize),
    map(itemize),
    join('\n'),
    chaoticListify,
);
```

如果需要的话，我们可以添加更多的函数，并且可以通过这种方式构建整个应用程序。

## 有什么大不了的？

我觉得 `compose`，`flow`，`pipe` 函数已经非常简洁，但是我可以理解一些人还是会有所怀疑。毕竟，我们可以使用变量赋值的形式来书写上面的管道代码。

```js
const withoutNazis       = commentStrs.filter(noNazi);
const topTen             = withoutNazis.slice(0, 10);
const itemizedComments   = topTen.map(itemize);
const emphasizedComments = itemizedComments.map(emphasize);
const joinedList         = emphasizedComments.join('\n');
const comments           = chaoticListify(joinedList);
```

这个代码很好。对于大部分人来说，它非常熟悉并且是可阅读的。而且他得到了和我们上面的版本一样的结果。为什么会有人想到使用 `pipe` 函数呢？

为了解答这个问题，我希望我们看看这两个代码块并且做这两件事：

1. 计算它们彼此1的分号数量
2. 观察我们在变量赋值版本中使用了哪些实用函数

看看变量赋值版本如何有六个分号？ `pipe()` 版本如何有一个？这里发生了一些微妙但重要的事情。在变量赋值版本中，我们创建了六个语句。在 `pipe()` 版本中，我们将整个事物组合成一个表达式。使用表达式进行编码是函数式编程的核心。

现在，您可能一点也不关心函数式编程。没关系。但是使用 `pipe()` 开辟了一种全新的方式来构建程序。使用语句，我们将代码编写为计算机的一系列指令。这很像书上的菜谱，做这个；然后这样做；然后做这件事。但是通过组合，我们将代码表示为函数之间的关系。

这似乎仍然没有那么令人印象深刻。谁在乎组合是否开辟了另一种编写代码的方式？几十年来，我们一直在写语句，它完成了工作。当然，该变量分配版本会创建更多的中间变量。但所做的只是改变解释器使用调用堆栈的哪一部分。本质上，两个版本都在做同样的事情。但是组合的重要性不在于它如何改变代码。不，它的意义在于它如何改变我们。具体来说，它如何改变我们的**思维方式**。

组合鼓励我们将代码视为表达式之间的关系。这反过来又鼓励我们专注于我们想要的结果。也就是说，而不是关注每个步骤的细节。更重要的是，组合还鼓励我们使用小的、可重用的函数进行编码。这加强了我们对结果的关注，而不是实施细节。结果，我们的代码变得更具声明性。

根据我们目前的示例代码，这种**思维方式**的转移可能并不明显。我们一直在比较的两个例子并没有太大的不同。但我们可以证明 `pipe()` 版本更具声明性。我们可以在不更改单个字符的情况下使 pipe() 版本更高效。比如，我们将更改它使用的实用函数：

```js
const map = f => function*(iterable) {
  for (let x of iterable) yield f(x);
};

const filter = p => function*(iterable) {
  for (let x of iterable) {
    if (p(x)) yield x;
  }
};

const take = n => function*(iterable) {
  let i = 0;
  for (let x of iterable) {
    if (i >= n) return;
    yield x;
    i++;
  }
};

const join = s => iterable => [...iterable].join(s);
```

我们一点也不需要改变我们的管道

```js
const comments = pipe(commentStrs,
    filter(noNazi),
    take(10),
    map(emphasize),
    map(itemize),
    join('\n'),
    chaoticListify,
);
```

实用功能如何工作的细节并不是很重要。总之，他们使用生成器而不是内置的数组方法。使用生成器意味着我们不再创建中间数组。但这里的重点不在于效率，生成器代码可能根本不会提高性能。没关系。关键是它有**有效**。它使用完全不同的机制来遍历数据，但它提供了相同的结果。

这里的重点是思维的转变。公平地说，我们可以编写一个使用变量赋值和生成器的代码版本。我们会得到同样的好处。但是将代码编写为一系列语句并不鼓励这种思维转变。我们将管道定义为功能之间的关系。为此，我们需要一堆可重用的实用程序函数。在**领域驱动**的设计术语中，这些功能创建了一个自然的**反腐败层**。这让我们可以在不改变高级意图的情况下更改实现细节。这就是为什么函数组合很重要的原因。

## 结语

从本质上讲，函数组合并不复杂，组合两个功能很简单，容易明白。我们已经研究了如何采用这个想法并将其扩展为一次组合许多功能。在主题的变化中我们已经探索了 `compose()`、`flow()` 和 `pipe()` 函数。我们可以使用这些函数来创建简洁、优雅的代码。但组合的真正优雅之处不在于代码，而在于它如何改变我们。它如何为我们提供了思考代码的新方法。

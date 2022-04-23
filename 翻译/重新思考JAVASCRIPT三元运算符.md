# 重新思考 JAVASCRIPT 三元运算符

原文链接: [http://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-intro/](https://jrsinclair.com/articles/2021/rethinking-the-javascript-ternary-operator/);
原文作者: [James Sinclair](https://jrsinclair.com/about.html);

我们都想编写既清晰又简洁的代码。但有时我们不得不在两者之间做出选择。我们可以清晰或简洁，但不能同时两者兼而有之。而且很难选择一条道路。双方都有很好的论据。更少的代码行意味着更少的错误隐藏位置。但是清晰、可读的代码更容易维护和修改。不过，总的来说，经验告诉我们，清晰胜过简洁。如果您必须在可读性和简洁性之间做出决定，请选择可读性。

## 三元运算符的麻烦

为什么人们会如此怀疑三元运算符呢？有那么糟糕吗？这不像普通的程序员每天早上醒来会想，“我今天会讨厌三元组。”怀疑一定来自某个地方。人们有充分的理由不喜欢三元运算符。让我们仔细看看其中的几个。

### 怪异

人们不喜欢三元运算符的一个原因是它们实在是太奇怪了。作为运算符，就是这样。 `JavaScript` 有很多二元运算符——作用于两个表达式的运算符。您可能熟悉 +、-、* 和 / 等算术运算符。以及像 &&、|| 这样的布尔运算符和===。总共至少有 28 个二元运算符。 （也就是说，取决于我们讨论的 `ECMAScript` 版本）。它们熟悉且直观。左边是表达式，运算符符号，右边是表达式。非常简单。

一元运算符较少。但它们也不是那么奇怪。您可能熟悉否定运算符，!。而且您可能也以一元形式使用过 + 和 -。例如，-1。大多数情况下，它们对符号右侧的表达式进行操作。而且它们不会造成太大的麻烦。

但是只有一个三元运算符。而且，顾名思义，它作用于三个表达式。因此我们用两个符号来写它：？和 ：。否则，我们无法判断中间表达式从哪里开始和结束。所以它看起来像这样

```js
(/* First expression*/) ? (/* Second expression */) : (/* Third expression */)
```

在实践中，我们这样使用它：

```js
const protocol = (request.secure) ? 'https' : 'http';
```

如果第一个表达式是“真”，则三元解析为第二个表达式的值。否则，它解析为第三个表达式的值。但是为了保持这三个表达式的区别，我们需要两个符号。没有其他运算符是由这样的多个符号组成的。

不过，这并不是唯一奇怪的事情。大多数二元运算符具有一致的类型。算术运算符处理数字。布尔运算符处理布尔值。位运算符同样适用于数字。对于所有这些，双方的类型都是相同的[^1]。但是三元运算符有奇怪的类型。使用三元运算符，第二个和第三个表达式可以是任何类型。但是解释器总是将第一个转换为布尔值。这是独一无二的。所以，就运算符而言，这样的行为很奇怪。

### 初学者不友好

所以三元运算符很奇怪。因此，人们批评它使初学者困惑也就不足为奇了。这里有很多值得记住的地方。如果你看到一个问号符号，你必须去寻找一个冒号。与 if 语句不同的是，很难将三元组解读为伪代码。例如，假设我们有一个这样的 if 语句：

```js
if (someCondition) {
    takeAction();
} else {
    someOtherAction();
}
```

把它翻译成伪代码并不需要太多的努力。如果 someCondition 计算结果为 true，则调用不带参数的函数 takeAction。否则，调用不带参数的函数 someOtherAction。这不是一个很大的飞跃。而三元运算符是由神秘的符号组成的，读起来不像英文。这需要更多的努力，学习编码已经够难的了！

### 难以阅读

即使您不是初学者，三元运算符也可能难以阅读。那些神秘的符号可能会绊倒我们中最优秀的人。特别是如果三元运算符组成了长表达式。考虑使用 `Ratio` 库[^2]的这个例子：

```js
const ten = Ratio.fromPair(10, 1);
const maxYVal = Ratio.fromNumber(Math.max(...yValues));
const minYVal = Ratio.fromNumber(Math.min(...yValues));
const yAxisRange = (!maxYVal.minus(minYVal).isZero()) ? ten.pow(maxYVal.minus(minYVal).floorLog10()) : ten.pow(maxYVal.plus(maxYVal.isZero() ? Ratio.one : maxYVal).floorLog10());
```

很难理解上面的代码是什么意思。三元运算符中的每个表达式至少有两个链式方法调用。更不用说嵌套在最终表达式中的另一个三元运算符。这种三元表达式很难阅读。我不建议您编写这样的代码。

当然，我们可以通过添加换行符使它稍微好一点。 `Prettier`（格式化库）会这样做：

```js
const ten = Ratio.fromPair(10, 1);
const maxYVal = Ratio.fromNumber(Math.max(...yValues));
const minYVal = Ratio.fromNumber(Math.min(...yValues));
const yAxisRange = !maxYVal.minus(minYVal).isZero()
    ? ten.pow(maxYVal.minus(minYVal).floorLog10())
    : ten.pow(maxYVal.plus(maxYVal.isZero() ? Ratio.one : maxYVal).floorLog10());
```

这稍微好一点。但不是很大的进步。我们可以通过添加垂直对齐来进行另一个小的改进。

```js
const ten        = Ratio.fromPair(10, 1);
const maxYVal    = Ratio.fromNumber(Math.max(...yValues));
const minYVal    = Ratio.fromNumber(Math.min(...yValues));
const yAxisRange = !maxYVal.minus(minYVal).isZero()
                 ? ten.pow(maxYVal.minus(minYVal).floorLog10())
                 : ten.pow(maxYVal.plus(maxYVal.isZero() ? Ratio.one : maxYVal).floorLog10());
```

但是仍然很难阅读。一般来说，在三元运算符中投入太多太容易了。而且你投入的越多，它们就越难以阅读。

嵌套三元运算符尤其成问题。阅读时很容易漏掉一个冒号。在上面的示例中，换行符有点帮助。但是我们很容易像下面这样写。

```js
const ten        = Ratio.fromPair(10, 1);
const maxYVal    = Ratio.fromNumber(Math.max(...yValues));
const minYVal    = Ratio.fromNumber(Math.min(...yValues));
const yAxisRange = !maxYVal.minus(minYVal).isZero()
                 ? ten.pow(maxYVal.minus(minYVal).floorLog10()) : ten.pow(maxYVal.plus(maxYVal.isZero() ? Ratio.one
                 : maxYVal).floorLog10());
```

当然，这是一个虚构的例子。所以这是一种**稻草人论点**（逻辑谬误)。我故意写了糟糕的代码来说明这个问题。但重点仍然存在。编写不可读的三元表达式太容易了。尤其是嵌套三元运算符。可读性很重要。正如马丁·福勒所说：

> 任何傻瓜都可以编写计算机可以理解的代码。优秀的程序员编写人类可以理解的代码。[^3]

我们要编写可阅读的代码。这是人们对三元表达式的主要问题。太容易在三元表达式中塞太多东西了。一旦你开始嵌套它们，你制造混乱的机会就会成倍增加。所以我能理解为什么你会鼓励初级程序员避免使用三元表达式。坚持使用漂亮、安全的 `if` 语句要好得多。

但是 `if` 语句有多安全？

## if 语句的不可信性

三元表达式有其缺点。如果仅此而已，我会毫无疑问地避开三元表达式。我希望我的代码易于其他人阅读——包括初学者。但三元反对者倾向于做出两个假设：

1. 使用三元组的唯一原因是简洁或聪明；和

2. `if` 语句也可以代替三元组。

我考虑得越多，就越相信这两个假设都不正确。使用三元表达式有充分的理由。与编写更短代码无关的原因。这是因为 if 语句和三元运算符不同。没有微妙的不同——显著不同。以一种直接影响 JavaScript 构建块的方式有所不同。

为了说明，让我们看一个例子。这里有两段代码。

首先，一个 if 语句：

```js
let result;
if (someCondition) {
    result = calculationA();
} else {
    result = calculationB();
}
```

接下来，使用三元:

```js
const result = (someCondition) ? calculationA() : calculationB();
```

人们倾向于假设这两个例子是等价的。从某种意义上说，他们是对的。在两段代码的末尾，一个名为 result 的 变量将被设置为一个值，要么是 `calculationA()`，要么是 `calculationB()`。但从另一个意义上说，这两个例子是完全不同的。 `if` 语句示例中的 `let` 为我们提供了第一个线索。

有什么不同？简而言之，`if` 语句是一个语句，而三元是一个表达式。

但是，这是什么意思？这是一个总结:

- 表达式总是计算出某个值
- 语句是“独立的执行单元“[^4]

这是一个重要的概念。表达式计算为一个值。语句没有。您不能将语句的结果分配给变量。您不能将语句的结果作为函数参数传递。`if` 语句是语句，而不是表达式。 `if` 语句无法解析为值。因此，只有引起副作用,它才可以做任何有用的事情。

什么是副作用？副作用是我们的代码除了解析为值之外所做的任何事情。这包括很多东西：

- 网络请求；

- 读写文件；

- 数据库查询；

- 修改 `DOM` 元素；

- 修改全局变量；

- 甚至写入控制台。

它们都是副作用。

现在，有人可能会想“那又怎样？谁在乎我们是否会造成副作用？”毕竟，副作用是我们编写代码的全部原因，对吧？只要我们完成工作，那有什么关系？

从某种意义上说，这并不重要。工作代码才是最重要的。在这一点上，我们同意。但是你怎么**知道**它在起作用？你怎么知道你的程序只做你认为它做的事情。你怎么知道它不是也在挖掘狗狗币或删除数据库表？

在某种程度上，这是函数式编程的核心思想。通过谨慎对待副作用，我们获得了对代码的信心。只要有可能，我们更喜欢使用纯函数。如果一个函数是纯函数，我们知道它**只会**进行计算并返回一个值。就是这样。

这对 if 语句和三元组意味着什么？这意味着我们应该以一定的**怀疑**态度对待 if 语句。让我们看一下之前的示例。

```js
if (someCondition) {
    takeAction();
} else {
    someOtherAction();
}
```

哪个分支 `someCondition` 导致我们失败并不重要。 if 语句唯一能做的就是产生副作用。它调用 `takeAction()` 或 `someOtherAction()`。但这些都没有返回值。 （或者，如果他们这样做了，我们不会将其分配给任何东西。）这些函数可以做任何有用的事情的唯一方法是到达块之外。它可能是没有问题的，比如在外部范围内改变一个值。但这仍然是一个副作用。

我是否建议我们永远不要使用 `if` 语句？不，但一定要认清它们的本质。每次看到一个，你必须问自己“这里发生了什么**副作用**？”如果您无法回答问题，则说明您不了解代码。

## 重新考虑三元表达式

似乎我们有充分的理由怀疑 if 语句。那么三元呢？他们总是更好吗？不，但是是的……而且没有。我们之前讨论的所有批评仍然有效。但是，三元组至少具有表达式的优势。这意味着他们不那么可疑——至少在副作用方面。但副作用并不是我们喜欢使用表达式编码的唯一原因。

我们喜欢表达式，因为表达式组合比语句更好。运算符和函数允许我们从简单的表达式中构建复杂的表达式。例如，我们可以使用连接运算符构建复杂的字符串：

```js
('<h1>' + page.title + '</h1>');
```

我们可以采用这个表达式并将其作为函数参数传递。或者我们可以将它与使用更多运算符的其他表达式结合起来。我们可以继续将表达式与表达式组合来执行复杂的计算。组合表达式是编写代码的绝佳方式。

除此之外，你可能想知道：“为什么表达式组合这么特别？语句不也是“可组合的”吗？我们可以愉快地在 `if` 语句中添加一个 `for` 循环。还有一个 `for` 循环中的 `case-switch` 语句，没问题。语句相互嵌套就好了。我们可以使用语句来构建其他更复杂的语句。表达式有什么大不了的？

表达式相对于语句的优势是我们称之为**引用透明性**的东西。这意味着我们可以获取一个表达式的值，并在我们可以使用该表达式本身的任何地方使用它。我们可以用数学上的确定性来做到这一点，结果将是相同的。确切地！总是！ 100%！每次!

现在，您可能会想，“这与组合有什么关系？”好吧，引用透明性解释了为什么组合语句不同于组合表达式。我能想到的最好的类比是**乐高®积木**与**印花布购物袋**。

语句的组合方式就像印花布杂货袋的构成方式。我可以把印花布袋放在印花布袋里就好了。这些袋子里可能还有其他东西。我甚至可以用印花布袋小心地将单个物品包裹起来。然后将这些包裹的物品整齐地堆叠在其他印花布袋中。甚至结果可能在美学上令人愉悦。但这些袋子彼此之间没有任何真正的关系。它们通过嵌套连接。但就是这样。连接袋子没有**组织原则**。

同样，有些语句可以嵌套。也就是说，带有块的那些可以（例如 `if`语句和 `for` 循环）。但他们之间没有任何联系。这些块只是你想放在那里的任何东西的容器。就目前而言，这很好。但它是一种不同于表达式的组合。

表达式更像乐高®积木。它们的组合方式受到限制。顶部的小块与砖底部的间隙相连。但是一旦连接起来，这些砖就会形成一个新的形状。并且该形状可以与具有相同配置的任何其他形状互换。考虑下面的图片。我们有两个连接的形状。尽管形状由不同的块组成，但最终的形状是相同的。换句话说，它们是可以互换的。类似地，表达式可以与其计算值互换。我们如何计算价值并不重要。重要的是结果。

![To shapes constructed of LEGO bricks. The one on the left is constructed from three pieces: Two green 2 by 2 bricks stacked on top of one blue 2 by 4 brick. The one on the right is constructed from two blue 2-by-4 bricks stacked together.](https://jrsinclair.com/assets/lego-bricks.png)

现在，这个类比并不完美。它失败了，因为印花布袋的用途与乐高®积木不同。但这只是一个类比。这个想法仍然存在。组合表达式具有明显的优势，我们在组合语句时得不到的好处。而且由于三元运算符是一个表达式，它比 `if` 语句具有优势。

这是否意味着我们应该总是更喜欢三元表达式？他们绝对更好吗？不幸的答案是，不。在 `JavaScript` 中，与大多数语言一样，您可以随意在任何地方产生副作用。这包括内部表达式。这种自由的代价是永远保持警惕。你永远不知道意外的副作用可能会出现在哪里。例如：

```js
const result = (someCondition) ? dropDBTables() : mineDogecoin();
```

但是，我们不能立即放弃三元表达式。因为 if 语句和它不是一回事，而且更加冗长。当您看到三元表达式时，请考虑作者可能已经做出了深思熟虑的选择。除了简洁之外，他们可能有充分的理由使用三元表达式。

## 负责任地使用条件

那么我们该怎么办呢？三元表达式不是那么好。 if 语句也不是那么棒。我们应该怎么做？使用其他语言？

也许。但通常这不是一个可选项。所以我能给出的最准确、普遍适用的建议是：谨慎行事。考虑你同事的编码风格和偏好。考虑您要解决的问题的具体细节。权衡选项并且与他们深入交流。

除了之外，上面的建议可能不是那么有帮助。你可以这样说任何编码问题，它对我们使用条件没有帮助。为了更加有所帮助，我将给出一些具体的建议。**注意**，这只是我的看法。其他人有不同的看法。没关系。这些不是诫命或法律。只是我对如何编写更安全的条件偏好。

### 一些语句比其他语句更好

在讨论细节之前，让我们先考虑一下 `JavaScript` 代码的结构。你会注意到没有语句就不可能写出像样的代码。 `JavaScript` 程序主要是语句。你无法逃脱他们。但是有些语句比其他语句更安全。

最危险的语句是带有块的语句。 （这些是带有花括号 {…} 的位）。这包括 `if` 语句、`for` 循环、`while` 循环和 `switch-case` 语句。它们很危险，因为对它们有用的唯一方法就是引起某种**副作用**。有些东西必须超出块范围并改变上下文环境。

更安全的语句是变量赋值和返回语句。变量赋值很方便，因为它们将表达式的结果绑定到标签。我们称之为变量。而那个变量本身就是一个表达式。我们可以在其他表达式中尽可能频繁地再次使用它。所以，只要我们小心避免**修改**，变量赋值就很好了。

`Return` 语句很有用，因为它们使函数调用解析为一个值。函数调用是表达式。因此，就像变量赋值一样，`return` 语句帮助我们构建表达式。所以大多数时候他们也很好。

有了这些知识，我们就可以思考如何编写更安全的条件语句。

### 更安全的 if 语句

为了编写更安全的 if 语句，我遵循一个简单的规则：第一个（`“then”`）分支必须以 `return` 结束。这样，即使 if 语句不解析为值，外部函数也会。例如：

```js
if (someCondition) {
    return resultOfMyCalculation();
}
```

因此，如果您遵循此规则，您将永远不需要 **`else 块`**(译者注： 想想为什么不需要)。永远不会。相反，如果你确实引入了一个 `else 块`，你就知道你引入了一个副作用。它可能很小且无害，但它仍然存在。

### 更具可读性的三元表达式

我对三元表达式的一般建议是保持它们很小。如果表达式太长，请使用垂直对齐来阐明意图。或者更好的是，添加一些变量赋值。例如，我们可以改进之前的示例：

```js
const ten     = Ratio.fromPair(10, 1);
const maxYVal = Ratio.fromNumber(Math.max(...yValues));
const minYVal = Ratio.fromNumber(Math.min(...yValues));

// 创建四个额外的变量来标记三元表达式中的每一位，现在更清楚每个计算的用途。
const rangeEmpty = maxYVal.minus(minYVal).isZero();
const roundRange = ten.pow(maxYVal.minus(minYVal).floorLog10());
const zeroRange  = maxYVal.isZero() ? Ratio.one : maxYVal;
const defaultRng = ten.pow(maxYVal.plus(zeroRange).floorLog10());

// 从变量中拼凑的三元表达式
const yAxisRange = !rangeEmpty ? roundRange : defaultRng;
```

现在，有人可能会指出我们现在正在做不必要的计算。如果 `rangeEmpty` 为 `false`，我们不需要计算 `zeroRange` 或 `defaultRng`。为了避免这种情况，我们可以使用函数。

```js
const ten     = Ratio.fromPair(10, 1);
const maxYVal = Ratio.fromNumber(Math.max(...yValues));
const minYVal = Ratio.fromNumber(Math.min(...yValues));

// 创建两个函数，只有需要的时候才会计算range
const rangeEmpty = maxYVal.minus(minYVal).isZero();
const roundRange = () => ten.pow(maxYVal.minus(minYVal).floorLog10());
const defaultRng = () => {
    const zeroRange  = maxYVal.isZero() ? Ratio.one : maxYVal;
    return ten.pow(maxYVal.plus(zeroRange).floorLog10());
};

// 使用我们的两个新函数拼凑最终的三元表达式。
const yAxisRange = !rangeEmpty ? roundRange() : defaultRng();
```

现在，整个代码比以前长了很多。但这不一定是坏事。我们更喜欢清晰而不是简洁，对吧？在这个版本中，代码的意图更加清晰。

但是嵌套三元表达式呢？这不是一直很糟糕吗？嗯，没有。如果您注意垂直对齐，即使是深度嵌套的三元表达式也可以阅读。事实上，我经常更喜欢它们而不是 case-switch 语句。特别是当我有类似查找表的东西时。在这些情况下，三元表达式允许我格式化表格之类的东西。例如：

```js
const xRangeInSecs = (Math.max(...xValues) - Math.min(...xValues));
// prettier-ignore
const xAxisScaleFactor =
    (xRangeInSecs <= 60)       ? 'seconds' :
    (xRangeInSecs <= 3600)     ? 'minutes' :
    (xRangeInSecs <= 86400)    ? 'hours'   :
    (xRangeInSecs <= 2592000)  ? 'days'    :
    (xRangeInSecs <= 31536000) ? 'months'  :
    /* otherwise */              'years';
```

如果您使用 `Prettier` 之类的格式化程序，则需要禁用它。您可以像我在上面所做的那样使用内联注释。

这确实需要一些工作，但可以负责任地使用三元和 if 语句。是的，它是有代价的。这不仅需要付出努力，而且我们可能还必须挑战 `linter` 和编码标准。人们会过多地嵌套三元表达式。要么是因为懒惰，要么是因为他们根本不知道更好。但我认为这比盲目假设 `if` 语句是“**安全的**”要好。

## 展望

即使我们可以编写负责任的条件，我们的选择也是有限的。但是有一些改变的希望，可以看看 TC39 “[do expressions](https://github.com/tc39/proposal-do-expressions)”提案。这将允许我们将许多语句转换为表达式。例如，我们可以编写如下代码：

```js
let x = do {
  if (foo()) { f() }
  else if (bar()) { g() }
  else { h() }
};
```

`do` 块可以包含任意数量的语句，并解析为“**完成值**”，也就是在完成 do 块之前解析的最后一个值。

一些人指出这对 `JSX` 来说很方便。在 `JSX` 组件中，通常仅限于使用表达式。使用 `do` 表达式，您可以加入一些语句，这可能会使代码更具可读性。

该提案已提交给 2020 年 6 月的 `TC39` 会议，现在处于 `Stage 1`。所以它可能需要一段时间才能被浏览器和 `Node.js`支持。同时，如果您喜欢它的话，可以使用 `Babel` 转换。

## 结论

一般来说，我们大多数人都同意编写清晰的代码比简洁更重要。因此，人们给三元表达式保持怀疑的态度是可以理解的。但也许考虑到出于简洁和**难以学习**并不是使用三元组的唯一原因。我也鼓励你仔细看看你的 `if` 语句。仅仅因为某些东西是熟悉的，并不意味着它是**安全**的。

附录（2021-03-16）：如果您有兴趣调整 `ESLint` 以指定您的三元首选项，`Kyle Simpson` 已经创建了一个漂亮的 `ESlint` 插件。就我而言，我不会将其设置为默认值。但它比内置的 `ESLint` 规则提供了更多的控制。

[^1]: 公平地说，在 `JavaScript` 中，布尔运算符不会强制转换第二个表达式。例如，这允许我们滥用 ||运算符来设置默认值。例如： `const sortOrder = sortParam || 'asc';`
[^2]: 这是一个故意不好的例子，但它是基于我正在从事的一个真实项目的代码。
[^3]: 马丁·福勒，2008 年，马丁·福勒，<https://en.wikiquote.org/wiki/Martin_Fowler>
[^4]: Scott Wlaschin，2012， [Expressions vs. statements](https://fsharpforfunandprofit.com/posts/expressions-vs-statements/).

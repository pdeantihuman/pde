# Pratt Parser

偶尔，我会偶然发现一些算法或想法，它们是如此聪明，如此完美地解决了一个问题，以至于我觉得自己变得更聪明，或者仅仅通过学习就获得了一个新的超级大国。堆(几乎是我从被截断的计算机科学教育中得到的唯一东西)就是这样一件事。我最近偶然发现了另一个: Pratt 解析器。

当您编写解析器时，递归下降就像涂花生酱一样容易。当你可以根据你正在解析的下一段代码来决定下一步该做什么时，它就很出色了。这在语言的顶层通常是正确的，比如类和语句，因为大多数语句都是从唯一标识它们的东西开始的( if 、for、while等)。)。

但是当你接触到表情的时候就变得很棘手了。当涉及到像++这样的中缀运算符、像++这样的后缀运算符，甚至像？:，很难判断你正在解析哪种表达式，除非你已经完成了一半。你可以用递归下降来实现，但这是一件苦差事。您必须为每个优先级编写单独的函数(例如，JavaScript有17个)，手动处理关联性，并在一堆解析代码中涂抹语法，直到很难看到为止。

## 秘密武器 PB & J

 Pratt 解析解决了这个问题。如果递归下降是花生酱，那么 Pratt 解析就是果冻。当您将两者混合在一起时，您会得到一个简单、简洁、可读的解析器，它可以处理您抛出的任何语法。

 Pratt 处理运算符优先级和中缀表达式的技术是如此简单和有效，很谜的是为什么几乎没有人知道。70年代以后，自上而下的运算符优先分析器似乎已经从地球上消失了。Douglas  · Crockford 的JSLint使用一个来解析JavaScript，但是他的处理是关于它的为数不多的现代文章之一。

我认为问题的一部分在于 Pratt 的术语不透明， Crockford 的文章本身相当模糊。 Pratt 使用“零分母”等术语， Crockford 混合了跟踪词汇范围等额外的东西，模糊了核心思想。

这个我的切入点。我不会做任何革命性的事情。我将尝试获取自上而下操作符优先分析器背后的核心概念，并尽可能清晰地展示它们。我将换出一些术语来(我希望)澄清事情。希望我不会冒犯任何人纯粹的感情。我将使用Java编程，这是编程语言中粗俗的拉丁语。我想如果你能用Java写，你可以用任何语言写。

## 我们最终会做个什么出来

我是一个边做边学的人，这意味着我也是一个边做边教的人。为了展示 Pratt 解析器是如何工作的，我们将为一种叫做 Bantam 的小玩具语言构建一个解析器。它只是有表达式，因为这是 Pratt 解析真正有用的地方，但这应该足以让人们相信它的有用性。

尽管很简单，但它有完整的运算符范围:前缀(+、-、~、！)，后缀(！)、中缀(+、-、*、/、^)，甚至是mixfix条件运算符(？:)。它有多个优先级别和左右关联运算符。它还有赋值、函数调用和括号用于分组。如果我们能解析这个，我们就能解析任何东西。

```
NAME "a"
PLUS "+"
NAME "b"
LEFT_PAREN "("
NAME "c"
RIGHT_PAREN ")"
```

同样，我们也不会解释或编译这段代码。我们只想把它解析成一个好的数据结构。出于我们的目的，这意味着我们的解析器应该咀嚼一堆令牌对象，并吐出实现表达式的某个类的实例。为了给你一个想法，这里有一个条件表达式类的简化版本:

```java
class ConditionalExpression implements Expression {
  public ConditionalExpression(
      Expression condition,
      Expression thenArm,
      Expression elseArm) {
    this.condition = condition;
    this.thenArm   = thenArm;
    this.elseArm   = elseArm;
  }

  public final Expression condition;
  public final Expression thenArm;
  public final Expression elseArm;
}
```

(你一定很喜欢 Java 的“请签署一式四份”的官僚作风。就像我说的，如果你能用Java做，你可以用任何语言做。)

我们将从一个简单的Parser类开始构建这个。它拥有 token 流，处理 lookahead，并提供编写自顶向下递归下降解析器所需的基本方法，该解析器只 lookahead 一个 token (即L1(1))。这足以让我们继续了。如果我们以后需要更多，很容易扩展它。

好吧，让我们为自己构建一个解析器！

尽管“完整的” Pratt 解析器非常小，但我发现它有点难以破译。有点像quicksort，这个实现是一堆看似简单但却深深交织在一起的代码。为了解决这个问题，我们将一步一步地把它建立起来。

最简单的解析表达式是前缀运算符和单token运算符。对于这些东西来说，当前的token告诉我们需要做的一切。Bantam有一个单token表达式、命名变量和四个前缀运算符:+、-、~、和！。最简单的解析代码是:

```java
Expression parseExpression() {
  if (match(TokenType.NAME))       // return NameExpression...
  else if (match(TokenType.PLUS))  // return prefix + operator...
  else if (match(TokenType.MINUS)) // return prefix - operator...
  else if (match(TokenType.TILDE)) // return prefix ~ operator...
  else if (match(TokenType.BANG))  // return prefix ! operator...
  else throw new ParseException();
}
```

但这有点单一。如您所见，我们在根据`TokenType`切换到分支到不同的解析行为。让我们通过从`TokenType`映射到解析代码块来直接编码。我们将这些块称为“解析器”，它们将实现这一点:

```java
interface PrefixParselet {
  Expression parse(Parser parser, Token token);
}
```

解析变量名的实现是这样的

```java
class NameParselet implements PrefixParselet {
  public Expression parse(Parser parser, Token token) {
    return new NameExpression(token.getText());
  }
}
```

我们可以为所有前缀运算符使用一个类，因为它们只在实际运算符token本身上有所不同:

```java
class PrefixOperatorParselet implements PrefixParselet {
  public Expression parse(Parser parser, Token token) {
    Expression operand = parser.parseExpression();
    return new PrefixExpression(token.getType(), operand);
  }
}
```

PrefixExpression 只有两个属性，一个是运算符，还有一个就是被操作数。

您会注意到它调用`parseExpression()`来解析出现在运算符后面的操作数(即解析`-a` 中的这个`a`)。这个递归处理像`-+~！a`.

回到Parser，链式if语句被一个更干净的映射替换:

```java
class Parser {
  public void register(TokenType token, PrefixParselet parselet) {
    mPrefixParselets.put(token, parselet);
  }

  public Expression parseExpression() {
    Token token = consume();
    PrefixParselet prefix = mPrefixParselets.get(token.getType());

    if (prefix == null) throw new ParseException(
        "Could not parse \"" + token.getText() + "\".");

    return prefix.parse(this, token);
  }

  // Other stuff...

  private final Map<TokenType, PrefixParselet> mPrefixParselets =
      new HashMap<TokenType, PrefixParselet>();
}
```

为了定义到目前为止我们拥有的语法(变量和四个前缀运算符)，我们将创建这个helper方法:

```java
void prefix(TokenType token) {
  register(token, new PrefixOperatorParselet());
}
```

现在我们就可以这样定义语法

```java
register(TokenType.NAME, new NameParselet());
prefix(TokenType.PLUS);
prefix(TokenType.MINUS);
prefix(TokenType.TILDE);
prefix(TokenType.BANG);
```

这已经是对递归下降解析器的改进，因为我们的语法现在更具声明性，而不是分散在几个命令函数上，我们可以在一个地方看到实际的语法。更好的是，我们可以通过注册新的解析器来扩展语法。我们不必改变Parser类本身。

如果我们只有前缀表达式，我们现在就完成了。唉，我们没有。

## 中缀表达式

到目前为止，只有当第一个token告诉我们要解析什么样的表达式时，我们所拥有的才会起作用，但情况并非总是如此。对于像a + b这样的表达式，直到我们解析a并得到+之后，我们才知道我们有一个add表达式。我们必须扩展解析器来支持这一点。

幸运的是，我们处在这样做的好地方。我们当前的parseExpression()方法将解析一个完整的前缀表达式，包括任何嵌套的前缀表达式，然后停止。所以，如果我们把这个扔给它:

```java
-a + b
```

它将解析`-a`，让我们坐在`+`。这正是我们需要告诉我们正在解析的中缀表达式的token。这里中缀表达式和前缀表达式的唯一区别是，在中缀运算符之前有另一个表达式，它需要作为参数。让我们定义一个支持以下内容的解析器:

```java
interface InfixParselet {
  Expression parse(Parser parser, Expression left, Token token);
}
```

唯一不同的是左边的参数，这是我们在得到中缀标记之前解析的表达式。我们将通过另一个中缀解析器表把它连接到解析器。

对于前缀和中缀表达式有单独的表是很重要的，因为对于单个TokenType，我们通常会有前缀和中缀解析器。例如，`(`的前缀解析器会处理表达式中的分组，如`* (b + c)`。同时，中缀解析器处理函数调用，如`a(b)`。

现在，在我们解析前缀表达式之后，我们把它交给包含它的任何中缀表达式:

```java
class Parser {
  public void register(TokenType token, InfixParselet parselet) {
    mInfixParselets.put(token, parselet);
  }

  public Expression parseExpression() {
    Token token = consume();
    PrefixParselet prefix = mPrefixParselets.get(token.getType());
    if (prefix == null) throw new ParseException(
        "Could not parse \"" + token.getText() + "\".");

    Expression left = prefix.parse(this, token);

    token = lookAhead(0);
    InfixParselet infix = mInfixParselets.get(token.getType());

    // No infix expression at this point, so we're done.
    if (infix == null) return left;

    consume();
    return infix.parse(this, left, token);
  }

  // Other stuff...

  private final Map<TokenType, InfixParselet> mInfixParselets =
      new HashMap<TokenType, InfixParselet>();
}
```

先解析前缀表达式，读到一个字符以后解析另一半表达式。

相当简单。我们可以为+这样的二进制算术运算符实现一个中缀解析器，使用类似如下的东西:

```java
class BinaryOperatorParselet implements InfixParselet {
  public Expression parse(Parser parser,
      Expression left, Token token) {
    Expression right = parser.parseExpression();
    return new OperatorExpression(left, token.getType(), right);
  }
}
```

这也适用于后缀运算符。我称它们为“中缀”解析器，但它们实际上是“除了前缀以外的任何东西”。如果token前面有表达式，它将由中缀解析器处理，其中包括后缀表达式和mixfix表达式，比如`?:`。

后缀就像单token前缀解析器一样简单:它只需要将左边的表达式打包成另一个表达式:

```java
class PostfixOperatorParselet implements InfixParselet {
  public Expression parse(Parser parser, Expression left,
      Token token) {
    return new PostfixExpression(left, token.getType());
  }
}
```

mixfix也是很简单的。它看上去很像递归下降解析器。

```java
class ConditionalParselet implements InfixParselet {
  public Expression parse(Parser parser, Expression left,
      Token token) {
    Expression thenArm = parser.parseExpression();
    parser.consume(TokenType.COLON);
    Expression elseArm = parser.parseExpression();

    return new ConditionalExpression(left, thenArm, elseArm);
  }
}
```

现在我们可以解析前缀表达式、后缀、中缀，甚至是mixfix。用相当少量的代码，我们就可以解析像a + (b？c！:-d)。我们结束了，对吧？嗯……差不多了。

## 对不起，萨莉阿姨

我们的解析器可以解析所有这些东西，但是它没有用正确的优先级或关联性来解析。如果你向它抛出`a - b - c`，它会像`- (b - c)`一样解析它，这是不对的。(嗯，实际上是对的——右结合就是这样。我们需要留着它。)

最后一步是 Pratt 解析器从相当好到完全激进。我们将做两个简单的改变。我们将扩展parseExpression()以获得优先级——一个指示哪些表达式可以被调用解析的数字。如果遇到优先级低于我们允许的表达式，它会停止解析并返回到目前为止的结果。

为了进行检查，我们需要知道任何给定中缀表达式的优先级。我们将通过让解析器指定它来做到这一点:

```java
public interface InfixParselet {
  Expression parse(Parser parser, Expression left, Token token);
  int getPrecedence();
}
```

```java
public Expression parseExpression(int precedence) {
  Token token = consume();
  PrefixParselet prefix = mPrefixParselets.get(token.getType());

  if (prefix == null) throw new ParseException(
      "Could not parse \"" + token.getText() + "\".");

  Expression left = prefix.parse(this, token);

  while (precedence < getPrecedence()) {
    token = consume();

    InfixParselet infix = mInfixParselets.get(token.getType());
    left = infix.parse(this, left, token);
  }

  return left;
}
```


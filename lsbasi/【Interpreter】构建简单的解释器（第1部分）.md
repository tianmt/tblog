@[toc]
# 【Interpreter】构建简单的解释器（第1部分）

> 简单翻译了下，方便查看，水平有限，喜欢的朋友去看 [原文](https://ruslanspivak.com/lsbasi-part1/)！

> **“If you don’t know how compilers work, then you don’t know how computers work. If you’re not 100% sure whether you know how compilers work, then you don’t know how they work.”** — Steve Yegge



思考一下上面这段话。不管你是菜鸟，还是经验丰富的软件开发人员：如果你不了解编译器和解释器的工作原理，那么你就不了解计算机的工作原理。就是这么简单。



那么，你知道编译器和解释器的工作原理吗？ 我的意思是，你是否100％确定你知道它们的工作原理？ 如果你不确定。

![lsbasi_part1_i_dont_know](https://img-blog.csdnimg.cn/20181226150321161.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RNVDEyMzQyMQ==,size_16,color_FFFFFF,t_70)

或者，如果你不了解并为此感到焦虑。

![lsbasi_part1_omg](https://img-blog.csdnimg.cn/20181226150431464.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RNVDEyMzQyMQ==,size_16,color_FFFFFF,t_70)

不用担心。 如果你能坚持完成这一系列的学习，并和我一起编写一个解释器和编译器，最终你会掌握它们的工作原理。 你也将会感到快乐并且信心倍增。 至少我希望如此。

![lsbasi_part1_i_know](https://img-blog.csdnimg.cn/20181226150518899.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RNVDEyMzQyMQ==,size_16,color_FFFFFF,t_70)

为什么要学习解释器和编译器？我给你三个理由。

1. 编写一个解释器或编译器，需要具备并且能综合运用许多专业技能。 通过编写解释器或编译器，可以帮助自己提高这些专业技能，并成为更优秀的软件开发人员。 同时，学到的这些技能能帮助自己编写任何软件，而不仅仅限于解释器或编译器。
2. 想了解计算机的工作原理。 解释器和编译器通常看起来像魔法。 你不应该满足于对这种魔法的简单运用。 您希望揭开构建解释器和编译器的神秘面纱，理解它们的工作原理，并且实现对它们的控制。
3. 创造自己的编程语言或领域特定语言。 如果你想创造一门语言，则需要为其创建解释器或编译器。 最近，人们再次掀起对新编程语言的兴趣热潮。 而且几乎每天都会看到一种新的编程语言：Elixir、Go、Rust，仅仅列出这几个例子。

好了。但是，到底什么是解释器和编译器呢？

解释器或编译器的目标是将某些高级语言的源程序翻译成其他形式。 很含糊是吗？ 请忍一下，在后面的章节系列中你将切实了解源程序被翻译成了什么。

此时，你可能很好奇解释器和编译器之间的区别。 在本系列所有章节中，让我们约定，如果翻译器将源程序翻译成机器语言，那么它就是编译器。 如果翻译器处理并执行源程序，而没有首先将其翻译成机器语言，那么它就是一个解释器。 大概如下图所示：

![lsbasi_part1_compiler_interpreter](https://img-blog.csdnimg.cn/20181226150606163.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RNVDEyMzQyMQ==,size_16,color_FFFFFF,t_70)

我希望到现在你确信你真的想学习并构建一个解释器和编译器。 你希望从这个解释器系列章节中学到什么呢？

在此说明。 我们将为 Pascal 语言大子集编写一个简单的解释器。 在本系列的最后，您将会得到一个有效的 Pascal 解释器和一个像 Python 的 pdb 这样的源代码级调试器。

你可能会问，为什么选 Pascal？ 首先，它不是我为这个系列捏造的一种虚构的语言：它是一种具有许多重要语法结构的真实编程语言。 一些很久以前的，但很有用的计算机科学的书籍在他们的示例中使用 Pascal 编程语言（我知道这并不是选择这种语言来构建一个解释器的充分理由，但我认为这也是学习一种非主流语言的好机会 ：）

下面是 Pascal 编写的阶乘函数示例，可以使用自己的解释器进行解释，还可以使用后面即将编写的交互式源码级调试器进行调试：

```pascal
program factorial;

function factorial(n: integer): longint;
begin
    if n = 0 then
        factorial := 1
    else
        factorial := n * factorial(n - 1);
end;

var
    n: integer;

begin
    for n := 0 to 16 do
        writeln(n, '! = ', factorial(n));
end.
```

我们将使用 Python 语言实现 Pascal 解释器，你也可以使用你熟悉的其他任何语言，因为所遵循的实现原理不依赖于任何特定的实现语言。 好的，让我们开始吧。 各就位，预备，开始！

下面将通过编写一个简单的算术表达式解释器（也称为计算器）开始尝试编写解释器和编译器。 今天的目标很简单：让你的计算器计算两个个位整数的和，如 3 + 5。 下面是计算器的源代码，不，是解释器：

```python
# Token types
#
# EOF (end-of-file) token is used to indicate that
# there is no more input left for lexical analysis
INTEGER, PLUS, EOF = 'INTEGER', 'PLUS', 'EOF'


class Token(object):
    def __init__(self, type, value):
        # token type: INTEGER, PLUS, or EOF
        self.type = type
        # token value: 0, 1, 2. 3, 4, 5, 6, 7, 8, 9, '+', or None
        self.value = value

    def __str__(self):
        """String representation of the class instance.

        Examples:
            Token(INTEGER, 3)
            Token(PLUS '+')
        """
        return 'Token({type}, {value})'.format(
            type=self.type,
            value=repr(self.value)
        )

    def __repr__(self):
        return self.__str__()


class Interpreter(object):
    def __init__(self, text):
        # client string input, e.g. "3+5"
        self.text = text
        # self.pos is an index into self.text
        self.pos = 0
        # current token instance
        self.current_token = None

    def error(self):
        raise Exception('Error parsing input')

    def get_next_token(self):
        """Lexical analyzer (also known as scanner or tokenizer)

        This method is responsible for breaking a sentence
        apart into tokens. One token at a time.
        """
        text = self.text

        # is self.pos index past the end of the self.text ?
        # if so, then return EOF token because there is no more
        # input left to convert into tokens
        if self.pos > len(text) - 1:
            return Token(EOF, None)

        # get a character at the position self.pos and decide
        # what token to create based on the single character
        current_char = text[self.pos]

        # if the character is a digit then convert it to
        # integer, create an INTEGER token, increment self.pos
        # index to point to the next character after the digit,
        # and return the INTEGER token
        if current_char.isdigit():
            token = Token(INTEGER, int(current_char))
            self.pos += 1
            return token

        if current_char == '+':
            token = Token(PLUS, current_char)
            self.pos += 1
            return token

        self.error()

    def eat(self, token_type):
        # compare the current token type with the passed token
        # type and if they match then "eat" the current token
        # and assign the next token to the self.current_token,
        # otherwise raise an exception.
        if self.current_token.type == token_type:
            self.current_token = self.get_next_token()
        else:
            self.error()

    def expr(self):
        """expr -> INTEGER PLUS INTEGER"""
        # set current token to the first token taken from the input
        self.current_token = self.get_next_token()

        # we expect the current token to be a single-digit integer
        left = self.current_token
        self.eat(INTEGER)

        # we expect the current token to be a '+' token
        op = self.current_token
        self.eat(PLUS)

        # we expect the current token to be a single-digit integer
        right = self.current_token
        self.eat(INTEGER)
        # after the above call the self.current_token is set to
        # EOF token

        # at this point INTEGER PLUS INTEGER sequence of tokens
        # has been successfully found and the method can just
        # return the result of adding two integers, thus
        # effectively interpreting client input
        result = left.value + right.value
        return result


def main():
    while True:
        try:
            # To run under Python3 replace 'raw_input' call
            # with 'input'
            text = raw_input('calc> ')
        except EOFError:
            break
        if not text:
            continue
        interpreter = Interpreter(text)
        result = interpreter.expr()
        print(result)


if __name__ == '__main__':
    main()
```

将上面代码保存为 *calc1.py*  文件，或者直接从[ GitHub ](https://github.com/rspivak/lsbasi/blob/master/part1/calc1.py)中下载。 在深入研究代码之前，请先在命令行中运行它，并查看其运行情况。动手试试！下面是我的笔记本电脑上的会话示例 (如果你想在 python3 上运行， 应该用 *input* 替代 *raw_input*)：

```shell
$ python calc1.py
calc> 3+4
7
calc> 3+5
8
calc> 3+9
12
calc>
```

为确保你的简单计算器能正确执行，不抛出异常，你的输入需要满足以下规则:

- 只允许输入个位整数；
- 当前解释器只支持加法运算；
- 输入中任何位置都不允许出现空白字符；

这些规则非常有助于简化计算器。 不过不用担心，你的计算器很快就会变得复杂起来。

好了，现在让我们深入了解下你的解释器如何工作，以及它如何判别算术表达式。

当你在命令行输入表达式 3+5 时，你的解释器会得到一个字符串 “3+5”。 为了让解释器准确理解如何处理该字符串，解释器首先需要将输入的 “3+5” 分解成多个称为 token 的组件。 token 是具有类型和值的对象。 例如，对于字符串“3”，token 的类型会是 INTEGER，对应的值会是整数 3。

将输入字符串分解为 token 的过程称为 文法分析。 因此，解释器需要做的第一步是读取输入字符并将其转换为 token 流。 解释器中执行此操作的部分称为文法分析器，或简称 词法分析器。 您可能还会遇到这一组件的其他叫法，例如 scanner(扫描器) 或 tokenizer(标记符生成器)。 它们都代表相同的东西，即：解释器或编译器中将输入字符转换为 token 流的组件。

Interpreter 类中的 get_next_token 方法就是你的词法分析器。 每次调用它时，都会获得从传递给解释器的输入字符中创建的下一个 token。 让我们仔细看看方法本身，看看它是如何实现将字符串转换为 token 的。 输入字符存储在变量 text 中，变量 pos 存储该字符串的索引（将字符串视为字符数组）。 pos初始化为0，指向字符 '3'。 该方法首先检查 pos 位置的字符是否为整形数字，如果是，则 pos变量加1，并返回 INTEGER 类型的 token 实例，最后将 token 实例的值设置为字符串 “3” 的整数值，即整数 3：

![lsbasi_part1_lexer1](https://img-blog.csdnimg.cn/20181226150658773.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RNVDEyMzQyMQ==,size_16,color_FFFFFF,t_70)

索引 pos 现在指向变量 text 中的 “+” 字符。 再次调用 get_next_token 方法时，它会判断索引 pos 指向的字符是不是整形数字，然而判断该字符是一个加号。结果，索引 pos 加1， 并返回一个新创建的类型为 PLUS、值为 “+” 的 token：

![lsbasi_part1_lexer2](https://img-blog.csdnimg.cn/20181226150728727.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RNVDEyMzQyMQ==,size_16,color_FFFFFF,t_70)

索引 pos 现在指向字符 '5'。 再次调用 get_next_token 方法，它会判断 pos 指向的字符是否为整形数字，这次是整形数字，则索引 pos 加 1 ，并返回一个新的 INTEGER 类型的 token，token 值设置为整形数字 5：

![lsbasi_part1_lexer3](https://img-blog.csdnimg.cn/20181226150753743.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RNVDEyMzQyMQ==,size_16,color_FFFFFF,t_70)

因为索引 pos 现在指向字符串“3 + 5”的最后位置，再次调用 get_next_token方法时，返回 EOF token：

![lsbasi_part1_lexer4](https://img-blog.csdnimg.cn/20181226150817936.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RNVDEyMzQyMQ==,size_16,color_FFFFFF,t_70)

亲手试一试，观察、了解计算器的词法分析器组件的工作原理：

```python
>>> from calc1 import Interpreter
>>>
>>> interpreter = Interpreter('3+5')
>>> interpreter.get_next_token()
Token(INTEGER, 3)
>>>
>>> interpreter.get_next_token()
Token(PLUS, '+')
>>>
>>> interpreter.get_next_token()
Token(INTEGER, 5)
>>>
>>> interpreter.get_next_token()
Token(EOF, None)
>>>
```

现在解释器可以访问由输入字符生成的 token 流，解释器会对 token 流做进一步处理：从词法分析器 get_next_token 生成的序列化 token 流结构中。 依次找到以下结构：INTEGER  - > PLUS  - > INTEGER。 也就是说，解释器尝试找出 token 序列：先是一个整数，然后一个加号，最后一个整数。

负责查找和解释 token 结构的方法是 expr。 此方法验证输入的 token 序列是否与预期 token 序列一致，比如： INTEGER  - > PLUS  - > INTEGER。 在确认 token 序列结构无误后，expr 方法通过求 PLUS 结构左右两侧 INTEGER 结构存储值的和，得到表达式的结果。从而成功地解释用户输入到解释器的算术表达式。

expr 方法借助 eat 方法 来验证当前得到的 token 类型是否与预期 token 类型相匹配。 匹配 token 类型成功后，eat 方法会获取下一个 token 并赋值给 current_token 变量，从而有效地“吃掉”当前匹配的 token 并推进 token 流中的虚拟指针。 如果 token 流中的结构与预期 INTEGER PLUS INTEGER 的 token 序列不对应的话，eat 方法会抛出异常。



让我们回顾一下你的解释器在验证算术表达式时所做的工作：

- 解释器接受一个输入字符串，比如： “3 + 5”
- 解释器调用 expr 方法查找词法分析器 get_next_token 返回的标记流中的结构。 解释器试图找到 INTEGER PLUS INTEGER 形式的结构。 在确认结构正确后，解释器会求两个 INTEGER 类型 token 的值的和，很明显，此时解释器需要做的事就两个整数 3，5 求和运算。



祝贺你。 你已经学会了如何构建属于你的第一个解释器！

下面该开始练习了。

![lsbasi_exercises2](https://img-blog.csdnimg.cn/20181226150903886.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RNVDEyMzQyMQ==,size_16,color_FFFFFF,t_70)

你不会认为仅仅读完这篇文章就够了是吗？ 好吧，现在完成下面练习：

1. 修改代码，允许输入多位数整数，例如 “12+3”
2. 添加一个忽略空白字符的方法，让计算器可以处理带有空白字符的输入，如 “12 + 3”
3. 修改代码，使用 '-' 代替 '+' 来处理像 “7-5” 这样的减法



**测试你对文章的理解**

1. 什么是解释器？
2. 什么是编译器？
3. 解释器和编译器的区别是什么？
4. 什么是 token？
5. 将输入分解为 token 的进程的名称是什么？
6. 解释器中词法分析的部分是什么？
7. 解释器或编译器中的上题其余部件，其他通用名是什么。



在本文结束之前，我真切希望你正在学习解释器和编译器。希望你马上就学，而不是扔到一边。别等了。如果你已经概略看过本位，再读一遍。如果你认真读过，但没有完成练习——那就现在开始（做练习）。如果你没有做完，那完成它。你读懂大意，理解怎么回事了吗？签署保证书，今天就开始学习解释器和编译器!



在本文结束之前，我真的希望你致力于研究解释器和编译器。 我希望你现在就学。 不要放到一边。 不要等。如果你大概浏览了这篇文章，请重新读一遍。 如果你认真仔细阅读，但没有做过练习 — 现在就去练习。 如果你只做了其中一些，那么完成其余的练习工作。 理解怎么回事了吗？签署保证书，今天就开始学习解释器和编译器!



**I,  ____________________, of being sound mind and body, do hereby pledge to commit to studying interpreters and compilers starting today and get to a point where I know 100% how they work!**

**Signature:**

**Date:**

![lsbasi_part1_commitment_pledge](https://img-blog.csdnimg.cn/20181226150929516.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RNVDEyMzQyMQ==,size_16,color_FFFFFF,t_70)

签好名，注明日期，放到每天都能看到的显眼处，以确保你坚持你的承诺。牢记承诺的定义:

> **“Commitment is doing the thing you said you were going to do long after the mood you said it in has left you.” — Darren Hardy**



好的，今天就是这样。 在该系列文章的下一篇中，将扩展计算器以处理更多算术表达式。 敬请关注。

 

如果你等不及第二篇文章，并且正在深入挖掘解释器和编译器，那么我在下面的推荐书目将会对你有帮助：

1. Language Implementation Patterns: Create Your Own Domain-Specific and General Programming Languages (Pragmatic Programmers)
2. Writing Compilers and Interpreters: A Software Engineering Approach
3. Modern Compiler Implementation in Java
4. Modern Compiler Design
5. Compilers: Principles, Techniques, and Tools (2nd Edition)



**原文链接：**[Let’s Build A Simple Interpreter. Part 1.](https://ruslanspivak.com/lsbasi-part1/)

**作者博客：**[Ruslan's Blog](https://ruslanspivak.com/)



------
**——2018-12-26——**
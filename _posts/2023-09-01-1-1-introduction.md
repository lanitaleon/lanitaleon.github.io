原文：

https://craftinginterpreters.com/introduction.html

> Fairy tales are more than true: not because they tell us that dragons exist, but because they tell us that dragons can be beaten.
——G.K. Chesterton by way of Neil Gaiman, Coraline
>
> 童话故事不仅仅是真实的: 不是因为它们告诉我们龙的存在，而是因为它们告诉我们龙是可以被打败的。

I’m really excited we’re going on this journey together. This is a book on implementing interpreters for programming languages. It’s also a book on how to design a language worth implementing. It’s the book I wish I’d had when I first started getting into languages, and it’s the book I’ve been writing in my head for nearly a decade.

我很高兴我们能一起踏上这段旅程。这是一本关于为编程语言实现解释器的书。它也是一本关于如何设计一种值得实现的语言的书。这是我刚开始学习语言时就希望拥有的书，也是我已经在脑海中写了近十年的书。

> To my friends and family, sorry I’ve been so absentminded!
>
> 对我的朋友和家人，抱歉我一直心不在焉

In these pages, we will walk step-by-step through two complete interpreters for a full-featured language. I assume this is your first foray into languages, so I’ll cover each concept and line of code you need to build a complete, usable, fast language implementation.

在这些页面中，我们将一步一步通过两个完整的解释器为一个全功能的语言。我假设这是您第一次涉足语言，因此我将介绍构建一个完整的、可用的、快速的语言实现所需的每个概念和每行代码。

In order to cram two full implementations inside one book without it turning into a doorstop, this text is lighter on theory than others. As we build each piece of the system, I will introduce the history and concepts behind it. I’ll try to get you familiar with the lingo so that if you ever find yourself at a cocktail party full of PL (programming language) researchers, you’ll fit in.

为了在一本书中塞进两个完整的实现，而不使它变成一个门挡，这篇文章在理论上比其他文章更轻松。在构建系统的每个部分时，我将介绍系统背后的历史和概念。我会尽量让你熟悉这些术语，这样如果你在一个鸡尾酒会上发现自己身边全是 PL (编程语言)研究人员，你就能融入其中。

> Strangely enough, a situation I have found myself in multiple times. You wouldn’t believe how much some of them can drink.
>
> 奇怪的是，这种情况我遇到过很多次。你不会相信他们有多能喝。

But we’re mostly going to spend our brain juice getting the language up and running. This is not to say theory isn’t important. Being able to reason precisely and formally about syntax and semantics is a vital skill when working on a language. But, personally, I learn best by doing. It’s hard for me to wade through paragraphs full of abstract concepts and really absorb them. But if I’ve coded something, run it, and debugged it, then I get it.

但是我们大部分时间都花在了让语言运行起来上。这并不是说理论不重要。在处理语言时，能够对语法和语义进行精确和正式的推理是一项至关重要的技能。但是，就我个人而言，我通过实践学习得最好。对于我来说，费力地阅读充满抽象概念的段落并真正理解它们是很困难的。但如果我编码了什么东西，运行它，调试它，那么我就能得到它。

> Static type systems in particular require rigorous formal reasoning. Hacking on a type system has the same feel as proving a theorem in mathematics.
>
> 静态类型系统尤其需要严格的形式推理。黑进一个类型系统的感觉和证明一个数学定理的感觉是一样的。

> It turns out this is no coincidence. In the early half of last century, Haskell Curry and William Alvin Howard showed that they are two sides of the same coin: the Curry-Howard isomorphism.
>
> 事实证明这不是巧合。上世纪上半叶，Haskell Curry 和 William Alvin Howard 表明，他们是一枚硬币的两面: 柯里-霍华德同构。

That’s my goal for you. I want you to come away with a solid intuition of how a real language lives and breathes. My hope is that when you read other, more theoretical books later, the concepts there will firmly stick in your mind, adhered to this tangible substrate.

这就是我对你的目标。我希望你们能对一门真正的语言如何生存和呼吸有一个可靠的直觉。我的希望是，当你阅读其他更多的理论书籍后，那里的概念会牢牢地扎根在你的脑海里，坚持这种有形的基质。

## 1.1 Why Learn This Stuff?

Every introduction to every compiler book seems to have this section. I don’t know what it is about programming languages that causes such existential doubt. I don’t think ornithology books worry about justifying their existence. They assume the reader loves birds and start teaching.

每一本编译器书籍的介绍似乎都有这一部分。我不知道为什么编程语言会引起这种存在主义的怀疑。我不认为鸟类学书籍担心证明它们的存在。他们假设读者喜欢鸟类，然后开始教学。

But programming languages are a little different. I suppose it is true that the odds of any of us creating a broadly successful, general-purpose programming language are slim. The designers of the world’s widely used languages could fit in a Volkswagen bus, even without putting the pop-top camper up. If joining that elite group was the only reason to learn languages, it would be hard to justify. Fortunately, it isn’t.

但是编程语言有点不同。我认为，我们中的任何一个人创造一个广泛成功的通用编程语言的可能性确实很小。世界上广泛使用的语言的设计者们甚至不需要装上露营车，就可以装进一辆大众公交车。如果加入这个精英群体是学习语言的唯一理由，那么就很难证明这是合理的。幸运的是，事实并非如此。

### 1.1.1 Little languages are everywhere

For every successful general-purpose language, there are a thousand successful niche ones. We used to call them “little languages”, but inflation in the jargon economy led to the name “domain-specific languages”. These are pidgins tailor-built to a specific task. Think application scripting languages, template engines, markup formats, and configuration files.

每一种成功的通用语言，都有上千种成功的利基语言。我们过去称它们为“小语言”，但行话经济的膨胀导致了“领域特定语言”的名称。这些是为特定任务量身定制的 pidgins。考虑应用程序脚本语言、模板引擎、标记格式和配置文件。

![小语言](/crafting-interpreters/1-1-1-little-languages.png)

> A random selection of some little languages you might run into.
>
> 随机选择一些你可能会碰到的小语言。

Almost every large software project needs a handful of these. When you can, it’s good to reuse an existing one instead of rolling your own. Once you factor in documentation, debuggers, editor support, syntax highlighting, and all of the other trappings, doing it yourself becomes a tall order.

几乎每个大型软件项目都需要一些这样的软件。如果可以的话，最好重用一个现有的，而不是滚动您自己的。一旦你考虑到文档、调试器、编辑器支持、语法突显和所有其他的陷阱，自己动手就变成了一项艰巨的任务。

But there’s still a good chance you’ll find yourself needing to whip up a parser or other tool when there isn’t an existing library that fits your needs. Even when you are reusing some existing implementation, you’ll inevitably end up needing to debug and maintain it and poke around in its guts.

但是，如果没有适合您需要的现有库，那么您仍然很有可能需要启动解析器或其他工具。即使在重用某些现有实现时，您也不可避免地需要调试和维护它，并在其内部进行探索。

### 1.1.2 Languages are great exercise

Long distance runners sometimes train with weights strapped to their ankles or at high altitudes where the atmosphere is thin. When they later unburden themselves, the new relative ease of light limbs and oxygen-rich air enables them to run farther and faster.

长跑运动员有时会将重物绑在脚踝上或在空气稀薄的高海拔地区进行训练。 当他们后来卸下负担时，轻质的四肢和富含氧气的空气使他们能够跑得更远更快。

Implementing a language is a real test of programming skill. The code is complex and performance critical. You must master recursion, dynamic arrays, trees, graphs, and hash tables. You probably use hash tables at least in your day-to-day programming, but do you really understand them? Well, after we’ve crafted our own from scratch, I guarantee you will.

实现一种语言是对编程技能的真正考验。 代码很复杂并且性能至关重要。 您必须掌握递归、动态数组、树、图和哈希表。 您可能至少在日常编程中使用哈希表，但您真的了解它们吗？ 好吧，在我们从头开始制作自己的产品后，我保证您会的。

While I intend to show you that an interpreter isn’t as daunting as you might believe, implementing one well is still a challenge. Rise to it, and you’ll come away a stronger programmer, and smarter about how you use data structures and algorithms in your day job.

虽然我打算向您展示口译员并不像您想象的那么令人畏惧，但良好地实施口译员仍然是一项挑战。 勇敢地面对它，你会成为一名更强大的程序员，并且在日常工作中更聪明地使用数据结构和算法。

### 1.1.3 One more reason

This last reason is hard for me to admit, because it’s so close to my heart. Ever since I learned to program as a kid, I felt there was something magical about languages. When I first tapped out BASIC programs one key at a time I couldn’t conceive how BASIC itself was made.

最后一个原因让我很难承认，因为它是如此贴近我的内心。 自从我小时候学习编程以来，我就觉得语言有一些神奇的东西。 当我第一次按一个键敲出 BASIC 程序时，我无法想象 BASIC 本身是如何制作的。

Later, the mixture of awe and terror on my college friends’ faces when talking about their compilers class was enough to convince me language hackers were a different breed of human—some sort of wizards granted privileged access to arcane arts.

后来，当我的大学朋友们谈论他们的编译器课程时，脸上混合着敬畏和恐惧的表情足以让我相信语言黑客是另一种人类——某种授予了神秘艺术特权的巫师。

It’s a charming image, but it has a darker side. I didn’t feel like a wizard, so I was left thinking I lacked some inborn quality necessary to join the cabal. Though I’ve been fascinated by languages ever since I doodled made-up keywords in my school notebook, it took me decades to muster the courage to try to really learn them. That “magical” quality, that sense of exclusivity, excluded me.

这是一个迷人的形象，但它也有阴暗的一面。 我不觉得自己像个巫师，所以我觉得自己缺乏加入阴谋集团所需的一些天生品质。 尽管自从我在学校笔记本上乱写了虚构的关键词以来，我就对语言着迷，但我花了几十年的时间才鼓起勇气尝试真正学习它们。 那种“神奇”的品质，那种排他性的感觉，将我排除在外。

> And its practitioners don’t hesitate to play up this image. Two of the seminal texts on programming languages feature a dragon and a wizard on their covers.
>
> 它的实践者毫不犹豫地渲染这个形象。 两本关于编程语言的开创性文本的封面上有一条龙和一个巫师。

When I did finally start cobbling together my own little interpreters, I quickly learned that, of course, there is no magic at all. It’s just code, and the people who hack on languages are just people.

当我最终开始拼凑我自己的小解释器时，我很快意识到，当然，根本没有魔法。 它只是代码，而破解语言的人也只是人。

There are a few techniques you don’t often encounter outside of languages, and some parts are a little difficult. But not more difficult than other obstacles you’ve overcome. My hope is that if you’ve felt intimidated by languages and this book helps you overcome that fear, maybe I’ll leave you just a tiny bit braver than you were before.

有一些技巧是你在语言之外不常遇到的，而且有些部分有点困难。 但并不比你克服的其他障碍更困难。 我的希望是，如果你对语言感到害怕，而这本书可以帮助你克服这种恐惧，也许我会让你比以前更勇敢一点。

And, who knows, maybe you will make the next great language. Someone has to.

而且，谁知道呢，也许你会创造出下一种伟大的语言。 必须有人这么做。

## 1.2 How the Book Is Organized

This book is broken into three parts. You’re reading the first one now. It’s a couple of chapters to get you oriented, teach you some of the lingo that language hackers use, and introduce you to Lox, the language we’ll be implementing.

本书分为三个部分。 你现在正在读第一部分。 这几章可以帮助您了解方向，教您一些语言黑客使用的行话，并向您介绍我们将要实现的语言 Lox。

Each of the other two parts builds one complete Lox interpreter. Within those parts, each chapter is structured the same way. The chapter takes a single language feature, teaches you the concepts behind it, and walks you through an implementation.

其他两部分分别构建一个完整的 Lox 解释器。 在这些部分中，每一章的结构都相同。 本章通过单个语言特性，教您其背后的概念，并引导您完成实现。

It took a good bit of trial and error on my part, but I managed to carve up the two interpreters into chapter-sized chunks that build on the previous chapters but require nothing from later ones. From the very first chapter, you’ll have a working program you can run and play with. With each passing chapter, it grows increasingly full-featured until you eventually have a complete language.

我花了很多时间尝试和犯错，但我设法将这两个解释器分成章节大小的块，这些块建立在前面的章节的基础上，但不需要后面的章节的任何内容。 从第一章开始，您将拥有一个可以运行和使用的工作程序。 每读完一章，它的功能就会变得越来越齐全，直到您最终拥有了一门完整的语言。

Aside from copious, scintillating English prose, chapters have a few other delightful facets:

除了丰富、精彩的英语段落之外，章节还有其他一些令人愉快的方面：

### 1.2.1 The code

We’re about crafting interpreters, so this book contains real code. Every single line of code needed is included, and each snippet tells you where to insert it in your ever-growing implementation.

我们致力于打造解释器，因此本书包含真实的代码。 所需的每一行代码都包含在内，每个片段都会告诉您将其插入到不断增长的实现中的何处。

Many other language books and language implementations use tools like Lex and Yacc, so-called compiler-compilers, that automatically generate some of the source files for an implementation from some higher-level description. There are pros and cons to tools like those, and strong opinions—some might say religious convictions—on both sides.

许多其他语言书籍和语言实现都使用 [Lex](https://en.wikipedia.org/wiki/Lex_(software)) 和 [Yacc](https://en.wikipedia.org/wiki/Yacc) 等工具，即所谓的编译器-编译器，它们可以根据一些更高级别的（语法）描述自动生成一些源文件。 此类工具各有利弊，观点双方各执一词（有些人可能会说是信仰）。

> Yacc is a tool that takes in a grammar file and produces a source file for a compiler, so it’s sort of like a “compiler” that outputs a compiler, which is where we get the term “compiler-compiler”.
>
> Yacc 是一个接受语法文件并为编译器生成源文件的工具，所以它有点像一个输出编译器的“编译器”，这就是我们得到术语“编译器-编译器”的地方。
>
> Yacc wasn’t the first of its ilk, which is why it’s named “Yacc”—Yet Another Compiler-Compiler. A later similar tool is Bison, named as a pun on the pronunciation of Yacc like “yak”.
>
> Yacc 并不是同类产品中的第一个，这就是为什么它被命名为“Yacc”——Yet Another Compiler-Compiler。 后来的一个类似工具是 [Bison](https://en.wikipedia.org/wiki/GNU_Bison)，它的名字是 Yacc 发音的双关语，就像“yak”一样。
>
> ![yak](/crafting-interpreters/1-2-1-yak.png)
>
> If you find all of these little self-references and puns charming and fun, you’ll fit right in here. If not, well, maybe the language nerd sense of humor is an acquired taste.
>
> 如果您发现所有这些小自我引用和双关语既迷人又有趣，那么您就适合这里。 如果不是，那么，也许语言书呆子的幽默感是一种后天习得的品味。

We will abstain from using them here. I want to ensure there are no dark corners where magic and confusion can hide, so we’ll write everything by hand. As you’ll see, it’s not as bad as it sounds, and it means you really will understand each line of code and how both interpreters work.

我们将在这里放弃使用它们。 我想确保没有黑暗的角落可以隐藏魔法和混乱，所以我们将手写所有内容。 正如您将看到的，它并不像听起来那么糟糕，这意味着您将真正理解每一行代码以及两个解释器的工作原理。

A book has different constraints from the “real world” and so the coding style here might not always reflect the best way to write maintainable production software. If I seem a little cavalier about, say, omitting private or declaring a global variable, understand I do so to keep the code easier on your eyes. The pages here aren’t as wide as your IDE and every character counts.

为一本书写代码和在“现实世界”中编程是有区别的，因此这里的编码风格可能并不总是符合编写可维护生产软件的最佳实践。 如果我看起来有点漫不经心，比如省略 `private` 或声明全局变量，请理解我这样做是为了让代码更容易被理解。 书页不像您的 IDE 那么宽，所以每个字符都很重要。

Also, the code doesn’t have many comments. That’s because each handful of lines is surrounded by several paragraphs of honest-to-God prose explaining it. When you write a book to accompany your program, you are welcome to omit comments too. Otherwise, you should probably use // a little more than I do.

另外，代码没有太多注释。 这是因为每一小段代码都环绕着几段解释性的描述。 当你为你的程序写一本书时，也欢迎你省略评论。 否则，您可能要比我多用一些 `//` 。

While the book contains every line of code and teaches what each means, it does not describe the machinery needed to compile and run the interpreter. I assume you can slap together a makefile or a project in your IDE of choice in order to get the code to run. Those kinds of instructions get out of date quickly, and I want this book to age like XO brandy, not backyard hooch.

虽然本书包含了每一行代码并讲解了每行代码的含义，但它没有描述编译和运行解释器所需的机制。 我假设您可以简单地拼凑出一个 makefile 或者在您选择的 IDE 中创建一个工程项目，以便让代码运行。 类似这样的说明很快就会过时，我希望这本书像 XO 白兰地一样历久弥新，而不是像自酿一样很快就会过期。

### 1.2.2 Snippets

Since the book contains literally every line of code needed for the implementations, the snippets are quite precise. Also, because I try to keep the program in a runnable state even when major features are missing, sometimes we add temporary code that gets replaced in later snippets.

由于本书实际上包含了实现所需的每一行代码，因此这些片段非常精确。 另外，因为即使缺少主要功能，我也会尝试让程序保持可运行的状态，因此有时我们会添加临时代码，这些代码后续就会被替换掉。

A snippet with all the bells and whistles looks like this:

包含完整功能的片段如下所示：

> lox/Scanner.java
> in scanToken()
> replace 1 line

```java
default:
```

```java
	if (isDigit(c)) {
  		number();
	} else {
  		Lox.error(line, "Unexpected character.");
	}
```

```java	
	break;
```

In the center, you have the new code to add. It may have a few faded out lines above or below to show where it goes in the existing surrounding code. There is also a little blurb telling you in which file and where to place the snippet. If that blurb says “replace _ lines”, there is some existing code between the faded lines that you need to remove and replace with the new snippet.

中间彩色的部分是要添加的新代码。 它的上方或下方可能有一些淡出的行，用来显示它在现有代码中的位置。 还有一个小简介告诉您在哪个文件中以及在何处放置代码片段。 如果该简介说“replace _ lines”，则代表淡出行之间存在一些现有代码，您需要将其删除并替换为新代码段。

> 在本译文中，淡出行和新代码会分开放在衔接的不同代码块里。

### 1.2.3 Asides

Asides contain biographical sketches, historical background, references to related topics, and suggestions of other areas to explore. There’s nothing that you need to know in them to understand later parts of the book, so you can skip them if you want. I won’t judge you, but I might be a little sad.

旁白包含传记概要、历史背景、相关主题的参考资料以及其他需要探索的领域的建议。 您无需了解其中的任何内容即可理解本书的后续部分，因此如果您愿意，可以跳过它们。 我不会评判你，但我可能会有点难过。

> Well, some asides do, at least. Most of them are just dumb jokes and amateurish drawings.
>
> 嗯，至少有些旁白是这样的。 其中大多数只是愚蠢的笑话和业余绘画。

### 1.2.4 Challeges

Each chapter ends with a few exercises. Unlike textbook problem sets, which tend to review material you already covered, these are to help you learn more than what’s in the chapter. They force you to step off the guided path and explore on your own. They will make you research other languages, figure out how to implement features, or otherwise get you out of your comfort zone.

每章最后都有一些练习。 与教科书习题集不同的是，教科书习题集往往会复习您已经涵盖的材料，这些习题集旨在帮助您学习本章之外的更多内容。 它们迫使你离开引导的道路并自行探索。 它们会让你研究其他语言，弄清楚如何实现功能，或者以其他方式让你走出舒适区。

Vanquish the challenges and you’ll come away with a broader understanding and possibly a few bumps and scrapes. Or skip them if you want to stay inside the comfy confines of the tour bus. It’s your book.

克服这些挑战，你将会获得更广泛的理解，也可能会经历一些坎坷和擦伤。 或者，如果您想留在舒适的旅游巴士内，则可以跳过它们。 这是你的书。

> A word of warning: the challenges often ask you to make changes to the interpreter you’re building. You’ll want to implement those in a copy of your code. The later chapters assume your interpreter is in a pristine (“unchallenged”?) state.
>
> 警告：挑战通常要求您对正在构建的解释器进行更改。 您需要在代码副本中实现这些内容。 后面的章节假设你的解释器处于原始（“不受挑战”？）状态。

### 1.2.5 Design notes

Most “programming language” books are strictly programming language implementation books. They rarely discuss how one might happen to design the language being implemented. Implementation is fun because it is so precisely defined. We programmers seem to have an affinity for things that are black and white, ones and zeroes.

大多数“编程语言”书籍都是严格意义上的编程语言实现书籍。 他们很少讨论如何设计正在实现的语言。 实现很有趣，因为它的定义如此精确。 我们程序员似乎对黑与白、一和零的事物有着浓厚的兴趣。

> I know a lot of language hackers whose careers are based on this. You slide a language spec under their door, wait a few months, and code and benchmark results come out.
>
> 我认识很多语言黑客，他们的职业生涯都是以此为基础的。 你将语言规范塞进他们的门里，等待几个月，代码和基准测试结果就会出来。

Personally, I think the world needs only so many implementations of FORTRAN 77. At some point, you find yourself designing a new language. Once you start playing that game, then the softer, human side of the equation becomes paramount. Things like which features are easy to learn, how to balance innovation and familiarity, what syntax is more readable and to whom.

就我个人而言，我认为世界只需要这么多的 FORTRAN 77 实现。在某些时候，您会发现自己正在设计一种新语言。 一旦你开始玩这个游戏，那么柔和、人性化的一面就变得至关重要。 比如哪些功能容易学习、如何平衡创新和熟悉度、哪些语法更易读以及对谁来说更易读。

> Hopefully your new language doesn’t hardcode assumptions about the width of a punched card into its grammar.
>
> 希望您的新语言不会将有关打孔卡宽度的假设硬编码到其语法中。

All of that stuff profoundly affects the success of your new language. I want your language to succeed, so in some chapters I end with a “design note”, a little essay on some corner of the human aspect of programming languages. I’m no expert on this—I don’t know if anyone really is—so take these with a large pinch of salt. That should make them tastier food for thought, which is my main aim.

所有这些都会深刻地影响你的新语言的成功。 我希望你的语言取得成功，因此在某些章节中我会以一篇关于编程语言人性化方面的“设计说明”结尾。 我不是这方面的专家——我不知道是否有人真的是——所以对这些持保留态度。 这应该会让它们变得更引人入胜，这是我的主要目标。

## 1.3 The First Interpreter

We’ll write our first interpreter, jlox, in Java. The focus is on concepts. We’ll write the simplest, cleanest code we can to correctly implement the semantics of the language. This will get us comfortable with the basic techniques and also hone our understanding of exactly how the language is supposed to behave.

我们将用 Java 编写我们的第一个解释器 jlox。 重点是概念。 我们将编写最简单、最干净的代码来正确实现该语言的语义。 这将使我们熟悉基本技术，并磨练我们对语言应该如何表现的理解。

> The book uses Java and C, but readers have ported the code to [many other languages](https://github.com/munificent/craftinginterpreters/wiki/Lox-implementations). If the languages I picked aren’t your bag, take a look at those.
>
> 本书使用 Java 和 C，但读者已将代码移植到许多其他语言。 如果我选择的语言不适合您，请看看这些版本的实现。

Java is a great language for this. It’s high level enough that we don’t get overwhelmed by fiddly implementation details, but it’s still pretty explicit. Unlike in scripting languages, there tends to be less complex machinery hiding under the hood, and you’ve got static types to see what data structures you’re working with.

Java 是一门很棒的语言。 它的级别足够高，以至于我们不会被繁琐的实现细节所淹没，但代码仍然非常明确。 与脚本语言不同，它的底层机制不太复杂，并且您可以使用静态类型来查看正在使用的数据结构。

I also chose Java specifically because it is an object-oriented language. That paradigm swept the programming world in the ’90s and is now the dominant way of thinking for millions of programmers. Odds are good you’re already used to organizing code into classes and methods, so we’ll keep you in that comfort zone.

我还专门选择了Java，因为它是一种面向对象的语言。 这种范式在 90 年代席卷了编程世界，现在已成为数百万程序员的主导思维方式。 您很可能已经习惯了将代码组织到类和方法中，因此我们会让您保持在舒适区。

While academic language folks sometimes look down on object-oriented languages, the reality is that they are widely used even for language work. GCC and LLVM are written in C++, as are most JavaScript virtual machines. Object-oriented languages are ubiquitous, and the tools and compilers for a language are often written in the same language.

虽然学术语言专家有时会看不起面向对象语言，但事实是它们甚至被广泛用于语言工作。 GCC 和 LLVM 是用 C++ 编写的，大多数 JavaScript 虚拟机也是如此。 面向对象的语言无处不在，并且一种语言的工具和编译器通常是用同一种语言编写的。

> A compiler reads files in one language, translates them, and outputs files in another language. You can implement a compiler in any language, including the same language it compiles, a process called self-hosting.
> 
> 编译器读取一种语言的文件，翻译它们，然后输出另一种语言的文件。 您可以用任何语言实现编译器，包括它编译的相同语言，这个过程称为自托管。
>
> You can’t compile your compiler using itself yet, but if you have another compiler for your language written in some other language, you use that one to compile your compiler once. Now you can use the compiled version of your own compiler to compile future versions of itself, and you can discard the original one compiled from the other compiler. This is called bootstrapping, from the image of pulling yourself up by your own bootstraps.
>
> 您还不能使用编译器本身来编译您的编译器，但是如果您有另一个用其他语言编写的语言的编译器，则可以使用该编译器来编译您的编译器一次。 现在，您可以使用自己的编译器的编译版本来编译其自身的未来版本，并且可以丢弃从其他编译器编译的原始版本。 这被称为“自力更生”，意为通过自己的引导来提升自己。
> ![bootstrap](/crafting-interpreters/1-3-1-bootstrap.png)

And, finally, Java is hugely popular. That means there’s a good chance you already know it, so there’s less for you to learn to get going in the book. If you aren’t that familiar with Java, don’t freak out. I try to stick to a fairly minimal subset of it. I use the diamond operator from Java 7 to make things a little more terse, but that’s about it as far as “advanced” features go. If you know another object-oriented language, like C# or C++, you can muddle through.

最后，Java 非常受欢迎。 这意味着你很可能已经知道了，所以你需要学习的内容就更少了。 如果您对 Java 不太熟悉，请不要惊慌。 我尝试坚持其中相当小的一个子集。 我使用 Java 7 中的尖括号运算符使事情变得更简洁，但就“高级”功能而言仅此而已。 如果您了解另一种面向对象语言，例如 C# 或 C++，那您完全可以蒙混过关。

By the end of part II, we’ll have a simple, readable implementation. It’s not very fast, but it’s correct. However, we are only able to accomplish that by building on the Java virtual machine’s own runtime facilities. We want to learn how Java itself implements those things.

在第二部分结束时，我们将得到一个简单、可读的实现。 它不是很快，但它是正确的。 然而，我们只能通过构建在 Java 虚拟机自己的运行时设施上来实现这一点。 我们想了解Java本身是如何实现这些东西的。

## 1.4 The Second Interpreter

So in the next part, we start all over again, but this time in C. C is the perfect language for understanding how an implementation really works, all the way down to the bytes in memory and the code flowing through the CPU.

因此，在下一部分中，我们将重新开始，但这次是使用 C。C 是理解实现如何真正工作的完美语言，一直到内存中的字节和流经 CPU 的代码。

A big reason that we’re using C is so I can show you things C is particularly good at, but that does mean you’ll need to be pretty comfortable with it. You don’t have to be the reincarnation of Dennis Ritchie, but you shouldn’t be spooked by pointers either.

我们使用 C 的一个重要原因是我可以向您展示 C 特别擅长的事情，但这确实意味着您需要对它非常熟悉。 你不必成为丹尼斯·里奇的转世，但你也不应该被指针吓到。

If you aren’t there yet, pick up an introductory book on C and chew through it, then come back here when you’re done. In return, you’ll come away from this book an even stronger C programmer. That’s useful given how many language implementations are written in C: Lua, CPython, and Ruby’s MRI, to name a few.

如果您对 C 还没了解到这个程度，请拿起一本 C 语言入门书籍并仔细阅读，完成后再回到这里。 作为回报，读完这本书你将成为一名更强大的 C 程序员。 考虑到有多少语言实现是用 C 编写的：Lua、CPython 和 Ruby 的 MRI 等等，这非常有用。

In our C interpreter, clox, we are forced to implement for ourselves all the things Java gave us for free. We’ll write our own dynamic array and hash table. We’ll decide how objects are represented in memory, and build a garbage collector to reclaim them.

在我们的 C 解释器 clox 中，我们被迫自己实现 Java 免费提供给我们的所有东西。 我们将编写自己的动态数组和哈希表。 我们将决定对象在内存中的表示方式，并构建一个垃圾收集器来回收它们。

> I pronounce the name (clox) like “sea-locks”, but you can say it “clocks” or even “cloch”, where you pronounce the “x” like the Greeks do if it makes you happy.
>
> 我把这个名字 clox 读成“sea-locks”，但你可以说它是“clocks”，甚至是“cloch”，如果你高兴的话，你可以像希腊人一样发音“x”。

Our Java implementation was focused on being correct. Now that we have that down, we’ll turn to also being fast. Our C interpreter will contain a compiler that translates Lox to an efficient bytecode representation (don’t worry, I’ll get into what that means soon), which it then executes. This is the same technique used by implementations of Lua, Python, Ruby, PHP, and many other successful languages.

我们的 Java 实现注重正确性。 现在我们已经解决了正确性，我们将转向快速。 我们的 C 解释器将包含一个编译器，它将 Lox 转换为高效的字节码表示形式（别担心，我很快就会解释这是什么意思），然后执行它。 这与 Lua、Python、Ruby、PHP 和许多其他成功语言的实现所使用的技术相同。

> Did you think this was just an interpreter book? It’s a compiler book as well. Two for the price of one!
>
> 你以为这只是一本讲解释器的书吗？ 这也是一本讲编译器的书。 买一送一！

We’ll even try our hand at benchmarking and optimization. By the end, we’ll have a robust, accurate, fast interpreter for our language, able to keep up with other professional caliber implementations out there. Not bad for one book and a few thousand lines of code.

我们甚至会尝试进行基准测试和优化。 到最后，我们将为我们的语言提供一个强大、准确、快速的解释器，它可与其他专业水准的解释器比肩。 对于一本书和几千行代码来说已经不错了。

### Challenges

1) There are at least six domain-specific languages used in the [little system I cobbled together](https://github.com/munificent/craftinginterpreters) to write and publish this book. What are they?

1. 在我拼凑起来编写和出版这本书的小系统中，至少使用了六种特定于领域的语言。 这些是什么？

2) Get a “Hello, world!” program written and running in Java. Set up whatever makefiles or IDE projects you need to get it working. If you have a debugger, get comfortable with it and step through your program as it runs.

2. 用Java写一个“Hello, world!”程序并运行它。 设置让它正常运行所需的任何 makefile 或 IDE 项目。 如果您有调试器，请熟悉它并在程序运行时逐步执行程序。

3) Do the same thing for C. To get some practice with pointers, define a [doubly linked list](https://en.wikipedia.org/wiki/Doubly_linked_list) of heap-allocated strings. Write functions to insert, find, and delete items from it. Test them.

3. 对 C 执行同样的操作。为了进行一些指针练习，请定义一个堆分配字符串的双向链表。 编写函数进行增删改查。 测试它们。

### Design Note: What's in a name?

One of the hardest challenges in writing this book was coming up with a name for the language it implements. I went through pages of candidates before I found one that worked. As you’ll discover on the first day you start building your own language, naming is deviously hard. A good name satisfies a few criteria:

撰写本书时最困难的挑战之一是为其实现的语言命名。 在找到一个有效的候选名之前，我浏览了一页又一页的候选名。 当你某一天开始构建自己的语言时，就会发现命名非常得困难。 一个好的名字需要满足以下几个条件：

1) **It isn’t in use.** You can run into all sorts of trouble, legal and social, if you inadvertently step on someone else’s name.

1. **尚未使用。**如果您无意中用了别人取的名字，您可能会遇到各种麻烦，无论是法律上的还是社会上的。

2) **It’s easy to pronounce.** If things go well, hordes of people will be saying and writing your language’s name. Anything longer than a couple of syllables or a handful of letters will annoy them to no end.

2. **容易发音。** 如果一切顺利，一大群人会说并写下你的语言的名字。 任何超过几个音节或几个字母的东西都会让他们烦恼不已。

3) **It’s distinct enough to search for.** People will Google your language’s name to learn about it, so you want a word that’s rare enough that most results point to your docs. Though, with the amount of AI search engines are packing today, that’s less of an issue. Still, you won’t be doing your users any favors if you name your language “for”.

3. **足够独特，便于搜索。** 人们会通过谷歌搜索您的语言名称来了解它，因此您需要一个足够罕见的单词，以便大多数结果都指向您的文档。 不过，随着如今人工智能搜索引擎的数量不断增加，这已经不是什么问题了。 尽管如此，如果你将你的语言命名为“for”，你不会给你的用户带来任何帮助。

4) **It doesn’t have negative connotations across a number of cultures.** This is hard to be on guard for, but it’s worth considering. The designer of Nimrod ended up renaming his language to “Nim” because too many people remember that Bugs Bunny used “Nimrod” as an insult. (Bugs was using it ironically.)

4. **在大多数文化中都没有负面含义。** 这很难保证，但值得考虑。 Nimrod 的设计者最终将他的语言重命名为“Nim”，因为太多人记得兔八哥使用“Nimrod”作为一种侮辱。 （兔八哥讽刺地使用了它。）

If your potential name makes it through that gauntlet, keep it. Don’t get hung up on trying to find an appellation that captures the quintessence of your language. If the names of the world’s other successful languages teach us anything, it’s that the name doesn’t matter much. All you need is a reasonably unique token.

如果你的潜在名字能够通过考验，那就保留它。 不要沉迷于寻找一个能够体现您的语言精髓的称谓。 如果说世界上其他成功语言的名字能教会我们什么的话，那就是名字并不重要。 您所需要的只是一个相当独特的标记。

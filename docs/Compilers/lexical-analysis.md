# 词法分析器

在编译器理解输入程序的三个过程中，词法分析是第一阶段。词法分析器读取字符流，并产生单词流。它聚合字符形成单词，并应用一组规则来判断每个单词在源语言中是否合法。如果单词有效，词法分析器会分配给它一个语法范畴或词类。

## 概念路线图

1. 识别器：识别器是在字符流中识别特定单词的程序。有限自动机是识别器的一种形式化方法。
2. 正则表达式：正则表达式是一种用于定义语法的形式记号。
3. 一组构造方法，用于将正则表达式转换为识别器。
4. 实现词法分析器的三种不同的方式：表驱动词法分析器、直接编码的词法分析器和手工编码方法。

## 识别单词

逐字符的词法分析方法会让算法清晰易懂。我们可以用转移图表示逐字符处理的词法分析器，转移图又相当于一个有限自动机。转移图分为无环和有环，无限集需要有环的转移图。<br>
对于需要实现转移图（transition diagram）的代码来说，转移图充当了这些代码的抽象。转移图还可以看作是形式化的数学对象，成为有限自动机，它定义了识别器的规格。<br>
形式上，有限自动机（FA）是一个五元数组(S, Σ, δ, s<sub>0</sub>, S<sub>A</sub>)，其中各分量的含义如下所示：
- S是识别器中的有限状态集，
- Σ是识别器中使用的有限字母表。通常，Σ是转移图中边的标签的合集。
- δ(s, c)是识别器的转移函数。它将每个状态s ∈ S和每个字符c ∈ Σ的组合(s, c)映射到下一个状态。在状态s<sub>i</sub>遇到输入字符c，FA将采用转移s<sub>i</sub>→<sub>c</sub>δ(s<sub>i</sub>, c)。
- s<sub>0</sub> ∈ S是指定的起始状态。
- s<sub>A</sub>是接受状态的集合，s<sub>A</sub> ⊆ S。 s<sub>A</sub>中的每个状态都在转移图中表示为双层圆圈。

FA接受字符串的三种情况：
1. （正确）每个字符输入符合相应的状态转移，并在耗尽所有字符输入后处于接受状态。
2. （错误）FA在处理字符串时遇到错误——即某个字符x<sub>j</sub>可能使FA转移到错误状态s<sub>e</sub>。这种情况表明发生了词法错误。
3. （错误）FA耗尽其输入后，终止于s<sub>e</sub>之外的某个非接受状态。


## 正则表达式

有限自动机F所接受的单词的集合，形成了一种语言，记作L(F)。FA的转移图详细地定义了该语言，但转移图不够直观。对于任意FA，我们还可以使用一种称为正则表达式（Regular Expression, RE）的符号表示法来描述其语言。通过RE描述的语言称为正则语言。<br>
一个RE描述了一个定义在某个字母表Σ上的字符串集合，外加一个表示空串的字符ε。我们将字符串的这种集合称为一种语言。一个RE由三个基本的操作构建而成：

1. 选择
2. 链接
3. 闭包：分为柯林闭包R<sub>\*</sub>和正闭包R<sub>+</sub>

其他操作有：求补运算符^，转义序列。

操作优先级：
- 括号 > 求补 > 闭包 > 连接 > 选择

FA的开销：
- FA的运行开销与输入长度成正比，和生成FA的RE长度或复杂性无关。更复杂的RE可能产生具有更多状态的FA，而这种FA又需要更多空间。

程序设计语言和自然语言的异同：
- 针对英文的词法分析器也可以使用基于FA的技术来识别可能的单词，因为所有的英语单词都取自一个有限的字母表。但此后，词法分析器还必须在字典中查找目标单词，以判断是否是一个真实存在的单词。同时，在很多情况下，要从语法上理解上下文才能归类此单词。有些情况下对单词及其上下文都需要进行语义上的理解。
- 相对而言，程序设计语言中的单词几乎总是从词法上就能够规定的，如`[a...z]([a...z]|[0...9])*`定义了Algol标志符的一个子集，而无需通过查找进行证实。当然，一些标志符可以保留用做关键字。但这些例外情况也可以从词法上规定，不需要上下文。

## 从正则表达式到词法分析器

![构造法的循环](/res/构造法的循环.png)

对于有限自动机来说，我们的目标是，使得从一组RE导出可执行词法分析器的过程自动化。

### 非确定性有限自动机

#### NFA与DFA的异同

- RE中将空串规定为ε，在FA中ε也有应用。ε转移是针对空串输入ε进行的转移，不会改变输入流中的读写位置。可以使用针对ε输入的转移来合并FA，并组成更复杂RE的FA。
- 但利用ε转移合并两个FA，可能会使关于FA工作方式的模型复杂化。
- 如果一个FA包含了一种对单个输入字符有多种可能的转移的状态，则称为非确定性有限自动机。即DFA就是允许在空串输入ε上进行转移的FA。
- 相反，如果FA中的每个状态对任一输入字符都具有唯一可能的转移，则称为确定性有限自动机。DFA不允许ε转移。

#### NFA的两种解释模型

1. 每次NFA必须进行非确定性选择时，如果有使得输入字符串转向接受状态的转移存在，则采用这样的转移。本质上，NFA在每个状态都需要猜测正确的转移。
2. 每次NFA必须进行非确定性选择时，NFA都克隆自身，以追踪每个可能的转移。因而，对于一个给定的输入字符，NFA实际上是处于一个特定的状态集合，其中每个状态都由NFA的某个克隆来处理。NFA并发地追踪所有转移路径。在任一时刻，存在NFA克隆副本活动状态的那些集合称为NFA的配置。当NFA到达一个配置，此时NFA已经耗尽输入字符串，且配置中的一个或多个克隆副本处于某个接受状态，则NFA接受该输入字符串。

#### NFA与DFA的等价性

NFA和DFA在表达力上是等价的。任何DFA都是某个NFA的一个特例。任何NFA都可以通过一个DFA模拟，这一事实可以通过子集构造法确立。

### 从正则表达式到NFA：Thompson构造法

Thompson构造法有一个模版，用于构建对应于单字母RE的NFA，还有一种NFA上的转换，模拟了连接、选择和闭包等各个基本RE运算符的效果。

1. a的NFA ![a的NFA](/res/a的NFA.png)
2. b的NFA ![b的NFA](/res/b的NFA.png)
3. ab的NF ![ab的NFA](/res/ab的NFA.png)
4. a\|b的NFA ![a|b的NFA](/res/a或b的NFA.png)
5. a\*的NFA ![a*的NFA](/res/a闭包的NFA.png)

这个构造法从为输入RE中每个字符构建简单的NFA开始。接下来，它按照优先级和括号规定的顺序，对简单NFA的集合应用选择、连接和闭包等转换。对于a(b\|c)\*，按a、b、c、b\|c（括号优先级最高）、(b\|c)\*（闭包先于连接）、a(b\|c)\*的顺序构建。

### 从NFA到DFA：子集构造法

#### 目标

- 子集构造法以NFA(N, Σ, δ<sub>N</sub>, n<sub>0</sub>, N<sub>A</sub>)为输入，生成一个DFA(D, Σ, δ<sub>D</sub>, d<sub>0</sub>, D<sub>A</sub>)。
- NFA和DFA使用同样的字母表Σ，DFA的起始状态d<sub>0</sub>和接受状态D<sub>A</sub>可通过构造法逐渐得到。
- 复杂的部分是从N推导D，以及推导δ<sub>D</sub>。

#### 算法

- 该算法构造了一个集合Q，其每个元素qi都是N（NFA的状态集合）的一个子集。这个算法中qi都是NFA的一个有效配置（NFA的各个配置中，可以通过某个输入字符串到达的配置）。
- 集合WorkList与集合Q有着同种元素。
- 函数ε-closure(S)的输入S是一个由NFA状态构成的集合，该计算检查每个状态si ∈ S，并将从si出发通过一个或多个ε转移所能达到的任何状态都添加到S中，并返回集合S。
- 函数Delta(q, c)将NFA的转移函数应用到集合q中的每个元素，返回∪<sub>s∈q</sub>δ<sub>N</sub>(s, c)，即转移后的状态的集合。

```
q0 = ε-closure({n0})
Q = {q0}
WorkList = {q0}

while (WorkList != 0)
    remove q from WorkList
    for each character c ∈ Σ
        t = ε-closure(Delta(q, c))
        T[q, c] = t
        if t ∉ 0
            add t to Q and to WorkList
```

- q0是NFA起始状态n0的ε转移所到达状态的集合，Q和WorkList最初仅只有q0一个元素。
- 每经过一次while循环，WorkList中移出一个有效配置q。对于每个字符c（内层循环），q读取c后转移并ε转移到的状态的集合构成新配置t。
- t将转移（配置和字符）记录在表T中，并加入集合Q和WorkList。
- 在该算法停止时，每个qi ∈ Q都对应于DFA中的一个状态di ∈ D。
- 因为NFA的有效配置是有限的，每个配置在WorkList上只出现一次，所以while循环必定会停止。
- Q可能会很庞大，其中包含的配置数最大可达|2<sup>N</sup>|（N的所有子集的集合）个。但因子集构造法导致的状态集膨胀，并不影响DFA的运行时间。

子集构造法到此并未结束，我们需要从得到的Q和T中构造出D（DFA的状态集）、d<sub>0</sub>、D<sub>A</sub>和δ<sub>D</sub>，这个过程很简单：

- 每个qi ∈ Q都需要一个DFA状态di ∈ D来表示。
- NFA起始状态q0对应的d0即DFA的起始状态。
- 如果qi包含NFA的某个接受状态，则对应的di就是DFA的接受状态之一。
- 遵从qi到di的映射，从T构造出转移函数δ<sub>D</sub>。如T[qi, c]等于qj，即qi接受字符c后转移到qj，则δ<sub>D</sub>(di, c)为dj。

#### 不动点计算

子集构造法是不动点计算的一个例子，这是一种在计算机科学中经常出现的计算方法。这种计算的特点在于，对取自某个结构已知的域中的集族，重复应用一个单调函数。当计算到达某个状态时，如果进一步的迭代只能得出已有的结果，那么计算将终止，这相当于连续的迭代空间中遇到了一个“不动点”。

#### 离线计算ε-closure

这里给出一个ε-closure的离线算法，对NFA转移图中的每个状态n计算ε-closure({n})，该算法是不动点计算的一个例子

```
for each state n ∈ N
    E(n) = {n}

WorkList = N
while (WorkList != ∅)
    remove n from WorkList
    t = {n}
    for each p that n-ε->p ∈ δ
        t = t ∪ E(p)
    if t != E(n)
        E(n) = t
        WorkList = WorkList ∪ {m|m-ε->n ∈ δ}
```

将NFA转移图看作由结点和边构成的图：

- 该算法首先为每个结点创建一个集合E。对于状态n，E(n)包含当前对ε-closure({n})的近似。
- 最初，对每个结点n都将E(n)设置为{n}，并将每个状态都放置在WorkList上。
- while循环的每次迭代都从WorkList中删除一个状态n，并找到从n出发的所有ε转移，并将转移的目标结点添加到E(n)。如果计算改变了E(n)，那么它会将在ε转移路径上的前驱结点放置到WorkList上。（如果n在其前驱结点的ε-closure中，那么向E(n)添加结点时，这些结点同样会添加到前驱结点对应的集合E中）。
- 对WorkList使用位向量集合可以确保算法不会将同一结点的名字放入WorkList两次。

### 从DFA到最小DFA：Hopcroft算法

#### 最小化目的

从子集构造法产生的DFA可能有大量状态。虽然这不会增加扫描字符串所需的时间，但确实会增加识别器在内存中占用的空间。但在计算机上，内存访问的速度通常决定了计算的速度，更小的识别器可能更适合载入到处理器的高速缓存。

#### 算法

- 为了最小化DFA(D, Σ, δ, d<sub>0</sub>, D<sub>A</sub>)中的状态数目，需要一种技术来检测两个状态是否是等价的，即二者是否对任何输入字符串都产生同样的行为。
- 最小化算法根据DFA状态的行为来找到状态中的各个等价类，从这些等价类出发，我们可以构造一个最小DFA。
- 这个算法构造出所有DFA状态的一个集合划分P = {p<sub>1</sub>, p<sub>2</sub>, p<sub>3</sub>, ... p<sub>m</sub>}。这个划分根据DFA状态的行为对状态分组。同属于一个集合p<sub>s</sub>的两个DFA状态d<sub>i</sub>、d<sub>j</sub>，对所有相同的输入字符都具有同样的行为。即，如果<a href="http://www.codecogs.com/eqnedit.php?latex=d_{i}\overset{c}{\rightarrow}d_{x}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?d_{i}\overset{c}{\rightarrow}d_{x}" title="d_{i}\overset{c}{\rightarrow}d_{x}" /></a>，<a href="http://www.codecogs.com/eqnedit.php?latex=d_{j}\overset{c}{\rightarrow}d_{y}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?d_{j}\overset{c}{\rightarrow}d_{y}" title="d_{j}\overset{c}{\rightarrow}d_{y}" /></a>，而<a href="http://www.codecogs.com/eqnedit.php?latex=d_{i},&space;d_{j}\in&space;p_{s}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?d_{i},&space;d_{j}\in&space;p_{s}" title="d_{i}, d_{j}\in p_{s}" /></a>，那么<a href="http://www.codecogs.com/eqnedit.php?latex=d_{x}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?d_{x}" title="d_{x}" /></a>和<a href="http://www.codecogs.com/eqnedit.php?latex=d_{y}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?d_{y}" title="d_{y}" /></a>必定属于同一个集合<a href="http://www.codecogs.com/eqnedit.php?latex=p_{t}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?p_{t}" title="p_{t}" /></a>。

```
Hopcroft(D, Σ, δ, d0, DA):
    T = {DA, {D - DA}};
    P = ∅;
    while (P != T)
        P = T;
        T = ∅;
        for each set p ∈ P
            T = T ∪ Split(p);

Split(p):
    for each c ∈ Σ
        if c splits S into s1 and s2
            return {s1, s2};
    return S;
```

- 为最小化DFA，在行为等价性的约束下，应该使每个集合p<sub>s</sub> ∈ P尽可能大。该算法从一个初始的粗糙划分开始，初始划分包含两个集合，<a href="http://www.codecogs.com/eqnedit.php?latex=p_{0}&space;=&space;D_{A}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?p_{0}&space;=&space;D_{A}" title="p_{0} = D_{A}" /></a>和<a href="http://www.codecogs.com/eqnedit.php?latex=p_{1}&space;=&space;\{&space;D&space;-&space;D_{A}&space;\}" target="_blank"><img src="http://latex.codecogs.com/gif.latex?p_{1}&space;=&space;\{&space;D&space;-&space;D_{A}&space;\}" title="p_{1} = \{ D - D_{A} \}" /></a>。这种划分确保了在最终划分中不会有集合同时包含接受和非接受状态，因为算法从不合并两个划分。
- 该算法重复地考察每个p<sub>s</sub> ∈ P，来寻找p<sub>s</sub>中对某个输入字符串具有不同行为的状态，来改进初始划分（通过模拟给定状态响应单个字符的行为）。给定符号c ∈ Σ，c对于每个状态d<sub>i</sub> ∈ p<sub>s</sub>都必须产生同样的行为。如果不是这样，将围绕c来拆分p<sub>s</sub>。
- 为了构造新的DFA，我们必须分别构建一个状态来表示最终划分中的每个集合，并依据原来的DFA添加适当的转移，并指定初始和接受状态。
- 该算法是不动点计算的另一个例子。P是有限的，它最多可以包含|D|个元素。while循环拆分P中的集合，但从不合并它们。因而，|P|是单调增的。当某个迭代未能拆分P中的任何集合时，循环将停止。当DFA中的每个状态都由不同的行为时（即已经是最小DFA时），该算法的性能最差。

### 将DFA用做识别器

- 到现在为止，我们已经发展出相应的机制，可以根据一个RE构造出对应的DFA实现。要付诸实用，编译器的词法分析器必须能够识别源语言语法中出现的所有语法范畴。那么我们需要的是一个识别器，它能够处理该语言的微语法对应的所有RE。给定各种语法范畴对应的RE：r1, r2, r3,..., rk，我们可以构造一个对应于所有语法范畴的单一的RE，即(r1|r2|r3|...|rk)。
- 当编译器在某些输入上调用词法分析器时，词法分析器将逐个考察输入的字符，如果在耗尽输入时处于接受状态，那么将接受该字符串。词法分析器应该返回字符串文本及其语法范畴（或词类）。由于大多数实际的程序都班汉多余一个单词，我们需要对语言或识别器进行转换。
- 在语言层次上，我们可以要求每个单词都结束于某些容易识别的分隔符，如空格或制表符。但这要求分隔符环绕所有的运算符，如+、-、(、)和逗号。
- 在识别器层次上，我们可以改变DFA的实现及其对“接受”的定义。为找到与某个RE匹配的长单词，DFA应该一直运行下去，直至当前状态s对下一个输入字符没有转移可用为止。此时，实现必须判断它到底匹配哪个RE。会发生两种情况，第一种比较简单。如果s是一个接受状态，那么DFA已经发现了该语言中的一个单词，应该报告该单词及其语法范畴。
- 如果s不是接受状态，会出现两种情况。如果在DFA到达s的路径上穿越了一个或多个接受状态，那么识别器应该回转到最近一个接受状态。这种策略可以匹配输入字符串中的最长有效前缀。如果在到达当前状态的路径上，DFA并未经过接受状态，那么输入字符串的任何前缀都不是有效单词，识别器应该报告错误。
- 一个复杂的因素是，DFA中的一个接受状态可能表示了原来的NFA中的几个接受状态。例如，如果词法规格中既包含表示关键字的RE，也包含了表示标志符的RE，那么诸如new这样的关键字可能匹配两个RE。识别器必须判断返回哪个语法范畴：是标志符还是对应于关键字new的单元素范畴（singleton category）。

## 实现词法分析器

- 对于构建词法分析器的问题，形式语言的理论已经产生了相关工具，可以使词法分析器的实现自动化。编译器编写者可以直接从一组正则表达式生成一个速度可接受的词法分析器。编译器编写者可以为每个语法范畴建立一个RE，并将这些RE提供给词法分析器生成器作为输入。生成器会为每一个RE构建一个NFA，并用ε转移将各个NFA合并起来，从而创建一个对应的DFA，最后最小化DFA。
- 到这里，词法分析生成器必须将DFA转换为可执行代码。
- 这里讨论将DFA转换为可执行代码的三种实现策略：表驱动词法分析器、直接编码词法分析器和手工编码的词法分析器。所有这些词法分析器都通过模拟DFA的方式运转。
- 如果读入输入后的当前状态s是接受状态，词法分析器识别单词并向调用过程返回一个词素及其语法范畴。如果s不是接受状态，词法分析器必须判断在通往s的路径上，它是否遇到过接受状态。如果词法分析器确实遇到过接受状态，它应该将其内部状态和输入流都回滚到该点，并报告成功。如果没有遇到接受状态，它应该报告失败。
- 三种实现策略都具有相同的渐进复杂度，即处理每个字符的成本为常数，外加回滚的成本。效率差别仅在于处理每个字符的常量成本。

### 表驱动词法分析器

- 表驱动的方法使用一个框架词法分析器用于控制，使用一组生成的表来编码特定于语言的知识。编译器编写者提供一组以正则表达式指定的词法模式。词法分析器生成器接下来用于驱动框架词法分析器的表。
- 框架词法分析器分成四部分：初始化，模拟DFA行为对输入进行扫描的循环，在DFA越过标记末尾时用以处理的回滚循环，最后一部分解释并报告结果。扫描循环重复词法分析器的两个基本操作：读取一个字符并模拟DFA的操作。当DFA进入错误状态s<sub>e</sub>时，该循环停止。两个表CharCat和δ编码了关于DFA的所有知识。回滚循环使用一个状态栈，来将词法分析器恢复到最近的接受状态。
- 从字符到类别，然后从状态和类别到新状态的两部转换过程，使得词法分析器可以使用压缩的转移表。一个完整的表可以消除通过CharCat的间接映射，但将增加表的内存占用。未压缩的转移表随DFA中状态数和Σ中字符数的乘积而增长，随着转移表内存占用量的增大，它可能无法完全载入到CPU的高速缓存。
- 在压缩表和未压缩表的每字符成本之间的精确这种，同时取决于语言和运行词法分析器的计算机的性质。

```
NextWord():
    state = s0;
    lexeme = "";
    clear stack;
    push(bad);

    while (state != se)
        NextChar(char);
        lexeme = lexeme + char;
        if state ∈ SA
            clear stack;
        push(state);
        cat = CharCat[char];
        state = δ[state, cat];

    while (state ∉ SA and state != bad)
        state = pop();
        truncate lexeme;
        RollBack();

    if state ∈ SA
        return Type[state];
    else
        return invalid;
```

分类器表CharCat：

r  | 0, 1, 2,..., 9 | EOF | Other
-- | -------------- | --  | -----
Register | Digit | Other  | Other

转移表δ：

state | Register | Digit | Other
----- | -------- | ----- | -----
s0 | s1 | se | se
s1 | se | s2 | se
s2 | se | s2 | se
se | se | se | se

类型标记表Type：

s0 | s1 | s2 | s3
-- | -- | -- | --
invalid | invalid | register | invalid

对应的潜在DFA：

![对应的潜在DFA](/res/对应的潜在DFA.png)

- 框架词法分析器实用变量state来保存被模拟DFA的当前状态。它使用一个两步的查表过程来更新state。首先，它实用CharCat表将字符归类为若干类别之一。用于识别r[0...9]<sub>+</sub>的词法分析器有三个类别：Register、Digit、Other。接下来，它将当前状态和字符类别作为索引，来索引转移表δ。
- 该算法使用了一个宏NextChar，将其唯一参数设置为输入流中下一个字符。对应的宏RollBack，则将输入流向后移动一个字符。
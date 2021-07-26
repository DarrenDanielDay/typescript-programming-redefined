# TypeScript: Programming. Redefined.

TypeScript: 编程，已被重新定义。

> 本文假设你对JavaScript有一定的了解，并初步接触过TypeScript。
> 当然，并不需要是JavaScript的专家，但你需要理解JavaScript中的一些基础概念。
> 
> 本文还假设你至少有高中程度的数学能力。
> 当然，理解本文也不需要特别优秀的数学能力，但有更好的数学基础有助于更好地理解本文。

## 前言

本文的标题其实是vscode的宣传标语`Code Editing. Redefined.`的一个简单套用。

本文的主要内容为笔者对TypeScript语言设计的个人理解，带有一些主观性。

本文中对于一些概念的解释并非采用官方的解释，官方介绍请参考<https://www.typescriptlang.org/docs/handbook>



## 正文

### TypeScript: Typed JavaScript at Any Scale.

TypeScript本质上是带有类型的JavaScript。TypeScript相比JavaScript多出来的类型系统，是TypeScript的主要亮点。

TypeScript是JavaScript的超集。换言之，任何合法的JavaScript代码都是合法的TypeScript代码。
除去类型系统，TypeScript的代码和JavaScript的代码看起来应当是一样的。
但是，下面有一个说明TypeScript并不是JavaScript超集的“反例”：

```js
// JavaScript
console.log(new Map<1, 2>(null))
// Output: false true
```

```ts
// TypeScript
console.log(new Map<1, 2>(null))
// Output: Map(0) {}
```

TypeScript的泛型语法在某些场景下是“可以用于”JavaScript的，当然在JavaScript中被理解成了大于号和小于号。
这是一个相同代码语义不一致的狡猾的“反例”。当然，在大多数情况下我们也不会希望例子中的TypeScript代码具有例子中的JavaScript代码的语义。

从语法层面上来讲，TypeScript基本兼容所有的JavaScript语法，但语法分析的逻辑（文法）并不是包含关系，
如上面的反例，在JavaScript里分析成比较运算的，在TypeScript中可能就被分析为泛型了。

微软官方并没有为TypeScript实现自己的运行时。
目前TypeScript代码的运行需要先经过`tsc`（也可以是其他编译器，例如`babel`、`esbuild`）编译（转译）成JavaScript，并作为JavaScript运行在JavaScript的运行时上。
因此，TypeScript的本质是JavaScript。这一点非常重要。
凡是JavaScript做不到的执行逻辑，TypeScript一样会做不到。只要TypeScript还没有自己独立的运行时，这一点就不会改变。
所以，不要再搜索“用TypeScript如何实现xxx”了，你需要搜索的应当是“用JavaScript如何实现xxx”，你需要查阅的也是你使用的JavaScript运行环境所具有的API。

### Type System: 一种描述JavaScript对象和原始对象的集合

先上结论：TypeScript的类型语法本质是一种集合的描述语言，TypeScript的类型本质是一种特殊的集合。

说类型是**特殊**的集合，是因为这些集合具有一些与数学意义上的集合不同的性质。不符合数学意义上集合性质的集合，叫做`any`。

没错，这个TypeScript初学者最喜欢用来防止报错的逃生通道`any`类型，恰恰是TypeScript中最不合理的存在，后文会详细介绍。

尽管TypeScript官方教程中没有明确指出“TypeScript的类型就是一种集合”的概念，但我们从一些名词中可以看到集合的影子：
Union Type，联合类型，Union有并集的含义；Intersection Type，交叉类型，Intersection有交集的含义。

事实上仔细思考一下官方联合类型的[例子](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types)，你会察觉到联合类型似乎就是并集这么一回事：

```ts
function printId(id: number | string) {
  console.log("Your ID is: " + id);
}
// OK
printId(101);
// OK
printId("202");
// Error
printId({ myID: 22342 });
```

类型为`number | string`的参数既能够接受number，也能够接受string，该类型是两者的合并。

带着这样的大观点去看待TypeScript的类型系统，你会发现TypeScript的设计其实非常自然与优雅。

> 或许你注意到了一些TypeScript不同于JavaScript的语法：`id: number | string`。这后半部分的`:number | string`是TypeScript的类型标注，可以用于函数的参数/返回值与变量/常量声明的后面，以表示此处对应的值的类型。后文介绍的“类型”相关的语法，都可以在这些位置使用。

#### TypeScript类型集合的表示语法

现在，到你的数学知识发挥作用的时候了！

数学上的集合主要有三种表示方式：符号法、枚举法、描述法。

在TypeScript中，这三种表示方法也有对应的方法。

##### 符号法

符号法表示集合是最直接了当的表示方式。

数学上：例如**R**表示实数集，**Z**表示整数集，**Q**表示有理数集。

TypeScript类型：例如`string`表示字符串类型，`number`表示数字类型，`boolean`表示布尔类型。

在TypeScript中，用符号表示的简单类型包括以下：

符号|类型
|-|-|
`number`|数字类型
`string`|字符串类型
`true`|true的类型
`false`|false的类型
`boolean`|布尔类型，等价于`true \| false`
`undefined`|undefined的类型
`null`|null的类型
`symbol`|符号类型
`bigint`|大整数类型
`object`|除了以上类型以外的对象的类型
-|-
`unknown`|全集类型
`never`|空集类型
`any`|任意类型
`void`|无返回值函数返回类型

笔者将按分割线分成的两部分来介绍这些符号的具体含义。

###### 前半部分：JavaScript中的原始值与对象的描述

前半部分的类型中，除了`true`, `false`, `null`, `object`以外，其名称都与JavaScript中的typeof操作符作用于一个JavaScript值对象所得的字符串一致。
例如有以下代码：
```ts
let a: number = 1;
console.log(typeof a === "number");     // true
let b: string = "b";
console.log(typeof b === "string");     // true
let c: boolean = true;
console.log(typeof c === "boolean");    // true
let d: undefined = undefined;
console.log(typeof d === "undefined");  // true
let e: symbol = Symbol();
console.log(typeof e === "symbol");     // true
let f: bigint = 123n;
console.log(typeof f === "bigint");     // true
```

`true`、`false`、`null`虽然并不对应typeof得到的值，但也相对容易理解，不做过多解释。

值得注意的是`object`类型。`object`类型包括了除了以上所提到的类型的所有类型，包括JavaScript的对象（`typeof obj === "object" && obj !== null`）与**函数**（`typeof fn === "function"`）。

重要的事情说三遍！

函数属于`object`类型！

函数属于`object`类型！

函数属于`object`类型！

笔者认为这样设计的理由并不是TypeScript官方不想加入`function`类型，而是许多现有的JavaScript第三方库大量使用了给函数对象添加属性实现的API。

例如`glob`的API：

```ts
import glob from "glob";
console.log(typeof glob === "function");        // true
console.log(typeof glob.sync === "function");   // true
```

> 如果你想要运行以上代码查看结果，你可能需要一个[`nodejs`](https://nodejs.org/)的运行环境，并安装[`glob`](https://www.npmjs.com/package/glob)库。

`glob`导出的函数上面还有别的方法，而对于其他原始类型的值则不能这样做，这也是将函数对象视作一种`object`的理由。

由于函数的行为更加复杂，需要进行的校验也更多，TypeScript将函数类型作为一种独特的类型进行检查。后文将会介绍。

---

回过头来看，你会发现这些类型已经包括了所有的JavaScript对象！所有的JavaScript对象都可以用这些类型来描述！
整理一下思路，你应该能得出以下结论：JavaScript的具体原始值和对象，例如例子里的`1`，`"b"`，`true`，`123n`等等，都是这些类型集合中的元素。
 
类型与JavaScript原始值、JavaScript对象（这两者在TypeScript的概念中被笼统地称为“值”）的关系就像是集合与元素之间的关系。

值`1`的类型是`number`，即`1`属于集合`number`，用数学符号表示就是`1 ∈ number`。

值`"a"`的类型是`string`，即`"a"`属于集合`string`，用数学符号表示就是`"a" ∈ string`。

通过类比，你可能会想到这个看起来有点怪怪的结论：

值`undefined`的类型是`undefined`，即`undefined`属于集合`undefined`，用数学符号表示就是`undefined ∈ undefined`。


这里其实需要做一个概念上的区分。
左边的`undefined`是JavaScript中的`undefined`全局变量，是一个具体的JavaScript原始值；
右边的`undefined`则是指TypeScript的内置类型，是一个TypeScript类型。
两者并不是同一个概念，只是用了同一个单词表示，并不会产生罗素悖论。

###### 后半部分：特殊的抽象内置类型

正如前文的表格所介绍，`unknown`对应的集合是全集，`never`对应的集合是空集。

`unknown`是全集，所有的值都属于`unknown`集合，比较容易理解，套用前文的话，我们也能得到：

值`1`的类型是`unknown`，即`1`属于集合`unknown`，用数学符号表示就是`1 ∈ unknown`。

值`"a"`的类型是`unknown`，即`"a"`属于集合`unknown`，用数学符号表示就是`"a" ∈ unknown`。

> 此处应有一个小插曲：为什么一会儿说值`1`的类型是`number`，一会儿又是`unknown`？
> 其实这是类型所描述范围的精确性问题。
> 例如坐在屏幕前的你，可以说类型是“人类”，也可以说类型是“生物”，只不过“生物”这个类型是范围更大，概念更模糊的集合。
> `unknown`是最广泛的类型，它可能是任何具体的值，因此我们对它一无所知，这也是`unknown`这个名字的由来。
> 那么我们是否应该使用描述最精确的类型来描述一个值？这不一定，看具体需要的场景。后文将会提到，值`1`可以被比`number`更精确的`字面量类型`所描述。

---

`never`是空集，所有的值都不属于`never`集合。这使得never成为一个非常抽象的存在，因为我们找不到属于`never`类型的具体值。

那么`never`可以用在什么地方呢？官方的介绍给出了一个可以使用的地方：永远不可能返回值的函数的返回值类型。这个“永远不”正是`never`这个名字的由来。

请看下面的例子：

```ts
function throwsError(): never {
    throw new Error("This method never returns any value!");
}
const result1: never = throwsError();

function loopForever(): never {
    while (true) {}
}
const result2: never = loopForever();
```

这两个例子都是`never`类型最常见的使用场景。
`throwsError`总是会抛出异常，因此result1事实上接收不到任何值。
`loopForever`则是死循环，因此result2也接收不到任何值。

后文将会介绍`never`的其他用法：利用`never`作为空集性质的用法。

---

`any`则是一个不合理的微妙存在：从集合的角度来说，它既是全集，又是空集。

在代码中使用`any`声明近似等于告诉TypeScript的编译器：请关掉这一块相关代码的类型检查，我自愿放弃这一块代码的类型安全。

但是，即使是`any`类型的值，也不是空集的元素。具有`any`类型的值仍然不能赋值给`never`类型的变量。换言之，**`any`并不是万能的**。

---

`void`的设计，则是仿照了一些静态类型语言（C#）的设计。

void主要用于没有返回值的函数的返回值声明。
在JavaScript中，没有显式return值的函数，被调用后赋值给一个变量将会是`undefined`。
随意使用这些函数调用的结果是错误的，`void`类型正是为了避免这些错误。
`void`的本意是虚空，它像黑洞一样能够吸收所有的值，但外部可观测的部分是什么都没有（详细解释将会放在函数类型的解释里）。

```ts
function foo() {
    console.log("call foo");
}
let result: void = foo();
console.log(result)         // Output: undefined
```

##### 枚举法

枚举法，是“最笨”的表示集合的方式，但也最通俗易懂：集合中的元素被一一列举出来，仅包含有限的几个值。

数学上的枚举法：例如`{ 1, 2, 3 }`表示一个仅包含整数`1`，`2`，`3`的集合。

集合A在TypeScript中对应的语法大概会是这样：

```ts
// Example
let a: 1 | 2 | 3 = 1        // OK
let b: 1 | 2 | 3 = 2        // OK
let c: 1 | 2 | 3 = 3        // OK
let d: 1 | 2 | 3 = 0        // Error
let e: 1 | 2 | 3 = ""       // Error
```

可以看到，类型标注为`1 | 2 | 3`的变量只能被赋值成`1`，`2`，`3`的其中之一。

> 或许你会疑惑，为什么不用`type Foo = 1 | 2 | 3`之类的代码将后面大量重复的`1 | 2 | 3`简化一下？
> 本文意在给读者呈现一种更深入的对TypeScript的理解，因此所有的概念和语法都不能突然凭空出现——正像构建数学大厦一样，应当自底向上逐一介绍。
> `type`声明属于相对高阶的内容。阅读完后面的相关章节后，相信你会对`type`的认识会有所改观——因为它并不单纯是将类型作为一个名称那么简单。

事实上，`1 | 2 | 3`这段代码主要涉及到了两个TypeScript的概念：`字面量类型`与`联合类型`。

###### 字面量类型

前文有提到，JavaScript的整数值`1`，除了`number`以外，还有更精确的类型可以描述，那就是字面量类型`1`。

字面量类型和JavaScript的原始值类型表示方式完全一样，但他们是**类型**，不是**值**。

请看下面的代码实例：

```ts
let one: 1 = 1;
```

这里有两个`1`。前一个`1`是类型，后一个`1`则是值。前一个`1`用数学语言来表达就是`{ 1 }`，后一个`1`用数学语言来表达则是`1`。同样，这个赋值成立也可以表达为`1 ∈ { 1 }`。

其他原始类型对应的字面量类型，简单举几个例子：

```ts
let a: "aaa" = "aaa";
let b: true = true;
let c: false = false;
let d: 123.456 = 123.456;
let e: 1.23456789e10 = 1.23456789e10;
let f: 1234567890n = 1234567890n;
```

> 前文在符号法里提到了`true`与`false`。其实将其归为字面量类型也容易理解，因为布尔类型的字面量值只有`true`和`false`两种。`undefined`和`null`同理，但此处并未列出。

###### 联合类型

在前文中，其实笔者已经提出了联合类型是集合的并集这一要点。

结合前文字面量类型的理解，我们不难得出前文实例代码中的`1 | 2 | 3`类型用数学语言来表达就是`{ 1 } ∪ { 2 } ∪ { 3 }`，化简后就得到了`{ 1, 2, 3 }`。联合类型语法中的`|`和数学符号中的求并集符号`∪`所具有的含义基本一致。

---

总结一下以上两点：字面量类型能够精确地表示一个原始值，而联合类型操作符`|`可以将这些单个值的集合合并起来，从而形成更大的枚举集合。

除了字面量+联合类型的表示方法，TypeScript还有专门为枚举而生的类型——枚举类型`enum`。

---

###### 枚举类型

枚举类型的意义主要在于语义化常量，便于代码重构。当你需要用一些数字或者字符串来表示一个有限的状态集合时，枚举类型会非常有用。

枚举类型根据使用的枚举值可再细分为两种：数字枚举和字符串枚举。

数字枚举的语法例子：

```ts
enum E {
    V1,     // 0，默认从0开始
    V2,     // 1，不给值就是上一个+1
    V3,     // 2
    V4 = 5, // 5，中间可以断开    
    V5,     // 6
    V6 = 1, // 1，可以多个名称对应同一个值
    V7      // 2，不给值就是上一个+1
}

console.log(E)
// { "0": "V1", "1": "V6", "2": "V7", "5": "V4", "6": "V5", "V1": 0, V2": 1, "V3": 2, "V4": 5, "V5": 6, "V6": 1, "V7": 2 } 
```

字符串枚举的例子：

```ts
enum ProgrammingLanguage {
    TypeScript = 'TypeScript',
    JavaScript = 'JavaScript',
    CSharp = 'C#',  // 值为字符串即可，不需要符合标识符的语法
}
console.log(ProgrammingLanguage.CSharp)
// C#
console.log(ProgrammingLanguage)
// { "TypeScript": "TypeScript", "JavaScript": "JavaScript", "CSharp": "C#" } 
```

需要注意的是，`enum`声明的枚举名既可以作为`类型`，也可以作为`值`。

作为类型就是所有的枚举值的集合。例如上面的`ProgrammingLanguage`作为类型相当于就是`ProgrammingLanguage.TypeScript | ProgrammingLanguage.JavaScript | ProgrammingLanguage.CSharp`

作为值，正如输出所示，其本质就是一个JavaScript Object的实例。
对于数字枚举，在这个Object中所有的枚举名（例中的`V1`~`V7`）都会映射到自己对应的枚举值（例中的0,1,2,3,5,6），同时枚举值也反过来映射到与其值相同的**最后一个定义的枚举名**上（`1`映射到了`V6`，`2`映射到了`V7`）。
对于字符串枚举，则只有单向的从枚举名到枚举值的映射。

此外，枚举类型与字面值类型并不重合（被认为类型不兼容）。不同名称的枚举中的枚举名，即使枚举值一样，也不被认为是类型兼容的。

```ts
enum PL {
    TS = 'TypeScript',
    JS = 'JavaScript',
    CS = 'C#'
}

// 沿用上文的ProgrammingLanguage
let p1: ProgrammingLanguage = ProgrammingLanguage.TypeScript    // OK
let p1: ProgrammingLanguage = PL.TS     // Error. 尽管枚举值都是字符串'TypeScript'，但类型不兼容
```

这样做的目的在于检查是否不小心使用了不同的枚举类型的枚举值。只有同一个枚举类型对象上的属性才是类型兼容的，使用其他的枚举对象的枚举值可能产生误传。

具体而言，假设你的程序中有一种表示账号状态的枚举，分别是未认证，已认证，已停用，并分别用`0`，`1`，`2`表示：

```ts
enum AccountStateEnum {
    Unauthorized = 0,
    Authorized = 1,
    Deactivated = 2,
}
```

假设你的程序里有一种表示账号权限的枚举，分别是游客用户，正式用户，管理员用户，并也用`0`，`1`，`2`表示：

```ts
enum AuthorityEnum {
    Tourist = 0,
    User = 1,
    Administrator = 2,
}
```

显然，使用`AuthorityEnum.Administrator`来判断一个账号是否已被停用是没有意义的，也是令人费解的，尽管在JavaScript中你可以得到同样的比较结果。

TypeScript的枚举类型可以从根源上避免这个问题：认为在不同`enum`中定义的枚举值是不可比较的。

```ts
// 沿用上文的AccountStateEnum与AuthorityEnum
let account = {
    state: AccountStateEnum.Deactivated,
    role: AuthorityEnum.Administrator
}
if (account.state === AuthorityEnum.Administrator) { // Error，不可以比较不同的枚举类型的枚举值
    console.log("用户已注销");
}
```

除此之外，还有一种特殊的枚举类型——常量枚举`const enum`。在默认情况下，常量枚举被编译成对应的常量，但TypeScript为常量枚举提供了同样的类型检查。例如以下的TypeScript代码：

```ts
const enum E {
    A = 0,
    B = 1,
}
console.log(E.A);
console.log(E.B);
```

经过`tsc`的编译可以得到以下JavaScript代码：

```js
console.log(0 /* A */);
console.log(1 /* B */);
```

换言之，你将无法获取枚举对象`E`。如果你需要枚举的映射对象，你可以开启`preserveConstEnums`选项，这样你将会得到这样的代码：

```js
var E;
(function (E) {
    E[E["A"] = 0] = "A";
    E[E["B"] = 1] = "B";
})(E || (E = {}));
console.log(0 /* A */);
console.log(1 /* B */);
```

##### 描述法

剩下的描述法就是TypeScript类型系统的重头戏了：描述JavaScript对象与函数的类型。

前文介绍的主要都是JavaScript原始值类型在TypeScript的类型系统中的类型表示方法，但JavaScript里使用最多的，还是对象和函数。

TypeScript对JavaScript对象的特征描述是**结构性**的。尽管这些描述的许多概念都和面向对象编程的思想有着高度的一致性，但TypeScript的描述注重于**对象的结构**，面向对象的描述则注重于**继承树**。在TypeScript中只要结构一致，即使没有继承/实现关系，类型也是兼容的。这样设计的理由是JavaScript本身并不具有静态的类型系统，从语义上来说，所有的属性的访问与方法调用都是动态的，并非可静态编译的。

在TypeScript中，描述法表示的类型集合根据被描述的值可分为对象类型和函数类型两大类。笔者将先介绍函数部分，因为对象类型涉及了函数类型的语法。

在正式开始介绍之前，先复习一下数学中的描述法表示集合的语法：

```
{ 代表元素的符号 | 代表元素满足的性质 }
```

例如大于2的整数集可以用描述法表示为`{ x | x ∈ Z 且 x > 2 }`。

###### 描述函数

函数类型的描述主要在于参数列表与返回值两部分。

下面的表格简单介绍了基本的语法与对应的语义。其中的`Reflect.apply、Reflect.construct`涉及了JavaScript标准库的API，可参考<https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect/apply>

例子|语义|备注
-|-|-
`() => string` |`{ fn \| fn() ∈ string }`|`string`可为任意类型
`(name: string) => number`|`{ fn \| 对于任意name ∈ string，有fn(name) ∈ number }`|`string`和`number`可为任意类型，位置参数`name: string`可以有任意多个，参数名不能重复
`(this: string) => number`|`{ fn \| 对于任意thisArg ∈ string，有Reflect.apply(fn, thisArg, []) ∈ number }`|`string`和`number`可为任意类型
`new (param: string) => object`|`{ fn \| 对于任意param ∈ string，有new fn(param) ∈ object }`|`string`和`object`可为任意类型
`(this: string, arg1: number, arg2: boolean, ...args: string[]) => number`|`{ fn \| 对于任意thisArg ∈ string，arg1 ∈ number，arg2 ∈ boolean，args ∈ string[]，有Reflect.apply(fn, thisArg, [arg1, arg2, ...args]) ∈ number }`|除了`string[]`处必须为数组类型外，其他类型可为任意类型

> 这里出现了一种特殊的类型：数组类型。其本质也是一种对象类型，但函数的语法必然涉及到，在此先对`string[]`做一个简单的解释：`string[]`类型集合里的值都是数组对象，并且这些数组对象里的item都是string类型。

其中有一个带有`new`的语法，是针对JavaScript的`new`操作符的设计产生的。如果你还不熟悉这些，可参考<https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new>

函数类型和其他类型一样，可以直接用于标注，并在相关的上下文产生类型推论：

```ts
let fn: (name: string) => number = (name) => {      // name隐式获得了string的类型推论，而不需要显式标注
    return parseInt(name[0]);                       // 返回值也有类型推论的约束
}
let cls: new (param: string) => object = class {    // ES6的class语法本质是定义一个函数，并且可以通过new操作符构造
  constructor(param: string) {                      // constructor的定义参数应当与构造签名的参数列表匹配兼容性
    console.log(param);
  }
};
```


除此之外，函数类型的语法中还有`this context`、泛型、类型守卫等更高级的机制。这些语法将在更后面的部分介绍。

###### 描述对象

描述对象结构具有以下的语法：

1. 属性
2. 方法
3. 索引签名
4. 调用签名
5. 构造签名

这些语法都用于描述对象所具有的性质，并可以任意组合（放在同一个花括号内，通过换行符或者`;`、`,`分割），组合时为**且**的关系。

下面的表简单介绍了语法与对应的语义：

语法|类别|语义|备注
-|-|-|-
`{ name: string }`|属性|`{ obj \| obj.name ∈ string }`|`string`可为任意类型，`name`可为任意合法JavaScript标识符
`{ name?: string }`|属性|`{ obj \| obj.name ∈ string ∪ undefined }` | 同上
`{ readonly name: string }`|属性|`{ obj \| obj.name ∈ string，且不可修改obj.name }` | 同上
`{ "my-name": string }`|属性|`{ obj \| obj["my-name"] ∈ string }`|`string`可为任意类型，`"my-name"`可为任意字符串
`{ [Symbol.iterator]: string }`|属性|`{ obj \| obj[Symbol.iterator] ∈ string }`|`string`可为任意类型，`Symbol.iterator`可为任意类型为`symbol`的变量、属性表达式（例如`foo.bar.baz`）或字符串字面量
`{ name(): void }`|方法|`{ obj \| obj.name ∈ () => void }`|`name`的位置语法同上述的属性名，但不能被`readonly`修饰，`(): void`对应函数类型的语法`() => void`
`{ [name: string]: number }`|索引签名|`{ obj \| 对于任意name ∈ string，有obj[name] ∈ number ∪ undefined }`|`name`可为任意合法JavaScript标识符且不影响语义，`string`处必须是`string`或者`number`，`number`处可为任意类型
`{ (): void }`|调用签名|`{ obj \| obj ∈ () => void }`| 不能被`?`修饰，参数列表和返回值语法同函数类型
`{ new(count: number): string }`|构造签名|`{ obj \| (count) => new obj(count) ∈ (count: number) => string }`| 不能被`readonly`和`?`修饰，类似于函数类型

同类别的语法相似的部分可以进行组合，例如`{ readonly [Symbol.iterator]?: string }`，读者可以自行探索，这里不做过多解释。

几个实例代码：

```ts
let a: { name: string; age: number, } = { name: "yukari", age: 17 };
let b: { name: string, age?: number; } = { name: "furandoru" };
let c: { readonly size: number } = new Set();
c.size = 1; // Error 因为size是只读的
let d: { "my-name": string } = { "my-name": "marisa" };
let e: { [Symbol.toPrimitive]: string } = { [Symbol.toPrimitive]: "e to string" }
let f: { square(n: number): number } = { square(n) { return n * n } }
let g: {(): void} = function() {
    console.log("call g()");
}
let h: { new(name: string): object } = class { constructor(name) { console.log(name); } };
```

你可能会注意到一个细节：我们给一个具有对象类型标注的变量赋值的时候不能添加类型标注里没有声明的属性，像这样：

```ts
let a: { name: string } = {
    name: "Mike",
    age: 18,  // Error
}
```

报错信息如下：

```txt
Type '{ name: string; age: number; }' is not assignable to type '{ name: string; }'.
  Object literal may only specify known properties, and 'age' does not exist in type '{ name: string; }'.(2322)
```

这个报错并不是因为类型不兼容错了，因为`{ name: "Mike", age: 18 } ∈ { name: string; }`是成立的。如果你加入一个中间变量，这个报错将消失：

```ts
let tmp = {
    name: "Mike",
    age: 18,
};
let a: { name: string } = temp; // OK
```

前一个例子报错是因为对象字面量（`object literal`）中只能指定类型上已知的属性，而`age`不是类型`{ name: string; }`的已知属性。

这样设计是为了**防止拼写错误**。假设你有这样的代码：

```ts
let foo: { bar?: string } = { baz: "value" };
//         ^^^                ^^^ 
```

两个属性的拼写不一致，但类型标注中的`bar`属性是可选的，这样的代码类型上是正确的，但代码编写者的本意可能是设置`bar`属性，但由于疏忽错写成了`baz`。这个机制有助于发现此类拼写错误的问题：`baz`不是类型`{ bar?: string }`中声明的已知属性。

如果你不想要这个检查，你可以通过编译器选项`suppressExcessPropertyErrors`关闭这个检查。

#### TypeScript类型集合的运算

前文已经提到了部分类型运算：联合类型正是类型集合的并集运算。下面也通过一个表进行简单的总结（大写字母一般都代指类型，未备注的都可为任意类型）：

名称|语法|语义|备注
-|-|-|-
联合类型|`A \| B`|`{ obj \| obj ∈ A 或 obj ∈ B }`，即`A ∪ B`|
交叉类型|`A & B`|`{ obj \| obj ∈ A 且 obj ∈ B }`，即`A ∩ B`|
`keyof`操作符|`keyof A`|A的所有属性名/方法名（包括可选）的集合|`A`可为任何类型，开启编译器选项`keyofStringsOnly`后只包括字符串属性名，结果将不包含`number`和`symbol`
`typeof`操作符|`typeof obj`|`obj`在当前上下文内的类型|所有变量/属性在一个特定上下文内都有确定的类型，`typeof`操作符可用于获取这个类型。`obj`可以是一个变量名或属性表达式
索引类型|`A[B]`|`{ value \| 存在 obj ∈ A 和 key ∈ B，使得value ∈ typeof obj[key] }`|`A`可为任何类型，`B`须满足`B ⊆ keyof A`，如果强行忽略错误，结果将会是`any`
条件类型|`A extends B ? C : D`|`若A ⊆ B`，则结果为`C`，否则为`D`|`B`中可以使用`infer`子句，并且`C`中可以使用这些`infer`子句产生的新类型名
条件类型|`A extends B ? C : D`|`若A ⊆ B`，则结果为`C`，否则为`D`|B须满足`B ⊆ keyof A`，如果强行忽略错误结果将会是`any`
映射类型|`{ [K in Keys]: T }`|`{ obj \| 对任意K ∈ Keys，obj[K] ∈ T }`|`Keys`须满足`Keys ⊆ string \| number \| symbol`，`T`中可以使用`K`的类型表达式
映射类型|`{ [K in Keys as Renamed]: T }`|`{ obj \| 对任意K ∈ Renamed，obj[K] ∈ T }`|`Renamed`中可以使用`K`的类型表达式，且须满足`Renamed ⊆ string \| number \| symbol`，`T`中可以使用`K`和`Renamed`的类型表达式

#### interface与type的真正区别

先上结论：`interface`是`数据结构`，而`type`是算法。

#### TypeScript中的class

`class`是TypeScript的一个大坑——因为声明一个class既声明了一个函数，又声明了一种类型。如果没有在一开始就明确地知道类型与值的区别，很容易搞混概念。

### ESNext: ECMAScript标准最新草案实验性特性的降级编译

## 结语

总结起来，TypeScript的设计核心思想有以下几点：

1. 类型是值的集合，类型和值在概念上不重合。
2. 无限集合的表示，可以通过特殊符号，或用描述其元素特征的方式（描述法）来定义。
3. 集合的运算能生成新的集合——类型运算能生成类型，进行类型推导。
4. 静态检查运行类型运算代码，运行时无类型约束——开发时严格的静态属性静态类型提升开发体验+运行时灵活的动态属性鸭子类型提升代码能实现的逻辑的广度=重新定义编程。
5. 需要编译成JavaScript——编译除了擦除类型之外可以顺便做一些其他语法草案的降级转译，面向未来。
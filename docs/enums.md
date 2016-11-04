* [枚举](#enums)
* [枚举和数值](#enums-and-numbers)
* [枚举和字符串](#enums-and-strings)
* [改变数值和枚举的关联](#changing-the-number-associated-with-an-enum)
* [枚举是开放的](#enums-are-open-ended)
* [作为标识的枚举](#enums-as-flags)
* [常量枚举](#const-enums)
* [枚举静态函数](#enum-with-static-functions)

### 枚举{#enums}
枚举是一种组织相关值集合的方法。很多其他编程语言（C/C#/Java）都有 `enum` 数据类型，但是 JavaScript 没有。然而 TypeScript 是有的。这是一个 TypeScript 枚举的示例定义：

```ts
enum CardSuit {
	Clubs,
	Diamonds,
	Hearts,
	Spades
}

// 示例用法
var card = CardSuit.Clubs;

// 安全
card = "not a member of card suit"; // Error : string is not assignable to type `CardSuit`
```

#### 枚举和数值{#enums-and-numbers}
TypeScript 是基于数值的。这意味着数值可以被赋值到一个枚举的实例，而因此其他东西也可以兼容 `number`。

```ts
enum Color {
    Red,
    Green,
    Blue
}
var col = Color.Red;
col = 0; // 跟 Color.Red 一致
```

#### 枚举和字符串{#enums-and-strings}
:在我们深入了解枚举之前，让我们先看看它生成的 JavaScript，这里是示例的 TypeScript：

```ts
enum Tristate {
    False,
    True,
    Unknown
}
```
生成下面的 JavaScript

```js
var Tristate;
(function (Tristate) {
    Tristate[Tristate["False"] = 0] = "False";
    Tristate[Tristate["True"] = 1] = "True";
    Tristate[Tristate["Unknown"] = 2] = "Unknown";
})(Tristate || (Tristate = {}));
```

让我们关注这行 `Tristate[Tristate["False"] = 0] = "False";`。 其中`Tristate["False"] = 0` 应该自我解释，即设置 `Tristate` 变量的 `"False"` 成员为 `0`。需要注意的是 JavaScript 里赋值操作返回的是被赋的值（在这个例子中是 `0`）。因而被 JavaScript 运行时执行的下一个东西是 `Tristate[0] = "False"`。这意味着你可以使用 `Tristate` 变量来从枚举的字符串版本转换到一个数字或者从数字版本到字符串。如下所示：

```ts
enum Tristate {
    False,
    True,
    Unknown
}
console.log(Tristate[0]); // "False"
console.log(Tristate["False"]); // 0
console.log(Tristate[Tristate.False]); // "False" 因为 `Tristate.False == 0`
```

#### 改变数值和枚举的关联{#changing-the-number-associated-with-an-enum}
默认枚举是基于 `0` 的，每一个随后的值自动加 1。如下所示：

```ts
enum Color {
    Red,     // 0
    Green,   // 1
    Blue     // 2
}
```
然而你可以通过特定赋值改变数值和任何枚举成员的关联。下面展示的是我们从 3 开始自动增加：

```
enum Color {
    DarkRed = 3,  // 3
    DarkGreen,    // 4
    DarkBlue      // 5
}
```

> 提示：我通常初始化第一个枚举为 ` = 1` 以对于枚举值做安全可信检查。

#### 枚举是开放的{#enums-are-open-ended}
再次展示枚举生成的 JavaScript：

```js
var Tristate;
(function (Tristate) {
    Tristate[Tristate["False"] = 0] = "False";
    Tristate[Tristate["True"] = 1] = "True";
    Tristate[Tristate["Unknown"] = 2] = "Unknown";
})(Tristate || (Tristate = {}));
```

我们已经解释了 `Tristate[Tristate["False"] = 0] = "False";` 的部分。现在来关注下周边代码 `(function (Tristate) { /*code here */ })(Tristate || (Tristate = {}));` 尤其是 `(Tristate || (Tristate = {}));` 这块。这本质上捕获了一个本地变量 `TriState`，它可能指向一个已经定义好的 `Tristate` 值或者初始化为一个新的空 `{}` 对象。

这意味着你可以跨多文件分裂（和扩展）一个枚举定义。例如下面的例子中我们把 `Color` 的定义分成了两块：

```ts
enum Color {
    Red,
    Green,
    Blue
}

enum Color {
    DarkRed = 3,
    DarkGreen,
    DarkBlue
}
```

需要注意的是你*应该*重初始化第一个成员（这里是 `DarkRed = 3`）在一个枚举的延续中来使生成的代码不跟之前的定义（即 `0`、`1`、...等值）冲突。如果你不总是这样做的话，TypeScript 会警告你（错误信息 `In an enum with multiple declarations, only one declaration can omit an initializer for its first enum element.`）。

#### 作为标识的枚举{#enums-as-flags}
一个卓越的用法是把枚举用作 `Flags`。标识允许你检查在一系列条件中的一个特定条件是否为真。考虑在下面的例子中我们有关于动物的一系列属性：

```ts
enum AnimalFlags {
    None           = 0,
    HasClaws       = 1 << 0,
    CanFly         = 1 << 1,
    EatsFish       = 1 << 2,
    Endangered     = 1 << 3
}
```
这里我们使用左移操作符来在一个特定位层次里移动 `1` 来获得按位不相交的数字 `0001`、`0010`、`0100` 和 `1000`（如果你好奇的话，十进制里是 `1`、`2`、`4`、`8`）。如下所示，按位操作符 `|`（或）/ `&`（与）/ `~`（非）是你在跟标识一起工作的时候的最好朋友：

```ts

enum AnimalFlags {
    None           = 0,
    HasClaws       = 1 << 0,
    CanFly         = 1 << 1,
}

function printAnimalAbilities(animal) {
    var animalFlags = animal.flags;
    if (animalFlags & AnimalFlags.HasClaws) {
        console.log('animal has claws');
    }
    if (animalFlags & AnimalFlags.CanFly) {
        console.log('animal can fly');
    }
    if (animalFlags == AnimalFlags.None) {
        console.log('nothing');
    }
}

var animal = { flags: AnimalFlags.None };
printAnimalAbilities(animal); // nothing
animal.flags |= AnimalFlags.HasClaws;
printAnimalAbilities(animal); // animal has claws
animal.flags &= ~AnimalFlags.HasClaws;
printAnimalAbilities(animal); // nothing
animal.flags |= AnimalFlags.HasClaws | AnimalFlags.CanFly;
printAnimalAbilities(animal); // animal has claws, animal can fly
```

这里：

* 我们使用了 `|=` 来添加标识
* `&=` 和 `~` 的组合来清除标识
* `|` 来合并标识

注意：你可以在枚举定义中通过合并标识来创建方便的缩写，例如下面的 `EndangeredFlyingClawedFishEating`。

```ts
enum AnimalFlags {
	None           = 0,
    HasClaws       = 1 << 0,
    CanFly         = 1 << 1,
    EatsFish       = 1 << 2,
    Endangered     = 1 << 3,

	EndangeredFlyingClawedFishEating = HasClaws | CanFly | EatsFish | Endangered,
}
```

#### 常量枚举{#const-enums}

假设你有类似于下面的枚举定义：

```ts
enum Tristate {
    False,
    True,
    Unknown
}

var lie = Tristate.False;
```
`var lie = Tristate.False` 行会被编译成 JavaScript `var lie = Tristate.False`（对的，输入和输出一样）。这意味着执行的时候，运行时会寻找 `Tristate` 以及 `Tristate.False`。为了提升性能，这里你可以把 `enum` 标记为 `const enum`。 如下所示

```ts
const enum Tristate {
    False,
    True,
    Unknown
}

var lie = Tristate.False;
```

生成的 JavaScript：
```js
var lie = 0;
```

即编译器：

1. *内联*了枚举的任何使用（`0` 替代了 `Tristate.False`）。
2. 因为使用被内联了，没有为枚举定义生成任何 JavaScript（在运行时没有 `Tristate` 变量）。

##### 常量枚举 preserveConstEnums
内联有着显而易见的性能益处。没有 `Tristate` 在运行时的事实是编译器帮助你不生成不在运行时实际运行的 JavaScript。然而你可能想要编译器仍然生成枚举定义的 JavaScript 版本，为了像是*数值转字符串*或者*字符串转数值*的东西。在这种场景下你可以使用编译器标识 `--preserveConstEnums`，而它会仍然生成 `var Tristate` 定义因此你可以在运行时手动使用 `Tristate["False"]` 或者 `Tristate[0]`，如果你想的话。这在任何情况下都不会影响内联。

### 枚举静态函数{#enum-with-static-functions}
你可以使用声明 `enum` + `namespace` 合起来给枚举添加静态函数。下面展示的例子中我们添加了一个静态成员 `isBusinessDay` 到枚举 `Weekday` 中：

```ts
enum Weekday {
	Monday,
	Tuesday,
	Wednesday,
	Thursday,
	Friday,
	Saturday,
	Sunday
}
namespace Weekday {
	export function isBusinessDay(day: Weekday) {
		switch (day) {
			case Weekday.Saturday:
			case Weekday.Sunday:
				return false;
			default:
				return true;
		}
	}
}

const mon = Weekday.Monday;
const sun = Weekday.Sunday;
console.log(Weekday.isBusinessDay(mon)); // true
console.log(Weekday.isBusinessDay(sun)); // false
```

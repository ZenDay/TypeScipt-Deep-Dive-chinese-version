### 迭代器

迭代器本身不是一个 TypeScript 或者 ES6 特性，它是一个面向对象编程语言中的一个普遍的行为设计模式。它通常就是，一个实现了下面接口的对象：

```ts
interface Iterator<T> {
    next(value?: any): IteratorResult<T>;
    return?(value?: any): IteratorResult<T>;
    throw?(e?: any): IteratorResult<T>;
}
```

这个接口允许去从属于对象的一些集合或者队列中获取值。

想象一下有一个一些帧的对象，包括了这个帧包含的组件列表。使用迭代器接口可以像下面这样从帧对象中取出组件：

```ts
'use strict';

class Component {
  constructor (public name: string) {}
}

class Frame implements Iterator<Component> {

  private pointer = 0;

  constructor(public name: string, public components: Component[]) {}

  public next(): IteratorResult<Component> {
    if (this.pointer < this.components.length) {
      return {
        done: false,
        value: this.components[this.pointer++]
      }
    } else {
      return {
        done: true
      }
    }
  }

}

let frame = new Frame("Door", [new Component("top"), new Component("bottom"), new Component("left"), new Component("right")]);
let iteratorResult1 = frame.next(); //{ done: false, value: Component { name: 'top' } }
let iteratorResult2 = frame.next(); //{ done: false, value: Component { name: 'bottom' } }
let iteratorResult3 = frame.next(); //{ done: false, value: Component { name: 'left' } }
let iteratorResult4 = frame.next(); //{ done: false, value: Component { name: 'right' } }
let iteratorResult5 = frame.next(); //{ done: true }

//可以通过 value 属性来访问迭代器结果的值：
let component = iteratorResult1.value; //Component { name: 'top' }
```
再次强调，迭代器本身不是 TypeScript 特性，这些代码在没有实际实现 Iterator 和 IteratorResult 接口的情况下是无效的。然而使用这些普遍的 ES6 [接口](./types/ambient/interfaces.md)对于代码一致性来说很有用。

Ok，非常好，但是可以更有用。ES6 定义了*迭代协议*，其中包括了 [Symbol.iterator] `symbol`，如果 Iterable 接口已经实现的话：

```ts
//...
class Frame implements Iterable<Component> {

  constructor(public name: string, public components: Component[]) {}

  [Symbol.iterator]() {

    let pointer = 0;
    let components = this.components;

    return {

      next(): IteratorResult<Component> {
        if (pointer < components.length) {
          return {
            done: false,
            value: components[pointer++]
          }
        } else {
          return {
            done: true
          }
        }
      }

    }

  }

}

let frame = new Frame("Door", [new Component("top"), new Component("bottom"), new Component("left"), new Component("right")]);
for (let cmp of frame) {
  console.log(cmp);
}
```

不幸运的是，`frame.next()` 在这种模式里没有作用，而且它看起来有些笨重。使用 IterableIterator 来补救！

```ts
//...
class Frame implements IterableIterator<Component> {

  private pointer = 0;

  constructor(public name: string, public components: Component[]) {}

  public next(): IteratorResult<Component> {
    if (this.pointer < this.components.length) {
      return {
        done: false,
        value: this.components[this.pointer++]
      }
    } else {
      return {
        done: true
      }
    }
  }

  [Symbol.iterator](): IterableIterator<Component> {
    return this;
  }

}
//...
```
通过 IterableIterator  接口，`frame.next()` 和 `for` 循环现在可以很好地起作用了。

迭代器不止可以迭代有限的值。典型的例子是斐波那契数列：

```ts
class Fib implements IterableIterator<number> {

  protected fn1 = 0;
  protected fn2 = 1;

  constructor(protected maxValue?: number) {}

  public next(): IteratorResult<number> {
    var current = this.fn1;
    this.fn1 = this.fn2;
    this.fn2 = current + this.fn1;
    if (this.maxValue && current <= this.maxValue) {
      return {
        done: false,
        value: current
      }
    } return {
      done: true
    }

  }

  [Symbol.iterator](): IterableIterator<number> {
    return this;
  }

}

let fib = new Fib();

fib.next() //{ done: false, value: 0 }
fib.next() //{ done: false, value: 1 }
fib.next() //{ done: false, value: 1 }
fib.next() //{ done: false, value: 2 }
fib.next() //{ done: false, value: 3 }
fib.next() //{ done: false, value: 5 }

let fibMax50 = new Fib(50);
console.log(Array.from(fibMax50)); // [ 0, 1, 1, 2, 3, 5, 8, 13, 21, 34 ]

let fibMax21 = new Fib(21);
for(let num of fibMax21) {
  console.log(num); //输出从 0 到 21 的斐波那契数列
}
```

#### 为 ES5 目标使用迭代器构件代码
上面的代码例子要求 ES6 为目标，然而它在 ES5 目标下也可以工作，如果目标 JS 引擎支持 `Symbol.iterator` 的话。这可以通过在 ES5 目标上使用 ES6 库（添加 es6.d.ts 到你的项目中）来使其编译。编译后的代码应该在 node 4+，Google Chrome 和一些其他引擎里有效。

#### `super`

注意到如果你在子类中调用 `super` 它会像下面那样重定向 `prototype`：

```ts
class Base {
    log() { console.log('hello world'); }
}

class Child extends Base {
    log() { super.log() };
}
```
生成：

```js
var Base = (function () {
    function Base() {
    }
    Base.prototype.log = function () { console.log('hello world'); };
    return Base;
})();
var Child = (function (_super) {
    __extends(Child, _super);
    function Child() {
        _super.apply(this, arguments);
    }
    Child.prototype.log = function () { _super.prototype.log.call(this); };
    return Child;
})(Base);

```
留意 `_super.prototype.log.call(this)`。

这意味着你不能在成员属性上使用 `super` 。取而代之的是你应该使用 `this`。

```ts
class Base {
    log = () => { console.log('hello world'); }
}

class Child extends Base {
    logWorld() { this.log() };
}
```

注意因为只有一个 `this` 在 `Base` 和 `Child` 间共享，你需要使用*不同的*名字（这里是 `log` 和 `logWorld`）。

同样需要注意的是如果你尝试去错误使用 `super`，TypeScript 会警告你：

```ts
module quz {
    class Base {
        log = () => { console.log('hello world'); }
    }

    class Child extends Base {
        // ERROR : only `public` and `protected` methods of base class are accessible via `super`
        logWorld() { super.log() };
    }
}
```

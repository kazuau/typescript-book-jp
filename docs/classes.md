### クラス(Classes)
JavaScriptにおいて、クラスを第一級(first class)のオブジェクトとして持つことが重要な理由は次の通りです:
1. [クラスが提供する有用な構造抽象化](./tips/classesAreUseful.md)
1. それぞれのフレームワーク(emberjs、reactjsなど)が独自に異なったバージョンのクラスを実装する代わりに、一貫したクラスを利用する方法を提供する
1. オブジェクト指向の開発者はすでにクラスを理解している

ようやくJavaScript開発者は`class`を使うことができます。ここではPointという初歩的なクラスがあります：
```ts
class Point {
    x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
    add(point: Point) {
        return new Point(this.x + point.x, this.y + point.y);
    }
}

var p1 = new Point(0, 10);
var p2 = new Point(10, 20);
var p3 = p1.add(p2); // {x:10,y:30}
```
このクラスは、ES5で動作する次のJavaScriptを生成します。
```ts
var Point = (function () {
    function Point(x, y) {
        this.x = x;
        this.y = y;
    }
    Point.prototype.add = function (point) {
        return new Point(this.x + point.x, this.y + point.y);
    };
    return Point;
})();
```
これは慣用的に使われてきた従来のJavaScriptのクラスのパターンであり、今は第一級(first classs)の言語構成要素としてのクラスです。

### 継承(Inheritance)
TypeScriptにおけるクラスは(他の言語のように)`extends`キーワードを使った*単一*継承をサポートします:

```ts
class Point3D extends Point {
    z: number;
    constructor(x: number, y: number, z: number) {
        super(x, y);
        this.z = z;
    }
    add(point: Point3D) {
        var point2D = super.add(point);
        return new Point3D(point2D.x, point2D.y, this.z + point.z);
    }
}
```
クラスにコンストラクタがある場合、コンストラクタから親コンストラクタを呼び出す必要があります(TypeScriptはこれを指摘します)。これにより、`this`にセットされるべきものが確実にセットされます。`super`を呼び出した後、コンストラクタで行いたい処理を追加できます(ここでは他のメンバ`z`を追加します)。

親のメンバ関数をオーバーライドする(ここでは`add`をオーバーライドします)場合でも、親クラスの機能を呼び出せることに注意してください(`super.`構文を使います)。

### 静的メンバ(Statics)
TypeScriptクラスは、クラスの全インスタンスで共有される`static`なプロパティをサポートします。静的メンバを配置する(アクセスする)自然な場所はクラスそのものであり、TypeScriptはそれを採用しています。

```ts
class Something {
    static instances = 0;
    constructor() {
        Something.instances++;
    }
}

var s1 = new Something();
var s2 = new Something();
console.log(Something.instances); // 2
```

静的メンバと同様に静的関数も使用できます。

### アクセス修飾子(Access Modifiers)
TypeScriptはアクセス修飾子として`public`、`private`、`protected`をサポートしています。これらは`class`メンバのアクセシビリティを次のように決定します：

|アクセス可能な場所(accessible on) | `public` | `protected` | `private`|
| ----------------- | ---------- | ------------- | ------ ----- |
|クラス(class)              |yes|yes|yes|
|子クラス(class children)    |yes|yes|no|
|クラスのインスタンス(class instances)|yes|no|no|

アクセス修飾子が指定されていない場合は、暗黙的に`public`となり、JavaScriptの*便利な*性質に一致します🌹。

アクセス修飾子は、ランタイム(生成されたJSの中)では何の影響もありませんが、TypeScriptで間違った使い方をするとコンパイルエラーが発生します。それぞれの例を以下に示します:

```ts
class FooBase {
    public x: number;
    private y: number;
    protected z: number;
}

// EFFECT ON INSTANCES
var foo = new FooBase();
foo.x; // okay
foo.y; // ERROR : private
foo.z; // ERROR : protected

// EFFECT ON CHILD CLASSES
class FooChild extends FooBase {
    constructor() {
      super();
        this.x; // okay
        this.y; // ERROR: private
        this.z; // okay
    }
}
```

いつも通り、これらの修飾子はメンバプロパティとメンバ関数の両方で利用可能です。

### Abstract修飾子
`abstract`はアクセス修飾子の1つと考えることができます。前述の修飾子とは異なり、クラスのメンバと同様に`class`にも適用できるので、別に説明します。`abstract`修飾子が主として意味することは、その機能を直接的に呼び出すことができず、子クラスがその機能を提供しなければならないということです。

* 抽象クラスを直接インスタンス化することはできません。その代わりに、利用者は`abstract class`を継承した`class`を作成しなければならない
* 抽象メンバは直接アクセスできず、子クラスがその機能を提供しなくてはならない

### コンストラクタは任意です

クラスはコンストラクタを持つ必要はありません。例えば、以下は正しく動作します。

```ts
class Foo {}
var foo = new Foo();
```

### コンストラクタを定義に使用する

以下のように、クラスのメンバを定義し、初期化します:

```ts
class Foo {
    x: number;
    constructor(x:number) {
        this.x = x;
    }
}
```
これはTypeScriptの省略形を使うことができる一般的なパターンです。メンバにアクセス修飾子を付けることができ、それが自動的にクラス内に宣言され、コンストラクタからコピーされます。なので、前の例は次のように書き直すことが可能です(`public x：number`に注目してください)：

```ts
class Foo {
    constructor(public x:number) {
    }
}
```

### プロパティ初期化子(Property initializer)
これはTypeScript(実際には、ES7から)でサポートされている気の利いた機能です。クラスのコンストラクタの外側でクラスのメンバを初期化できます。デフォルト値を指定するのに便利です(`members = []`に注目してください)。

```ts
class Foo {
    members = [];  // Initialize directly
    add(x) {
        this.members.push(x);
    }
}
```

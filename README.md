```
class Point {
  constructor(x) {
    super()
    this.b = x
  }

  toString() {
    // ...
  }

  toValue() {
    // ...
  }
}
```
```
Point.prototype = {
  constructor() {},
  toString() {},
  toValue() {},
}
```
①constructor 方法是类的构造函数，是一个默认方法，通过 new 命令创建对象实例时，自动调用该方法。一个类必须有 constructor 方法，如果没有显式定义，一个默认的 consructor 方法会被默认添加。所以即使你没有添加构造函数，也是会有一个默认的构造函数的。constructor 方法默认返回实例对象 this ，但是也可以指定 constructor 方法返回一个全新的对象，让返回的实例对象不是该类的实例。

②类的所有的方法都是定义在类的prototype属性上面

---

```
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}

var point = new Point(2, 3);
point.toString() // (2, 3)

point.hasOwnProperty('x') // true
point.hasOwnProperty('y') // true
point.hasOwnProperty('toString') // false
point.__proto__.hasOwnProperty('toString') // true
```
上面代码中，x和y都是实例对象point自身的属性（因为定义在this变量上），所以hasOwnProperty方法返回true，而toString是原型对象的属性（因为定义在Point类上），所以hasOwnProperty方法返回false。这些都与 ES5 的行为保持一致

---

```
class Foo {
  static classMethod() {
    return 'hello';
  }
}

Foo.classMethod() // 'hello'

var foo = new Foo();
foo.classMethod()  // TypeError: foo.classMethod is not a function
```
类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上static关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。

---

```
class Foo {
  static bar() {
    this.baz();
  }
  static baz() {
    console.log('hello');
  }
  baz() {
    console.log('world');
  }
}

Foo.bar() // hello
```


如果静态方法包含this关键字，这个this指的是类，而不是实例。
上面代码中，静态方法bar调用了this.baz，这里的this指的是Foo类，而不是Foo的实例，等同于调用Foo.baz。另外，从这个例子还可以看出，静态方法可以与非静态方法重名。

---

```
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
}

Bar.classMethod() // 'hello'
```
父类的静态方法，可以被子类继承。

---

```
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
  static classMethod() {
    return super.classMethod() + ', too';
  }
}

Bar.classMethod() // "hello, too"
```

静态方法也是可以从super对象上调用的。

---

```
class Point {
  constructor(x, y) {
  }
  toString() {}
}

class ColorPoint extends Point {
  constructor(x, y) {
    super(x, y) // 调用父类的constructor(x, y),返回自己的实例this
    this.x = x
    this.y = y
  }

  toString() {
    return super.toString(); // 调用父类的toString()
  }
}
```
子类必须在constructor方法中调用super方法，否则新建实例时会报错。

---

super这个关键字，既可以当作函数使用，也可以当作对象使用。
###函数使用
```
class A {}

class B extends A {
  constructor() {
    super();
  }
}
```
子类B的构造函数之中的super()，代表调用父类的构造函数。super虽然代表了父类A的构造函数，但是返回的是子类B的实例。因此super()在这里相当于A.prototype.constructor.call(this)。

---

### 对象使用
super作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。

###### 普通方法中
```
class A {
  constructor() {
    this.x = 1;
  }
  print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  m() {
    super.print();
  }
}

let b = new B();
b.m() // 2
```
ES6 规定，在子类普通方法中通过super调用父类的方法时，方法内部的this指向当前的子类实例。上面代码中，super.print()虽然调用的是A.prototype.print()，但是A.prototype.print()内部的this指向子类B的实例，导致输出的是2，而不是1。也就是说，实际上执行的是super.print.call(this)。


###### 静态方法中
```
class Parent {
  static myMethod(msg) {
    console.log('static', msg);
  }

  myMethod(msg) {
    console.log('instance', msg);
  }
}

class Child extends Parent {
  static myMethod(msg) {
    super.myMethod(msg);
  }

  myMethod(msg) {
    super.myMethod(msg);
  }
}

Child.myMethod(1); // static 1

var child = new Child();
child.myMethod(2); // instance 2
```
如果super作为对象，用在静态方法之中，这时super将指向父类，而不是父类的原型对象。
上面代码中，super在静态方法之中指向父类，在普通方法之中指向父类的原型对象。

---

```
class A {
  constructor() {
    this.x = 1;
  }
  static print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  static m() {
    super.print();
  }
}

B.x = 3;
B.m() // 3
```
在子类的静态方法中通过super调用父类的方法时，方法内部的this指向当前的子类，而不是子类的实例。
上面代码中，静态方法B.m里面，super.print指向父类的静态方法。这个方法里面的this指向的是B，而不是B的实例。




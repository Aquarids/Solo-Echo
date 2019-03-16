### js中什么是原型与原型链 [link](https://juejin.im/post/5a94c0de5188257a8929d837)

JavaScript中并没有引入类（class）的概念，但JavaScript仍然大量地使用了对象，为了保证对象之间的联系，JavaScript引入了原型与原型链的概念。与Java不同的是，JavaScript并非通过类而是直接通过构造函数来创建实例。

```js
function Dog(name, color) {
    this.name = name
    this.color = color
    this.bark = () => {
        console.log('wangwang~')
    }
}

const dog1 = new Dog('dog1', 'black')
const dog2 = new Dog('dog2', 'white')
```

在上面的代码中，有两个实例被创建，它们有自己的名字、颜色，但它们的bark方法是一样的，而通过构造函数创建实例的时候，每创建一个实例，都需要重新创建这个方法，再把它添加到新的实例中。这无疑造成了很大的浪费，既然实例的方法都是一样的，为什么不把这个方法单独放到一个地方，并让所有的实例都可以访问到呢。

这里就需要用到原型（prototype）
- 每一个构造函数都拥有一个prototype属性，这个属性指向一个对象，也就是原型对象。当使用这个构造函数创建实例的时候，prototype属性指向的原型对象就成为实例的原型对象。
- 原型对象默认拥有一个constructor属性，指向指向它的那个构造函数（也就是说构造函数和原型对象是互相指向的关系）
- 每个对象都拥有一个隐藏的属性[prototype]，指向它的原型对象，这个属性可以通过 Object.getPrototypeOf(obj) 或 obj.__proto__ 来访问。
- 在JavaScript中，所有的对象都是由它的原型对象继承而来，反之，所有的对象都可以作为原型对象存在。
- 访问对象的属性时，JavaScript会首先在对象自身的属性内查找，若没有找到，则会跳转到该对象的原型对象中查找。

```js
function Dog(name, color) {
    this.name = name
    this.color = color
}

Dog.prototype.bark = () => {
    console.log('wangwang~')
}

const dog1 = new Dog('dog1', 'black')
const dog2 = new Dog('dog2', 'white')
dog2.bark() = () => {
    console.log('miaomiaomiao???')
}
dog1.bark()  //'wangwang~'
dog2.bark()  //'miaomiaomiao???'
```

这里dog2重写bark方法并没有对dog1造成影响，因为它重写bark方法的操作实际上是为自己添加了一个新的方法使原型中的bark方法被覆盖了，而并非直接修改了原型中的方法。若想要修改原型中的方法，需要通过构造函数的prototype属性

JavaScript中所有的对象都是由它的原型对象继承而来。而原型对象自身也是一个对象，它也有自己的原型对象，这样层层上溯，就形成了一个类似链表的结构，这就是原型链（prototype chain）。所有原型链的终点都是Object函数的prototype属性，因为在JavaScript中的对象都默认由Object()构造。Objec.prototype指向的原型对象同样拥有原型，不过它的原型是null，而null则没有原型。通过原型链就可以在JavaScript中实现继承，JavaScript中的继承相当灵活，有多种继承的实现方法。

### js中的继承 [link](https://juejin.im/post/58f94c9bb123db411953691b)

原型链并非十分完美, 它包含如下两个问题：
- 当原型链中包含引用类型值的原型时,该引用类型值会被所有实例共享;
- 在创建子类型(例如创建Son的实例)时,不能向超类型(例如Father)的构造函数中传递参数.

> 经典继承：即在子类型构造函数的内部调用超类型构造函数。

```js
function Father(){
	this.colors = ["red","blue","green"];
}
function Son(){
	Father.call(this);//继承了Father,且向父类型传递参数
}
var instance1 = new Son();
instance1.colors.push("black");
console.log(instance1.colors);//"red,blue,green,black"

var instance2 = new Son();
console.log(instance2.colors);//"red,blue,green" 可见引用类型值是独立的
```
缺点：如果仅仅借用构造函数，那么将无法避免构造函数模式存在的问题--方法都在构造函数中定义，因此函数复用也就不可用了。而且超类型(如Father)中定义的方法，对子类型而言也是不可见的。

> 组合继承：将原型链和借用构造函数的技术组合到一块,从而发挥两者之长的一种继承模式。

```js
function Father(name){
	this.name = name;
	this.colors = ["red","blue","green"];
}
Father.prototype.sayName = function(){
	alert(this.name);
};
function Son(name,age){
	Father.call(this,name);//继承实例属性，第一次调用Father()
	this.age = age;
}
Son.prototype = new Father();//继承父类方法,第二次调用Father()
Son.prototype.sayAge = function(){
	alert(this.age);
}
var instance1 = new Son("louis",5);
instance1.colors.push("black");
console.log(instance1.colors);//"red,blue,green,black"
instance1.sayName();//louis
instance1.sayAge();//5

var instance1 = new Son("zhai",10);
console.log(instance1.colors);//"red,blue,green"
instance1.sayName();//zhai
instance1.sayAge();//10
```
组合继承避免了原型链和借用构造函数的缺陷,融合了它们的优点,成为 JavaScript 中最常用的继承模式. 而且, instanceof 和 isPrototypeOf( )也能用于识别基于组合继承创建的对象.

缺点：调用了两次父类构造函数, 造成了不必要的消耗。

> 原型继承：在object()函数内部, 先创建一个临时性的构造函数, 然后将传入的对象作为这个构造函数的原型,最后返回了这个临时类型的一个新实例。

```js
function object(o) {
	function F(){}
	F.prototype = o;
	return new F();
}
```
在 ECMAScript5 中,通过新增 object.create() 方法规范化了上面的原型式继承
```js
var person = {
	friends : ["Van","Louis","Nick"]
};
var anotherPerson = Object.create(person);
anotherPerson.friends.push("Rob");
var yetAnotherPerson = Object.create(person);
yetAnotherPerson.friends.push("Style");
alert(person.friends);//"Van,Louis,Nick,Rob,Style"

```

### new 运算符实际执行了哪些操作

```js
var obj  = {};
obj.__proto__ = F.prototype;
F.call(obj);
```

第一行，我们创建了一个空对象obj;
第二行，我们将这个空对象的__proto__成员指向了F函数对象prototype成员对象;
第三行，我们将F函数对象的this指针替换成obj，然后再调用F函数.
我们可以这么理解: 以 new 操作符调用构造函数的时候，函数内部实际上发生以下变化：
- 创建一个空对象，并且 this 变量引用该对象，同时还继承了该函数的原型。
- 属性和方法被加入到 this 引用的对象中。
- 新创建的对象由 this 所引用，并且最后隐式的返回 this.

### 闭包 [link](https://www.zhihu.com/question/19554716)

JavaScript 变量可以是局部变量或全局变量。私有变量可以用到[闭包](http://www.runoob.com/js/js-function-closures.html)。

> 函数作为返回值
```js
function lazy_sum(arr) {
    var sum = function () {
        return arr.reduce(function (x, y) {
            return x + y;
        });
    }
    return sum;
}
var f = lazy_sum([1, 2, 3, 4, 5]); // function sum()
f(); // 15
```

> 全局变量
```js
var a = 4;
function myFunction() {
    return a * a;
}
```
全局变量的作用域是全局性的，即在整个JavaScript程序中，全局变量处处都在。

而在函数内部声明的变量，只在函数内部起作用。这些变量是局部变量，作用域是局部性的；函数的参数也是局部性的，只在函数内部起作用。

> 内嵌函数

所有函数都能访问全局变量。  实际上，在 JavaScript 中，所有函数都能访问它们上一层的作用域。JavaScript 支持嵌套函数。嵌套函数可以访问上一层的函数变量。

```js
function add() {
    var counter = 0;
    function plus() {counter += 1;}
    plus();    
    return counter; 
}
```

> 函数私有变量
```js
var add = (function () {
    var counter = 0;
    return function () {return counter += 1;}
})();
 
add();
add();
add();
 
// 计数器为 3
```
变量 add 指定了函数自我调用的返回字值。自我调用函数只执行一次。设置计数器为 0。并返回函数表达式。add变量可以作为一个函数使用。非常棒的部分是它可以访问函数上一层作用域的计数器。这个叫作 JavaScript 闭包。它使得函数拥有私有变量变成可能。计数器受匿名函数的作用域保护，只能通过 add 方法修改。

### js中的this(上下文) [link](https://www.jianshu.com/p/fcbc21a2c507)

this总是指向调用该函数的对象  [箭头函数中不是](https://juejin.im/post/59aa71d56fb9a0248d24fae3)

### js中的call、apply、bind方法 [link](https://www.cnblogs.com/coco1s/p/4833199.html)

在 javascript 中，call 和 apply 都是为了改变某个函数运行时的上下文（context）而存在的，换句话说，就是为了改变函数体内部 this 的指向。JavaScript 的一大特点是，函数存在「定义时上下文」和「运行时上下文」以及「上下文是可以改变的」这样的概念。

```js
function fruits() {}
 
fruits.prototype = {
    color: "red",
    say: function() {
        console.log("My color is " + this.color);
    }
}
 
var apple = new fruits;
apple.say();    //My color is red
banana = {
    color: "yellow"
}
apple.say.call(banana);     //My color is yellow
apple.say.apply(banana);    //My color is yellow
```
call 和 apply 是为了动态改变 this 而出现的，当一个 object 没有某个方法（本栗子中banana没有say方法），但是其他的有（本栗子中apple有say方法），我们可以借助call或apply用其它对象的方法来操作。

对于 apply、call 二者而言，作用完全一样，只是接受参数的方式不太一样。

```js
var func = function(arg1, arg2) {};
func.call(this, arg1, arg2);
func.apply(this, [arg1, arg2])
```
JavaScript 中，某个函数的参数数量是不固定的，因此要说适用条件的话，当你的参数是明确知道数量时用 call。而不确定的时候用 apply，然后把参数 push 进数组传递进去。当参数数量不确定时，函数内部也可以通过 arguments 这个伪数组来遍历所有的参数。

bind() 方法与 apply 和 call 很相似，也是可以改变函数体内 this 的指向。bind()方法会创建一个新函数，称为绑定函数，当调用这个绑定函数时，绑定函数会以创建它时传入 bind()方法的第一个参数作为 this，传入 bind() 方法的第二个以及以后的参数加上绑定函数运行时本身的参数按照顺序作为原函数的参数来调用原函数。

```js
var bar = function(){
console.log(this.x);
}
var foo = {
x:3
}
bar(); // undefined
var func = bar.bind(foo);
func(); // 3
```
这里我们创建了一个新的函数 func，当使用 bind() 创建一个绑定函数之后，它被执行的时候，它的 this 会被设置成 foo, 而不是像我们调用 bar() 时的全局作用域。

在Javascript中，多次 bind() 是无效的。更深层次的原因， bind() 的实现，相当于使用函数在内部包了一个 call / apply ，第二次 bind() 相当于再包住第一次 bind() ,故第二次以后的 bind 是无法生效的。

- apply 、 call 、bind 三者都是用来改变函数的this对象的指向的；
- apply 、 call 、bind 三者第一个参数都是this要指向的对象，也就是想指定的上下文；
- apply 、 call 、bind 三者都可以利用后续参数传参；
- bind 是返回对应函数，便于稍后调用；apply 、call 则是立即调用 。

### 截流函数 [link](https://segmentfault.com/a/1190000008768202)

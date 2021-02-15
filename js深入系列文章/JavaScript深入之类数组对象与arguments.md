# JavaScript深入之类数组对象与arguments

>JavaScript深入系列第十三篇，讲解类数组对象与对象之间的相似与差异以及arguments的注意要点

## 类数组对象

所谓的类数组对象:

>拥有一个 length 属性和若干索引属性的对象

举个例子：

```js
var array = ['name', 'age', 'sex'];

var arrayLike = {
    0: 'name',
    1: 'age',
    2: 'sex',
    length: 3
}
```

即便如此，为什么叫做类数组对象呢？

那让我们从读写、获取长度、遍历三个方面看看这两个对象

## 读写

```js
console.log(array[0]); // name
console.log(arrayLike[0]); // name

array[0] = 'new name';
arrayLike[0] = 'new name';
```

## 长度

```js
console.log(array.length); // 3
console.log(arrayLike.length); // 3
```

## 遍历

```js
for(var i = 0, len = array.length; i < len; i++) {
   ……
}
for(var i = 0, len = arrayLike.length; i < len; i++) {
    ……
}
```

是不是很像？

那类数组对象可以使用数组的方法吗？比如：

```js
arrayLike.push('4');
```

然而上述代码会报错: arrayLike.push is not a function

所以终归还是类数组呐……

## 调用数组方法

如果类数组就是任性的想用数组的方法怎么办呢？

既然无法直接调用，我们可以用Function.call间接调用：

```js
var arrayLike = {0: 'name', 1: 'age', 2: 'sex', length: 3 }

Array.prototype.join.call(arrayLike, '&'); // name&age&sex

Array.prototype.slice.call(arrayLike, 0); // ["name", "age", "sex"] 
// slice可以做到类数组转数组

Array.prototype.map.call(arrayLike, function(item){
    return item.toUpperCase();
}); 
// ["NAME", "AGE", "SEX"]
```

## 类数组转对象

在上面的例子中已经提到了一种类数组转数组的方法，再补充三个：

```js
var arrayLike = {0: 'name', 1: 'age', 2: 'sex', length: 3 }
// 1. slice
Array.prototype.slice.call(arrayLike); // ["name", "age", "sex"] 
// 2. splice
Array.prototype.splice.call(arrayLike, 0); // ["name", "age", "sex"] 
// 3. ES6 Array.from
Array.from(arrayLike); // ["name", "age", "sex"] 
// 4. apply
Array.prototype.concat.apply([], arguments)
```

那么为什么会讲到类数组对象呢？以及类数组有什么应用吗？

要说到类数组对象，Arguments对象就是一个类数组对象。在客户端JavaScript中，一些DOM方法(document.getElementsByTagName())也返回类数组对象。

## Arguments对象

接下来重点讲讲Arguments对象。

Arguments对象只定义在函数体内，包括了函数的参数和其他属性。

举个例子：

```js
function foo(name, age, sex) {
    console.log(arguments);
}

foo('name', 'age', 'sex')
```

打印结果如下：

![arguments](https://github.com/mqyqingfeng/Blog/raw/master/Images/arguments.png)

我们可以看到除了类数组的索引属性和length属性之外，还有一个callee属性，接下来我们一个一个介绍。

## length属性

Arguments对象的length属性，表示实参的长度，举个例子：

```js
function foo(b, c, d){
    console.log("实参的长度为：" + arguments.length)
}

console.log("形参的长度为：" + foo.length)

foo(1)

// 形参的长度为：3
// 实参的长度为：1
```

## callee属性

Arguments对象的callee属性，通过它可以调用函数自身。

讲个闭包经典面试题使用callee的解决方法：

```js
var data = [];

for (var i = 0; i < 3; i++) {
    (data[i] = function () {
       console.log(arguments.callee.i) 
    }).i = i;
}

data[0]();
data[1]();
data[2]();

// 0
// 1
// 2
```

接下来讲讲arguments对象的几个注意要点：

## arguments和对应参数的绑定

```js
function foo(name, age, sex, hobbit) {

    console.log(name, arguments[0]); // name name

    // 改变形参
    name = 'new name';

    console.log(name, arguments[0]); // new name new name

    // 改变arguments
    arguments[1] = 'new age';

    console.log(age, arguments[1]); // new age new age

    // 测试未传入的是否会绑定
    console.log(sex); // undefined

    sex = 'new sex';

    console.log(sex, arguments[2]); // new sex undefined

    arguments[3] = 'new hobbit';

    console.log(hobbit, arguments[3]); // undefined new hobbit

}

foo('name', 'age')
```

传入的参数，实参和arguments的值会共享，当没有传入时，实参与arguments值不会共享

除此之外，以上是在非严格模式下，如果是在严格模式下，实参和arguments是不会共享的。

## 传递参数

将参数从一个函数传递到另一个函数

```js
function foo() {
    bar.apply(this, arguments);
}
function bar(a, b, c) {
    // logic
}
```

## 强大的ES6

使用ES6的...运算符，我们可以轻松转成数组。

```js
function func(...arguments) {
    console.log(arguments); // [1, 2, 3]
}

func(1, 2, 3);
```

## 应用

arguments的应用其实很多，在下个系列，也就是JavaScript专题系列中，我们会在jQuery的extend实现、函数柯里化、递归等场景看见arguments的身影。这篇文章就不具体展开了。

如果要总结这些场景的话，暂时能想到的包括：

1. 参数不定长
2. 函数柯里化
3. 递归调用
4. 函数重载
  ...

欢迎留言回复。


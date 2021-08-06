### 变量

###### 原始值和引用值
<font size=2 color=green>ECMAScript 变量可以包含两种不同类型的数据：原始值和引用值</font>
原始值：最简单的数据（Undefined、Null、Boolean、Number、String、Symbol）
引用值：保存在内存中的对象。操作对象时，实际上操作的是该对象的引用

<font size=2 color=green>ECMAScript函数的对象传参是按值传递的</font>
```
function setName(obj) { 
 obj.name = "James"; 
 obj = new Object(); 
 obj.name = "Koby"; 
} 
let person = new Object(); 
setName(person); 
console.log(person.name); // "James"
```
这里我们可以看到最后输出的"James",如果 person 是按引用传递的，那么person 应该自动将指针改为指向 name 为"Koby"的对象。<font size=2 color=green>说明函数中参数的值改变之后，原始的引用仍然没变。当 obj 在函数内部被重写时，它变成了一个指向本地对象的指针。而那个本地对象在函数执行结束时就被销毁了。</font>
###### 确定类型
<font size=2 color=green>typeof</font> 最适合判断一个变量是否是字符串、数值、布尔值或者undefined，对于对象或null时，<font size=2 color=green>typeof</font> 返回“object”。
```
console.log(typeof '123'); // string 
console.log(typeof 23); // number 
console.log(typeof true); // boolean 
console.log(typeof undefined); // undefined 
console.log(typeof null); // object 
console.log(typeof new Object()); // object
```
如果需要判断对象是什么类型时，我们需要用到 <font size=2 color=green>instanceof</font> 操作符，其结果取决于对象的原型链
```
console.log(person instanceof Object); // 变量 person 是 Object 吗？
console.log(colors instanceof Array); // 变量 colors 是 Array 吗？
console.log(pattern instanceof RegExp); // 变量 pattern 是 RegExp 吗？
```

### 作用域
<font size=2 color=green>变量或函数的上下文决定了它们可以访问哪些数据，以及它们的行为</font>
全局上下文是最外层的上下文。每个函数调用都有自己的上下文。

当代码执行流进入函数时，函数的上下文被推到一个上下文栈上。在函数执行完之后，上下文栈会弹出该函数上下文，将控制权返还给之前的执行上下文。ECMAScript程序的执行流就是通过这个上下文栈进行控制的。

上下文中的代码在执行的时候，会创建变量对象的一个<font size=2 color=green>作用域链（scope chain）</font>。这个作用域链决定了各级上下文中的代码在访问变量和函数时的顺序。代码正在执行的上下文的变量对象始终位于作用域链的最前端。如果上下文是函数，则其活动对象（activation object）用作变量对象。活动对象最初只有一个定义变量：arguments。（全局上下文中没有这个变量。）作用域链中的下一个变量对象来自含上下文，再下一个对象来自再下一个包含上下文。以此类推直至全局上下文；全局上下文的变量对象始终是作用域链的最后一个变量对象。

上下文在其所有代码都执行完毕后会被销毁，包括定义在它上面的所有变量和函数（全局上下文在应用程序退出前才会被销毁，比如关闭网页或退出浏览器）

>作用域链增强：这两种情况下，会在作用域链前端添加一个变量对象
> + try/catch 语句的 catch 块
> + with 语句
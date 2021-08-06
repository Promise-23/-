### 垃圾回收
JavaScript 通过自动内存管理实现内存分配和闲置资源回收
周期性：每隔一段时间，垃圾回收程序就会自动运行，确定哪个变量不会再使用，就会释放它占用的内存。

<font size=2 color=green>标记清理</font> 是主流的垃圾回收算法，其原理是在垃圾回收程序运行的时候，会标记内存中存储的所有变量，然后，它会将所有在上下文中的变量，以及被在上下文中的变量引用的变量的标记去掉。在此之后被加上标记的变量就是待删除的了。随后垃圾回收程序做一次内存清理，销毁带标记的所有值并收回他们的内存。
<font size=2 color=green>引用计数</font> 是另一种垃圾回收策略。其原理是对每个值都记录它被引用的次数。声明变量并且给它赋一个引用值时，这个值得引用数加1，如果保存对该值引用得变量被其他值覆盖了，那么引用数减1。垃圾回收程序下次运行得时候就会释放引用数为0的值的内存。<font size=2 color=red>引用计数算法可能导致出现循环引用的情形</font>

> 垃圾回收程序会周期性地运行，如果在内存中分配了很多变量，可能会造成性能损失，因此垃圾回收机制的时间调度尤其重要。

<font size=2 color=green>现代垃圾回收程序会基于对 JavaScript 运行时环境的探测来决定何时运行。探测机制因引擎而异，但基本上都是根据已分配对象的大小和数量来判断的。</font>

### 内存管理
优化内存占用的最佳手段就是保证在执行代码时只保存必要的数据。如果数据不再必要，那么把它设置为null，从而释放其引用。这也可以叫作<font size=2 color=green>解除引用</font>。这个建议最适合全局变量和全局对象的属性。局部变量在超出作用域后会被自动解除引用。将内存占用量保持在一个较小的值可以让页面性能更好。
```
function createPerson(name){ 
 let localPerson = new Object(); 
 localPerson.name = name; 
 return localPerson; 
} 
let globalPerson = createPerson("James"); 
// 解除 globalPerson 对值的引用
globalPerson = null;
```
解除引用的关键在于确保相关的值已经不在上下文里了，因此它在下次垃圾回收时会被回收。

在使用垃圾回收的编程环境中，开发者通常无须关心内存管理。但是优秀的程序员应该知道如果通过编码的手段优化管理内存。

<font size=3 color=orange>const 和 let 声明提升性能</font>
const 和 let 都可以声明块级作用域而非函数作用域，相比于使用var，使用 const 和 let 可能会使得垃圾回收程序更早地介入回收应该回收的内存。

<font size=3 color=orange>内存泄漏</font>
JavaScript 中的内存泄露大部分是由不合理的引用导致的。

<font size=2 color=green>避免创建全局变量</font>
```
function setName() {
  // name = 'James' // × 
  cosnt name = 'James' // √
}
```
<font size=2 color=green>定时器在不使用时注意清除</font>
```
const num = 123
let time = setInterval(() => {
  console.log(num) // 定时器只要不清除num就会一直占用内存
})

clearInterval(time)
```
<font size=2 color=green>闭包的使用很容易造成内存泄漏</font>
```
let outer = function() {
  let num = 123
  return function() {
    return num
  }
}

let getNum = outer()
// use getNum
getNum = null //注意使用完后要清除闭包的引用
```
<font size=3 color=orange>隐藏类</font>
Chrome是目前最流行的浏览器，使用 V8 JavaScript 引擎。V8 在将解释后的 JavaScript 代码编译为实际的机器码时会利用<font size=2 color=green>“隐藏类”</font>。运行期间，V8 会将创建的对象与隐藏类关联起来，以跟踪他们的属性特征。<font size=2 color=green>能够共享相同的隐藏类的对象性能会更好。</font>
```
function Person() {
  this.sex = 'male'
}
let p1 = new Person()
let p2 = new Person()
```
上述代码V8会在后台配置，让p1、p2两个实例共享相同的隐藏类，因为它们共享同一个构造函数和原型。
```
p2.name = 'James'
```
增加这段代码后，p1、p2就会对应不同的隐藏类。根据这种操作的频率和隐藏类的大小，这有可能会对性能产生明显影响。
<font size=2 color=green>解决方案是在构造函数中一次性声明所有属性</font>
```
function Person(name) {
  this.sex = 'male'
  this.name = name
}
let p1 = new Person()
let p2 = new Person('James')
```
这样两个实例又可以共享一个隐藏类了，从而获得潜在得性能提升

<font size=2 color=green>动态删除属性与动态添加属性导致的后果一样</font>
```
function Person() {
  this.sex = 'male'
  this.age = 18
}
let p1 = new Person()
let p2 = new Person()

delete p1.age
```
上面代码结束后，即使两个实例使用了同一个构造函数，他们也不再共享一个隐藏类
<font size=2 color=green>解决方案是将不想要得属性设置为null</font>
```
function Person() {
  this.sex = 'male'
  this.age = 18
}
let p1 = new Person()
let p2 = new Person()

p1.age = null
```
这样两个实例将继续共享一个隐藏类，同时也能达到删除引用值获得垃圾回收的效果
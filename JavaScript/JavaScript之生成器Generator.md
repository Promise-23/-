#### 基本概念

Generator 函数是 ECMAScript 6 新增的一个极为灵活的结构，拥有在一个函数块内暂停和恢复代码执行的能力。使用生成器可以自定义迭代器和实现协程。

<font size=2 color=green>函数名称前面加一个星号（*）表示它是一个生成器</font>
```
function* generatorFnA() {}
function * generatorFnB() {}
function *generatorFnC() {}
```
标识生成器函数的星号不受两侧空格的影响,上面三种方式都是合理的定义。

生成器对象一开始处于暂停执行（suspended）的状态。与迭代器相似，生成器对象也实现了 Iterator 接口，因此具有 next()方法。调用这个方法会让生成器开始或恢复执行
```
function* generatorFn() {} 
const g = generatorFn(); 
console.log(g); // generatorFn {<suspended>} 
console.log(g.next); // f next() { [native code] }
```
<font size=2 color=green>生成器函数只会在初次调用 next()方法后开始执行</font>
```
function* generatorFn() { 
 console.log('foo'); 
} 
// 初次调用生成器函数并不会打印日志
let generatorObject = generatorFn(); 
generatorObject.next(); // foo
```
<font size=2 color=green>生成器对象实现了 Iterable 接口，它们默认的迭代器是自引用的</font>
```
function* generatorFn() {}
const g = generatorFn(); 
console.log(g === g[Symbol.iterator]()) // true
```

#### yield 表达式
yield 关键字可以让生成器停止和开始执行，也是生成器最有用的地方。生成器函数在遇到 yield关键字之前会正常执行。遇到这个关键字后，执行会停止，函数作用域的状态会被保留。停止执行的生成器函数只能通过在生成器对象上调用 next()方法来恢复执行
```
function* generatorFn() { 
 yield 'foo'; 
 yield 'bar'; 
 return 'baz'; 
} 
let generatorObject = generatorFn(); 
console.log(generatorObject.next()); // { done: false, value: 'foo' } 
console.log(generatorObject.next()); // { done: false, value: 'bar' } 
console.log(generatorObject.next()); // { done: true, value: 'baz' }
```
通过 yield 关键字退出的生成器函数会处在 done: false 状态；通过 return 关键字退出的生成器函数会处于 done: true 状态。

> yield 关键字只能在生成器函数内部使用，用在其他地方会抛出错误。
<font size=2 color=green>可以使用星号增强 yield 的行为，让它能够迭代一个可迭代对象，从而一次产出一个值</font>
```
function* generatorFn() { 
 yield* [1, 2, 3]; 
} 
let g = generatorFn(); 
for (const x of g) { 
 console.log(x); 
} 
// 1 
// 2 
// 3
```
#### 提前终止生成器

<font size=2 color=green>return()和 throw()方法都可以用于强制生成器进入关闭状态</font>
```
function* generatorFn() {} 
const g = generatorFn(); 
console.log(g); // generatorFn {<suspended>} 
console.log(g.next); // f next() { [native code] } 
console.log(g.return); // f return() { [native code] } 
console.log(g.throw); // f throw() { [native code] }
```
+ return方式终止生成器

```
function* generatorFn() { 
 for (const x of [1, 2, 3]) { 
 yield x; 
 } 
} 
const g = generatorFn(); 
console.log(g); // generatorFn {<suspended>} 
console.log(g.return(4)); // { done: true, value: 4 } 
console.log(g); // generatorFn {<closed>}
```
生成器对象使用return（）方法进入关闭状态后，就无法恢复了，如上所示，生成器是closed状态后再次调用next（）方法后会显示为done:true状态

+ throw方式终止生成器
```
function* generatorFn() { 
 for (const x of [1, 2, 3]) { 
 yield x; 
 } 
} 
const g = generatorFn(); 
console.log(g); // generatorFn {<suspended>} 
try { 
 g.throw('foo'); 
} catch (e) { 
 console.log(e); // foo 
} 
console.log(g); // generatorFn {<closed>}
```
throw()方法会在暂停的时候将一个提供的错误注入到生成器对象中。如果错误未被处理，生成器就会关闭.不过，假如生成器函数内部处理了这个错误，那么生成器就不会关闭，而且还可以恢复执行。错误处理会跳过对应的 yield，因此在这个例子中会跳过一个值
```
function* generatorFn() { 
 for (const x of [1, 2, 3]) { 
 try { 
 yield x; 
 } catch(e) {} 
 } 
}
const g = generatorFn(); 
console.log(g.next()); // { done: false, value: 1} 
g.throw('foo'); 
console.log(g.next()); // { done: false, value: 3}
```
> 如果生成器对象还没有开始执行，那么调用 throw()抛出的错误不会在函数内部被捕获，因为这相当于在函数块外部抛出了错误。

#### JavaScript迭代器

JavaScript中很多内置类型都实现了Iterable接口：字符串、数组、映射、集合、arguments对象还有NodeList等DOM集合类型等。实现Iterable接口要求同时具备两种能力：支持迭代的自我识别能力和创建实现Iterable接口的对象的能力。这意味着必须暴露一个属性作为‘默认迭代器’，这个属性必须使用特殊的Symbol.iterable作为键。这个默认迭代器属性必须引用一个迭代器工厂函数，调用这个工厂函数必须返回一个新的迭代器。

<font size=2 color=green>检查是否存在默认迭代器属性</font>
```
let obj = {}
console.log(obj[Symbol.iterable]) // undefined

let arr = ['1', '2', '3']
console.log(arr[Symbol.iterable]) // f values() { [native code] }
console.log(arr[Symbol.iterator]()); // ArrayIterator {}
```
实现可迭代协议的所有类型都会自动兼容接收可迭代对象的任何语言特性。

迭代器 API 使用 next()方法在可迭代对象中遍历数据。每次成功调用 next()，都会返回一个 IteratorResult 对象，其中包含迭代器返回的下一个值。若不调用 next()，则无法知道迭代器的当前位置。
IteratorResult包含done和value两个属性。done是个布尔值，表示是否还可以再次调用next()取得下一个值；value包含可迭代对象的下一个值（done为false）或者undefined（done为true）
```
let arr = ['1', '2']
let iter = arr[Symbol.iterator]()

// 执行迭代
console.log(iter.next()); // { value: '1', done: false } 
console.log(iter.next()); // { value: '2', done: false } 
console.log(iter.next()); // { value: undefined, done: true }
console.log(iter.next()); // { value: undefined, done: true }
```

#### 自定义迭代器
```
// 自定义迭代器
class Counter {
  constructor (limit) {
    this.limit = limit
  }

  [Symbol.iterator]() {
    let counter = 1,
        limit = this.limit
    return {
      next() {
        if (counter <= limit) {
          return { done: false, value: counter++ }
        }else {
          return { done: true, value: undefined }
        }
      },
      return() { 
        console.log('Exiting early')
        return { done: true }
      }
    }
  }
}

// 创建迭代器并使用for-of遍历
let counter = new Counter(3)
for(let i of counter) {
  console.log(i)
}
// 1
// 2
// 3
```

### 迭代器的终止
上面自定义迭代器中，在next方法后面定义的return方法可以指定迭代器提前关闭时执行的逻辑，return()方法必须返回一个有效的 IteratorResult 对象。简单情况下，可以只返回{ done: true }。for-of 循环通过 break、continue、return 或 throw 提前退出。
```
let counter = new Counter(6)
for(let i of counter) {
  if (i > 2) {
    throw new Error('error')
  }
  console.log(i)
}

// 1 
// 2 
// Exiting early
```
> 数组的迭代器是不能关闭的，可以继续从上次离开的地方继续迭代

```
let a = [1, 2, 3, 4, 5]; 
let iter = a[Symbol.iterator](); 
for (let i of iter) { 
 console.log(i); 
 if (i > 2) { 
 break 
 } 
} 
// 1 
// 2 
// 3 
for (let i of iter) { 
 console.log(i); 
} 
// 4 
// 5
```

因为 return()方法是可选的，所以并非所有迭代器都是可关闭的。要知道某个迭代器是否可关闭，可以测试这个迭代器实例的 return 属性是不是函数对象。不过，仅仅给一个不可关闭的迭代器增加这个方法并不能让它变成可关闭的。这是因为调用 return()不会强制迭代器进入关闭状态。即便如此，return()方法还是会被调用。
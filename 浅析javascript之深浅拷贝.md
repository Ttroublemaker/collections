
# 浅析javascript之深浅拷贝

在日常搬砖过程中，经常会接触到数据拷贝，闲来无事，做下简单总结记录。

其实说到拷贝，我们常常又将其分为深拷贝&浅拷贝，那么二者的区别是什么呢？

**何为深拷贝 & 浅拷贝**

- 浅拷贝：  
浅拷贝只会将对象的各个属性进行依次复制，并不会进行递归复制，也就是说只会赋值目标对象的第一层属性。对于目标对象第一层为基本数据类型的数据，就是直接赋值，即「传值」； 而对于目标对象第一层为引用数据类型的数据，就是直接赋存于栈内存中的堆内存地址，即「传址」。所以当对源对象或目标对象进行操作时，可能会互相影响。

- 深拷贝：  
深拷贝不同于浅拷贝，它不只拷贝目标对象的第一层属性，而是递归拷贝目标对象的所有属性。它是在目标对象的基础上另外创造一个一模一样的对象，新旧对象不共享内存，因此修改其中的一个对象不会影响到另一个对象。可以理解为当拷贝动作完成后，源对象与拷贝对象已经没有任何关系了。
现在搞清了深浅拷贝的区别及含义，那么实现深浅拷贝的方式有哪些呢？

**深拷贝实现方式**
1. JSON.parse(JSON.stringify(obj)
```js
const source = {
    num: 1,
    str: 'string',
    arr: ['array'],
    obj: { object:'object' }
}
const target = JSON.parse(JSON.stringify(source));
source.num=2;
source.obj.object='newStr'

console.log(target === source); 
// false
console.log(target); 
// {
//   num: 1,
//   str: 'string',
//   arr: ['array'],
//   obj: { object:'object' }
// }
```
JSON.stringify()会将一个JS对象序列化一个JSON字符串，JSON.parse()则是将JSON字符串反序列化为一个JS对象，JSON字符串转化为JS对象时会建一个新对象。然而这样的黑科技仍然有如下缺陷：

- 会忽略undefined及Symbol
- 不能序列化函数
- 对循环引用的对象无效

比如下面的例子：
```js
const symbol=Symbol()
const fun = () =>{return 'i am a function'}
const source = {
    symbol,  
    fun,
    un: undefined
}
const target = JSON.parse(JSON.stringify(source));
console.log(target); 
// {}

let obj = { a: 1, b: 3 }
obj.c = obj
JSON.stringify(obj)
// Uncaught TypeError: Converting circular structure to JSON
```

2. 递归拷贝
```js
const deepClone = function (obj) {
  if (typeof obj !== 'object') {
    return obj
  }
  const newObj = obj instanceof Array ? [] : {}
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      newObj[key] = typeof obj[key] === 'object' ? deepClone(obj[key]) : obj[key]
    }
  }
  return newObj
}
```
**浅拷贝实现方式**
1. Object.assign({},obj)

Object.assign() 方法用于将所有可枚举属性的值从一个或多个源对象复制到目标对象，并返回目标对象。
```js
const source = {
  num: 1,
  str: 'string',
  arr: ['array'],
  obj: { object: 'object' }
}
const target = Object.assign({}, source);
source.num = 2;
source.obj.object = 'newStr'

console.log(target === source);
// false 
console.log(target);
// {
//   num: 1,
//   str: 'string',
//   arr: ['array'],
//   obj: { object: 'newStr' }
// }
```
从结果可以看出：通过Object.assign({},obj)方法拷贝的对象是浅拷贝，遇到引用类型的属性值，也只是拷贝引用指针，而不是开辟一块新内存。

2. 解构赋值

ES6中提供了一种直接操作对象的方式：解构赋值，通过对源对象的解构并赋值来达到拷贝的目的：

```js
const source = {
  num: 1,
  str: 'string',
  arr: ['array'],
  obj: { object: 'object' }
}
const target = { ...source };
source.num = 2;
source.obj.object = 'newStr'

console.log(target === source);
// false 
console.log(target);

// {
//   num: 1,
//   str: 'string',
//   arr: ['array'],
//   obj: { object: 'newStr' }
// }
```
end，撒花！


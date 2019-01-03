不知道小伙伴们ES6的特性学的怎么样了？ES2016(ES7)和ES2017(ES8)都已经要出来了，本文为大家整理介绍一下ES7的新特性。

## ES7特性
ES7特性只有两个：
- `Array.prototype.includes`
- Exponentiation Operator(求幂运算) `**`



###  `Array.prototype.includes`

`Array.prototype.includes（value：任意值）： boolean`

`includes()` 方法用来判断一个数组是否包含一个指定的值，根据情况，如果包含则返回 true，否则返回false。

```
let a = [1, 2, 3];

a.includes(2); // true 

a.includes(4); // false
```

语法：

```
arr.includes(searchElement)
arr.includes(searchElement, fromIndex)
```

看到这里大家会感觉和indexOf方法很像，
```
[1, 2, 3].includes(1)   // true
[1, 2, 3].indexOf(1)    // 0
```
但是有一点和indexOf不一样，`includes()` 方法能找到 NaN，而 `indexOf()` 不行：
```
[NaN].includes(NaN)    //true
[NaN].indexOf(NaN)     //-1
```
接下来我们看下polyfil
```
if (!Array.prototype.includes) {
  Object.defineProperty(Array.prototype, 'includes', {
    value: function(searchElement, fromIndex) {
      if (this == null) {
        throw new TypeError('"this" is null or not defined');
      }

      var o = Object(this);
      
      var len = o.length >>> 0;

      if (len === 0) {
        return false;
      }

      var n = fromIndex | 0;

      var k = Math.max(n >= 0 ? n : len - Math.abs(n), 0);

      while (k < len) {
        if (o[k] === searchElement) {
          return true;
        }
        k++;
      }

      return false;
    }
  });
}
```
可以看到`includes()`方法的实现，就是用while循环数组，如果找到了searchElement便返回`true`，否则返回`false`。


### Exponentiation Operator(求幂运算) `**`
在ES6，如果你想做一些求幂运算的话，你需要用到`Math.pow(x, y)`方法，

现在在ES7 /ES2016，以数学向导的开发者可以使用更短的语法:
```
let a = 7 ** 12
let b = 2 ** 7
console.log(a === Math.pow(7,12)) // true
console.log(b === Math.pow(2,7)) // true
```
`**`是一种运算符，就像`+`,`-`,`*`,`/`一样，还可以
```
let a = 7
a **= 12
let b = 2
b **= 7
console.log(a === Math.pow(7,12)) // true
console.log(b === Math.pow(2,7)) // true
```


参考阅读：

1. https://www.cnblogs.com/zhuanzhuanfe/p/7493433.html
2. https://w3ctech.com/topic/1614
3. https://www.jianshu.com/p/a138a525c287
4. https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/includes#%E6%B5%8F%E8%A7%88%E5%99%A8%E5%85%BC%E5%AE%B9%E6%80%A7
---
title: Javascript数组方法总结（一）
---
####一：检测数组的方法
`Array.isArray()`用于确定传递的值是否是一个`Array`，返回`true` or `false`；
```
Array.isArray(new Array())
//true
Array.isArray(Array.prototype)
//true
```
`instanceof` 和`Array.isArray()`
当检测`Array`实例时, `Array.isArray` 优于 `instanceof`,因为`Array.isArray`能检测`iframes`.
```
var iframe = document.createElement('iframe')
document.body.appendChild(iframe)
var xArray = window.frames[window.frames.length - 1].Array
> window.frames[window.frames.length - 1].Array
< ƒ Array() { [native code] }
> var arr = new xArray(1,2,3)
> Array.isArray(arr)
< true
> arr instanceof xArray
< true
> arr instanceof Array
< false
```
Polyfill：`Array.isArray()`是`es5`的方法，如果不支持可以用以下代码来创建该方法：
```
> Object.prototype.toString.call([1,2,3])
< "[object Array]"

if(!Array.isArray){
  Array.prototype.isArray = function(arg){
    return Object.prototype.toString.call(arg) === '[object Array]';
  }
}
```
####二：转换方法（toString(),toLocalString(),valueOf()）
所有对象都具有`toString()`,`toLocalString()`,`valueOf()`方法,调用`toString()`方法返回的是逗号分隔的字符串，为了创建字符串会调用数组每一项的`toString()`方法，`valueof()`返回的依然是数组。调用`join(‘分割符’)`方法返回分割符隔开的字符串，例如`‘hello&world’`；
####三：栈方法（push、pop）
栈方法`（LIFO:Last In First Out）`（在栈顶，后进先出）（在数组末尾操作数组）：
```
var arr = [1,2,3,4];
arr.push(2,6,7)  ;  //push()返回数组的长度；
arr.pop();//pop()移除数组末尾项返回被移除的元素；
```
####四：队列方法（shift、unshift）
队列方法（FIFO：First In First Out）（先进先出）（在数组前端操作数组）：
```
> var arr = [1,5,2,3,6];
> arr.shift()
< 1
> arr.unshift(6)
< 5
> arr
< [6, 5, 2, 3, 6]
arr.shift()  //移除数组的第一项，返回被移除的项；
arr.unshift()  //在数组前端添加项并返回数组的长度；
```
####五：重排序方法（reverse、sort）
**reverse()**方法将数组中元素的位置颠倒,并返回该数组。该方法会`改变原数组`。
**sort()**方法用[原地算法](https://en.wikipedia.org/wiki/In-place_algorithm)对数组的元素进行排序，并返回数组。排序算法现在是[稳定的](https://zh.wikipedia.org/wiki/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95#.E7.A9.A9.E5.AE.9A.E6.80.A7)。默认排序顺序是根据字符串Unicode码点。
由于它取决于具体实现，因此无法保证排序的时间和空间复杂性。
语法：`arr.sort([compareFunction])`
```
var arr = [1,2,3,4,5,6];
arr.sort((a,b) => a < b)  
//返回：[6, 5, 4, 3, 2, 1],原数组会被改变
arr.reverse()  //将数组反转；
arr.sort()  //sort传入一个比较函数或者排序规则的函数，返回新数组；
```
####六：操作方法（concat(),slice(),splice()）
**concat()** 拼接数组，基于当前数组创建一个新数组；
**slice()** 传入位置索引截取数组，返回新数组，不会改变原来的数组；
将类数组对象变为数组对象
```
function list() {
  return Array.prototype.slice.call(arguments);
}
list(1,2,3)
//(3) [1, 2, 3]
```
**splice()** 方法通过删除或替换现有元素或者原地添加新的元素来修改数组,并以数组形式返回被修改的内容。此方法会改变原数组。
语法：`array.splice(start[, deleteCount[, item1[, item2[, ...]]]])`
```
var myFish = ["angel", "clown", "mandarin", "sturgeon"];
var removed = myFish.splice(2, 0, "drum");   //从第 2 位开始删除 0 个元素，插入“drum”
// 运算后的 myFish: ["angel", "clown", "drum", "mandarin", "sturgeon"]
// 被删除的元素: [], 没有元素被删除

var myFish = ['angel', 'clown', 'drum', 'mandarin', 'sturgeon'];
var removed = myFish.splice(3, 1);  //从第 3 位开始删除 1 个元素
// 运算后的 myFish: ["angel", "clown", "drum", "sturgeon"]
// 被删除的元素: ["mandarin"]
```
```
var hello = ['jack','mate','mary','jhon']; 
hello.slice(1);  
//返回["mate", "mary", "jhon"] 原来数组不变["jack", "mate", "mary", "jhon"]

hello.splice(1,2,'first','second')
["mate", "mary"]   //返回被删除的像：["mary", "jhon"]，
hello
["jack", "first", "second", "jhon"]   //原数组被改变
```
####七：位置方法（indexOf(),lastIndexOf()）
indexOf()（从数组开头查找）,lastIndexOf()（从数组结尾查找）
 返回查找项在数组中的位置索引，如果找得到返回位索引，如果找不到返回-1
```
var s = [1,5,6,2,1,3,'hello'];
s.indexOf(1)
0
s.lastIndexOf(1)
4
s.lastIndexOf(4,4)
-1
s.indexOf(4,4)
-1
```
####八：迭代方法（every(),filter(),forEach(),map(),some()）
这些方法传入函数，`forEach()`没有返回值，`map()`返回数组，`filter`返回为`true的数组`

**every()**方法，查询数组中的项只要有一个条件不满足就返回false；

![](https://upload-images.jianshu.io/upload_images/5541401-be4325c6265500db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**some()**方法，查询数组中的项，只要有一个项满足条件就会返回true;

![](https://upload-images.jianshu.io/upload_images/5541401-bb86bc1c6e33d54d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**filter()**方法过滤数组，返回满足条件的数组：

![](https://upload-images.jianshu.io/upload_images/5541401-6aaaf8aa15ca47b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**map()**方法同样返回数组：

![](https://upload-images.jianshu.io/upload_images/5541401-ddeb3937ea8c5a81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**forEach()**方法跟for循环遍历数组一样，没有返回值：

![](https://upload-images.jianshu.io/upload_images/5541401-f8547536fefd9f09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####九：缩小方法（reduce，reduceRight）
**reduce()**（从数组前面开始）,**reduceRight()**（从数组末尾开始），这两个方法都会迭代数组的所有项，然后构建一个最终返回的值。
语法：`arr.reduce(callback[, initialValue])`
`reducer 函数`接收4个参数：
`Accumulator (acc) (累计器)` , `Current Value (cur) (当前值)` , `Current Index (idx) (当前索引)`, `Source Array (src) (源数组)`
，您的 reducer 函数的返回值分配给累计器，该返回值在数组的每个迭代中被记住，并最后成为最终的单个结果值。
```
var arr = [1,[2,[3,[6]]]];
function reduceArr(arr){
  return arr.reduce((prev,cur) => {return prev.concat(Array.isArray(cur) ? reduceArr(cur) : cur)},[])
}
reduceArr(arr); //[1, 2, 3, 6]

//将二维数组转化为一维数组
var flattened = [[0, 1], [2, 3], [4, 5]].reduce(
 ( acc, cur ) => acc.concat(cur),
 []
);
// flattened is [0, 1, 2, 3, 4, 5]
```
```
const array1 = [[0, 1], [2, 3], [4, 5]].reduceRight(
  (accumulator, currentValue) => accumulator.concat(currentValue)
);

console.log(array1);
// expected output: Array [4, 5, 2, 3, 0, 1]
```
参考：
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array

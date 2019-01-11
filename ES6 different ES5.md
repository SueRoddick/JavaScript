##  1、ler 与var 作用域不同 输出不用

```javascript
var a = []
for (var a = 0; a < 10; a++) {
  var c = i
  a[i] = function () {
    console.log(c)
  }
}
```

输出 a\[6]()         // 结果为9

如果把var c=i  换成  let c=i  则输出结果为6

```javascript
function f () {
  console.log('outside')
}
;(function () {
  if (false) {
    function f () {
      console.log('inside')
    }
  }
  f()
})()
```

// 上面代码在ES5中运行为 inside ；   ES6 则会得到outside

###  2、codePointAt方法

ES6提供了codePointAt方法 ；能够正确处理4个字节存储的字符，返回一个字符的Unicode编号；

// 对于两个字节存储的常规字符放回的结果和charCodeAt方法相同。 String.fromCodePont方法与codePointAt方法刚好相反

codePointAt('𠮷') // 134071

String.fromCodePont(134071) // 𠮷

// Unicode表示法： ES6做出了改进，只要将超过0xFFFF的编号放入大括号中，就能正确解读

;('\u{20BB7}') // "𠮷"

### 3、正则表达式

##### ES6 对正则表达式添加了u修饰符，处理发育\uFFFF的Unicode字符

```javascript
var s = '𠮷'
/^.$/.test(s)   // false
/^.$/u.test(s)  // true
```

##### ES6还为正则表达式添加了 y 修饰符，叫做“粘连”修饰符。

y 修饰符的作用与 g 修饰符类似，也是全局匹配，后一次匹配都从上一次匹配成功的下一个位置开始。不同之处在于，g 修饰符只要剩余位置中存在匹配就可，而 y 修饰符确保匹配必须从剩余的第一个位置开始，这也就是“粘连”的含义。

```javascript
var s = 'aaa_aa_a';  
var r1 = /a+/g;  
var r2 = /a+/y;    
console.log(r1.exec(s)) //["aaa", index: 0, input: "aaa_aa_a"]  
console.log(r2.exec(s)) //["aaa", index: 0, input: "aaa_aa_a"]  
console.log(r1.exec(s)) //["aa", index: 4, input: "aaa_aa_a"]  
console.log(r2.exec(s)) //null  
console.log(r1.exec(s)) //["a", index: 7, input: "aaa_aa_a"]  
console.log(r2.exec(s)) //["aaa", index: 0, input: "aaa_aa_a"]
```

上面代码有两个正则表达式，一个使用 g 修饰符，另一个使用 y 修饰符，这两个正则表达式各执行了三次，第一次执行的时候，两者行为相同，剩余字符串都是 _aa_a。由于 g 修饰符没有位置要求，所以第二次执行会返回结果，而 y 修饰符要求匹配必须从头部开始，所以返回null。

如果改一下正则表达式，保证每次都能头部匹配，y修饰符就会返回结果了。

```javascript
var s = 'aaa_aa_a';
var r1 = /a+_/g;
var r2 = /a+_/y;
console.log(r1.exec(s)) //["aaa", index: 0, input: "aaa_aa_a"]
console.log(r2.exec(s)) //["aaa_", index: 0, input: "aaa_aa_a"]
console.log(r1.exec(s)) //["aa", index: 4, input: "aaa_aa_a"]
console.log(r2.exec(s)) //["aa_", index: 4, input: "aaa_aa_a"]
console.log(r1.exec(s)) //["a", index: 7, input: "aaa_aa_a"]
console.log(r2.exec(s)) /
```

上面代码每次匹配，都是从剩余字符串的头部开始。使用 lastIndex 属性，可以更好地说明 y 修饰符。

```javascript
var s = 'aaa_aa_a';

const REGEX = /a/y;
// 指定从2号位置开始匹配
REGEX.lastIndex = 2;
// 不是粘连，匹配失败
REGEX.exec('aya') // null
// 指定从3号位置开始匹配
REGEX.lastIndex = 3;
// 3号位置是粘连，匹配成功
const match = REGEX.exec('xaxa');
match.index // 3
REGEX.lastIndex // 4
```

实际上，y 修饰符号隐含了头部匹配的标识 ^。实际上，y 修饰符号隐含了头部匹配的标识 ^。

```javascript
/b/y.exec('aba')
// null
```

上面代码由与不能保证头部匹配，所以返回 null ，y修饰符的设计本意，就是让头部匹配的标识 ^ 在全局匹配中都有效。

在 split 方法中使用 y 修饰符，原字符串必须以分隔符开头。这也意味着，只要匹配成功，数组的第一个成员肯定是空字符串。

```javascript
'x##'.split(/#/y)
//["x", "", ""]
'##x'.split(/#/y)
// ["", "", "x"]
```

后续的分隔符只有紧跟前面的分隔符，才会被识别。

```javascript
'#x#'.split(/#/y)
// ["", "x", ""]
'##'.split(/#/y)
// ["", "", ""]
```

下面是字符串对象的 replace 方法例子

```javascript
const REGEX = /a/gy;
'aaxa'.replace(REGEX, '-') // '--xa'
```

上面代码中，最后一个 a 因为不是出现在下一次匹配的头部，所以不会被替换。

单单一个 y 修饰符对，math 方法，只能返回第一个匹配，必须与 g 修饰符联用，才能返回所以匹配。

```javascript
'a1a2a3'.match(/a\d/y) // ["a1"]
'a1a2a3'.match(/a\d/gy) // ["a1", "a2", "a3"]
```

y 修饰符的一个应用，是从字符串提取 token（词元），y 修饰符确保了匹配之间不会漏掉的字符。

```javascript
const TOKEN_Y = /\s*(\+|[0-9]+)\s*/y;
const TOKEN_G  = /\s*(\+|[0-9]+)\s*/g;
 
tokenize(TOKEN_Y, '3 + 4')
// [ '3', '+', '4' ]
tokenize(TOKEN_G, '3 + 4')
// [ '3', '+', '4' ]
 
function tokenize(TOKEN_REGEX, str) {
  let result = [];
  let match;
  while (match = TOKEN_REGEX.exec(str)) {
    result.push(match[1]);
  }
  return result;
}
```

上面代码中，如果字符串里面没有非法字符，y修饰符与g修饰符的提取结果是一样的。但是，一旦出现非法字符，两者的行为就不一样了。

```javascript
tokenize(TOKEN_Y, '3x + 4')// [ '3' ]
tokenize(TOKEN_G, '3x + 4')// [ '3', '+', '4' ]
```

上面代码中，g修饰符会忽略非法字符，而y修饰符不会，这样就很容易发现错误。

##  4 、数值

二进制  0b

八进制 0o（逐步淘汰ES5中八进制前缀0）

Number.isFinite()   

Number.isNaN()

用Number.isSafeInteger()来判断一个整数是否在安全范围（-2<sup>53</sup>~2<sup>53</sup>)

ES6 引入了`Number.MAX_SAFE_INTEGER`和`Number.MIN_SAFE_INTEGER`这两个常量，用来表示这个范围的上下限。

Math.trunc()  用于去除一个数的小数部分，返回整数（4.9返回4）

`Number.isInteger()`用来判断一个数值是否为整数。JavaScript 内部，整数和浮点数采用的是同样的储存方法，所以 2 和 2.0 被视为同一个值。

ES6 将全局方法`parseInt()`和`parseFloat()`，移植到`Number`对象上面，行为完全保持不变。

ES6 在`Number`对象上面，新增一个极小的常量`Number.EPSILON`。根据规格，它表示 1 与大于 1 的最小浮点数之间的差。`Number.EPSILON`实际上是 JavaScript 能够表示的最小精度。误差如果小于这个值，就可以认为已经没有意义了，即不存在误差了。

#### 双曲函数方法

ES6 新增了 6 个双曲函数方法。

- `Math.sinh(x)` 返回`x`的双曲正弦（hyperbolic sine）

- `Math.cosh(x)` 返回`x`的双曲余弦（hyperbolic cosine）

- `Math.tanh(x)` 返回`x`的双曲正切（hyperbolic tangent）

- `Math.asinh(x)` 返回`x`的反双曲正弦（inverse hyperbolic sine）

- `Math.acosh(x)` 返回`x`的反双曲余弦（inverse hyperbolic cosine）

- `Math.atanh(x)` 返回`x`的反双曲正切（inverse hyperbolic tangent

  ES2016 新增了一个指数运算符（`**`）。

  ```javascript
  2 ** 2 // 4
  2 ** 3 // 8
  
  // 相当于 2 ** (3 ** 2)
  2 ** 3 ** 2  // 512
  ```

```javascript
let a = 1.5;
a **= 2;
// 等同于 a = a * a;

let b = 4;
b **= 3;
// 等同于 b = b * b * b;
```

## 5、对象

\_\_proto\__是每个对象都有的一个属性，而prototype是函数才会有的属性!!!

__ 使用Object.getPrototypeOf()代替\_\_proto__!!!

### `prototype`

几乎所有的函数（除了一些内建函数）都有一个名为prototype（原型）的属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以有特定类型的所有实例共享的属性和方法。prototype是通过调用构造函数而创建的那个对象实例的原型对象。`hasOwnProperty()`判断指定属性是否为自有属性；in操作符对原型属性和自有属性都返回true。 

### `__proto__`

对象具有属性`__proto__`，可称为隐式原型，一个对象的隐式原型指向构造该对象的构造函数的原型，这也保证了实例能够访问在构造函数原型中定义的属性和方法

### `Object.getPrototypeOf()` 与 `object.setprototypeof()`

一个对象实例通过内部属性`[[Prototype]]`跟踪其原型对象。使用原型对象的好处是可以让所有对象实例共享它所包含的属性和方法。可以调用对象的`Object.getPrototypeOf()`方法读取`[[Prototype]]`属性的值，也可以使用`isPrototypeOf()`方法检查某个对象是否是另一个对象的原型对象。大部分JavaScript引擎在所有对象上都支持一个名为`__proto__`的属性，该属性可以直接读写`[[Prototype]]`属性。 

Object.setPrototypeOf 方法的作用与`__proto__`相同，用来设置一个对象的 prototype 对象，返回参数对象本身，它是 ES6 正式推荐的设置原型对象的方法。

Object.setPrototypeOf() （写操作）、Object.getPrototypeOf()（读操作）两个方法配套。

### `symbol`

ES6引入了一种新的原始数据类型Symbol，表示独一无二的值。

基本数据类型有6种：Undefined、Null、布尔值（Boolean）、字符串（String）、数值（Number）、对象（Object）。

这里新添加了一种：Symbol

注意，`Symbol`函数前不能使用`new`命令，否则会报错。这是因为生成的Symbol是一个原始类型的值，不是对象

`Symbol`函数可以接受一个字符串作为参数，表示对Symbol实例的描述，主要是为了在控制台显示，或者转为字符串时，比较容易区分。

## 6、函数扩展

#### `rest参数`

rest参数搭配的变量是一个数组可以使用数组的一切操作。注意的是rest参数之后不能再有其他参数（只能是最后一个参数）否则会报错。

function rest(...values){
let sum=0;
for(var val of values){
sum+=val;
}
return sum;
}

add(1,2,3)  //6

#### `扩展运算符`

扩展运算符则可以看作是rest参数的逆运算。可以将数组转化为参数列表

如：console.log(1,...[2,3,4],5) //1 2 3 4 5 

可以替代apply方法：
Math.max.apply(null,[14,3,7])   //ES5写法

Math.max(...[14,3,7]) //ES6写法

用于合并数组:
[1,2].concat(more) //ES5
[1,2, ...more] //ES6

与解构赋值结合： 
let [first,...rest]=[1,2,3,4,5];

first //1

rest  //[2,3,4,5]

## 7 深拷贝 扩展运算符(...)与Object.assign()
```
var obj = {a: 1, b: 2, c: { a: 3 },d: [4, 5]}
var obj1 = obj
var obj2 = JSON.parse(JSON.stringify(obj))//深拷贝常用方法
var obj3 = {...obj}
var obj4 = Object.assign({},obj)
obj.a = 999
obj.c.a = -999
obj.d[0] = 123
console.log(obj1) //{a: 999, b: 2, c: { a: -999 },d: [123, 5]}
console.log(obj2) //{a: 1, b: 2, c: { a: 3 },d: [4, 5]}
console.log(obj3) //{a: 1, b: 2, c: { a: -999 },d: [123, 5]}
console.log(obj4) //{a: 1, b: 2, c: { a: -999 },d: [123, 5]}
```

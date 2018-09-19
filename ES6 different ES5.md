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
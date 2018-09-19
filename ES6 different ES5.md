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
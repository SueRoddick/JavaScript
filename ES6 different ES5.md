##  1、ler 与var 作用域不同 输出不用

```
var a = []
for (var a = 0; a < 10; a++) {
  var c = i
  a[i] = function () {
    console.log(c)
  }
}
```

输出 a[6]() // 9

// 如果把var c=i  换成  let c=i  则输出结果为6

```
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

// ES6 对正则表达式添加了u修饰符，处理发育\uFFFF的Unicode字符

var s = '𠮷'

/^.$/.test(s)   // false

/^.$/u.test(s)  // true
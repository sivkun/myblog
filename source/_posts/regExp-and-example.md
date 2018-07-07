---
title: regExp-and-example
date: 2017-07-09 16:21:19
tags: [javascript,RegExp]
categories: [web,RegExp]
---
上周复习巩固了正则表达式要点，做个总结！
# 起源
看到别人处理css选择器的时候，使用到了正则表达式，涉及到分组捕获等知识，还有实现数字每三位逗号分隔这种形式，使用正则表达式实现，想破头皮弄不出来，只好重新学习正则表达式。

# 易忘难点
1. 在操作符后面加一个问号?字符（?操作符的一个重载），如`/a+?/`:让表达式变成非贪婪的，进行最小限度的匹配。
2. `分组捕获`：正则表达式有一部分用括号进行分组时，它具有双重责任，同时也创建所谓的捕获。`/(ab)+/`匹配一个或多个连续出现的子字符串‘ab’
3. `或操作符`：可以用(`|`)字符表示或者的关系。`/(ab)+|(cd)+/`匹配出现一次或多次的"ab"或"cd"。
4. `反向引用`：捕获的反向引用。如：`/^([dtn]a\1)/`：可以匹配任意一个以"d""t""n"开头，且后面紧跟着一个a字符，并且后面跟着和第一个捕获相同字符的字符串。`\1`匹配的字符需要在匹配的时候才能确定。
5. 在匹配xml类型标记元素的时候可能会很有用，如：`/<(\w+)>(.+)<\/\1>/`可以匹配`<strong>something</strong>`这样的简单元素。如果不使用反向引用是无法做到的，因为我们无法知道闭合标签和开始标签是否匹配。
6. `\b` : 匹配单词边界,就是位于字符\w（[a-zA-Z0-9_]）和\W[^a-zA-Z0-9_]之间的位置，或者位于字符\w和字符串的开头或者结束之间的位置。
7. `\B`: 匹配非单词边界
8. `exp1(?=exp2)`:正向前瞻（零宽正向先行断言）,要匹配的exp1要满足后面是exp2
9. `exp1(?!exp2)`:负向前瞻（零宽负向先行断言），要匹配的exp1要满足后面是不是exp2
10. `(?:exp)`:正则表达式中小括号具有分组和捕获双重作用，如果在小阔号里面开始加上`?:`则可以使其不被捕获。
<!--more-->
# 用于正则表达式的方法
有2个是正则对象的方法，有4个字符串对象的方法
1. `RegExp.prototype.test()`
2. `RegExp.prototype.exec()`
3. `String.prototype.match()`
4. `String.prototype.search()`
5. `String.prototype.replace()`
6. `String.prototype.split()`
下面分别介绍

## `RegExp.prototype.test()`  
检测正则表达式和指定的字符串是否匹配，返回true或false。 

```js
function testinput(re, str){
    var midstring;
    if (re.test(str)) {
        midstring = " contains ";
    } else {
        midstring = " does not contain ";
    }
    console.log(str + midstring + re.source);
}
```
```js
var reg = /c/g
reg.test('ccc');//true
reg.test('ccc');//true
reg.test('ccc');//true
reg.test('ccc');//false
```
如果regexp是全局匹配，`test`可执行多次，可以手动记录匹配的次数。

## `String.prototypr.search(regexp)` 
执行正则表达式和string对象之间的一个搜索匹配，返回首次匹配索引，否则返回-1。

```js
function testinput(re, str){
  var midstring;
  if (str.search(re) != -1){
    midstring = " contains ";
  } else {
    midstring = " does not contain ";
  }
  console.log (str + midstring + re);
}
```

## `String.prototype.match(regexp)`

如果正则表达式没有 `g` 标志，则 `str.match()` 会返回和 `RegExp.exec()` 相同的结果。而且返回的 Array 拥有一个额外的 `input` 属性，该属性包含被解析的原始字符串。另外，还拥有一个 `index` 属性，该属性表示匹配结果在原字符串中的索引（以0开始）。

如果正则表达式包含 `g` 标志，则该方法返回一个 Array ，它包含所有匹配的子字符串而不是匹配对象。捕获组不会被返回(即不返回`index`属性和`input`属性)。如果没有匹配到，则返回  `null` 。

参看：RegExp 方法
* 如果你需要知道一个字符串是否匹配一个正则表达式 `RegExp` ，可使用 `search()` 。
* 如果你只是需要第一个匹配结果，你可能想要使用 `RegExp.exec()` 。
* 如果你想要获得捕获组，并且设置了全局标志，你需要用 `RegExp.exec()` 。

## `RegExp.prototype.exec(str)`

当正则表达式使用 "g" 标志时，可以多次执行 exec 方法来查找同一个字符串中的成功匹配。当你这样做时，查找将从正则表达式的  lastIndex 属性指定的位置开始。（test() 也会更新 lastIndex 属性）。例如，你使用下面的脚本：

```js
var myRe = /ab*/g;
var str = 'abbcdefabh';
var myArray;
while ((myArray = myRe.exec(str)) !== null) {
  var msg = 'Found ' + myArray[0] + '. ';
  msg += 'Next match starts at ' + myRe.lastIndex;
  console.log(msg);
}
```

## `String.prototype.replace()`  
`str.replace(regexp|substr,newSubStr|function)`  
> https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/replace
```js

"fontFamily".replace(/([A-Z])/g,'-$1').toLowerCase()//"font-family"

function replacer(match, p1, p2, p3, offset, string) {
  // p1 is nondigits, p2 digits, and p3 non-alphanumerics
  return [p1, p2, p3].join(' - ');
}
var newString = 'abc12345#$*%'.replace(/([^\d]*)(\d*)([^\w]*)/, replacer);
//'abc - 12345 - #$*%'
```

## `String.prototype.split()`  
`str.split(separator[,limit])`
参数
`separator`
指定用来分割字符串的字符（串）。`separator` 可以是一个字符串或正则表达式。 如果忽略 `separator`，则返回整个字符串的数组形式。如果 `separator` 是一个空字符串，则 `str` 将会把原字符串中每个字符的数组形式返回。
`limit`
一个整数，限定返回的分割片段数量。`split` 方法仍然分割每一个匹配的 `separato`r，但是返回的数组只会截取最多 `limit` 个元素。
```js
var myString = "Hello 1 word. Sentence number 2.";
var splits = myString.split(/(\d)/);
console.log(splits);
//[Hello ,1, word. Sentence number ,2,.]
```
## 没有捕获的分组(?:)
```js
var pattern = /((?:ninja-)+)sword/;
var ninjas = "ninja-ninja-ninja-sword".match(pattern);
```

# 正则常见解决方案
## 修剪字符串
```js
function trim(str){
  return (str||'').replace(/^\s+|\s+$/g,'');
}
```
## 匹配换行符
```js
var html = "<b>Hello</b>\n<i>world</i>";
/.*/.exec(html)[0]==='<b>Hello</b>'
/[\s\S]*/.exec(html)[0]==="<b>Hello</b>\n<i>world</i>"
/(?:.|\s)*/.exec(html)[0]==='<b>Hello</b>\n<i>world</i>'
```
## Unicode
```js
var text = "\u5FCD\u8005\u30D1\u30EF\u30FC";
var matchAll = /[\w\u0080-\uFFFF_-]+/;
text.match(matchAll)
```
## 转义字符
```js
var pattern = /^((\w+)|(\\.))+$/;
var tests = [
  "formUpdate",
  "form\\.update\\.whatever",
  "form\\:update",
  "\\f\\o\\r\\m\\up\\d\\a\\t\\e",
  "form:update"
];
for(var n = 0;n<tests.length;n++){
  console.log(pattern.test(tests[n]))//true true true true false
}

```
# 牛逼例子
## 例子1
要求：
1. 至少6个字符
2. 至少一个小写字母
3. 至少一个大写字母
4. 至少一个数字
5. 只包含数字和字母

方法一：
匹配多次，每个都满足。
```js
function validate(password) {
  return  /^[A-Za-z0-9]{6,}$/.test(password) &&
          /[A-Z]+/           .test(password) &&
          /[a-z]+/           .test(password) &&
          /[0-9]+/           .test(password) ;
}
```

方法二：

运用正向前瞻（零宽正向先行断言），出神入化、淋漓尽致！
```js
function validate(password) {
  return /^(?=.*\d)(?=.*[a-z])(?=.*[A-Z])[a-zA-Z0-9]{6,}$/.test(password);
}
```
下面是给出解释
/^(?=.*\d)(?=.*[a-z])(?=.*[A-Z])[a-zA-Z0-9]{6,}$/
  ^ assert position at start of the string
  (?=.*\d) Positive Lookahead - Assert that the regex below can be matched
    .* matches any character (except newline)
      Quantifier: Between zero and unlimited times, as many times as possible, giving back as needed [greedy]
    \d match a digit [0-9]
  (?=.*[a-z]) Positive Lookahead - Assert that the regex below can be matched
    .* matches any character (except newline)
      Quantifier: Between zero and unlimited times, as many times as possible, giving back as needed [greedy]
    [a-z] match a single character present in the list below
      a-z a single character in the range between a and z (case sensitive)
  (?=.*[A-Z]) Positive Lookahead - Assert that the regex below can be matched
    .* matches any character (except newline)
      Quantifier: Between zero and unlimited times, as many times as possible, giving back as needed [greedy]
    [A-Z] match a single character present in the list below
      A-Z a single character in the range between A and Z (case sensitive)
  [a-zA-Z0-9]{6,} match a single character present in the list below
    Quantifier: Between 6 and unlimited times, as many times as possible, giving back as needed [greedy]
    a-z a single character in the range between a and z (case sensitive)
    A-Z a single character in the range between A and Z (case sensitive)
    0-9 a single character in the range between 0 and 9
  $ assert position at end of the string
我的理解：
`/^(?=.*\d)(?=.*[a-z])(?=.*[A-Z])[a-zA-Z0-9]{6,}$/`
表达式`(?=exp)`（正向前瞻或者零宽正向先行断言）只是表示一种期望，期望接下来必须要出现的内容，如果不出现则会匹配失败，不占字符位置。
1. `^(?=.*\d)(?=.*[a-z])(?=.*[A-Z])`这么多表示，从字符串开头以后，期望接下来的字符中要出现至少一个数字，至少一个小写字母，至少一个大写字母，满足才能匹配。
2. 其实可以把这些期望去掉理解剩下的部分`/^[a-zA-Z0-9]{6,}$/`，表示字符串中只能出现字母和数字，并且字符长度不能少于6，满足这样的字符串才能匹配。
3. 现在把1和2的限制条件一综合，正是题目的要求。

## 例子2
实现indexOf和lastIndexOf
要求：
  第一个参数：可以是字符串或者正则表达式。
  第二个参数：源字符串的索引，表示开始匹配的位置。
  lastIndexOf是从开始匹配的位置从右向左匹配
  indexOf是从匹配开始的位置从左向右匹配
  返回结果： 正确匹配时，在源字符串中的索引。
            匹配失败时，返回-1
```js
String.prototype.indexOf = function(){
  var a = arguments[0], b = arguments[1];
  var result = 0,subStr = this;
  if( b != undefined ) {
    subStr = this.slice(parseInt(b));
    result += b
  }
  var index = subStr.search(a);
  result = index === -1 ? -1 : result + index;
  return result;
};
String.prototype.lastIndexOf = function(){
  var a = arguments[0], b = arguments[1];
  var result = -1;
  (a instanceof RegExp) && (a=a.source);
  a = new RegExp('(?:\\B|\\b)'+'(?='+a+')[\\s\\S]','g');
  b === undefined && (b = this.length);
  var arr = null;
  while(arr = a.exec(this)){
    if(arr.index <= b)
      result = arr.index;
    else
      break;
  }
  return result;
};
```
## 例子3
实现数字每3位用逗号分隔
```js
function splitNum(n){
 return (''+n).replace(/\B(?=(?:\d{3})+\b)/g,',');
}
splitNum(1234567890)//"1,234,567,890"
```
`'123456789'.replace(/\B(?=(?:\d{3})+\b)/g, ',')`
执行过程：
replace函数会进行多次匹配，\b表示了要匹配到单词边界。
1. 首先从1和2中间开始但是后面有8个数字不满足前瞻性条件
2. 然后到2和3中间，同样不满足
3. 到3和4中间，满足后面有6个数字
4. 把在3和4中间的\B(非单词边界)替换为","。
5. 依次往下匹配替换
6. 最终返回新的字符串"123,456,789"

正则很强大，比常规的实现方法简洁方便很多。
也可以参考：http://www.cnblogs.com/sivkun/p/7123963.html

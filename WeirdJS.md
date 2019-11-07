# JavaScript 怪怪 der

把每天在寫 Javascript 遇到的怪問題都記錄下來，希望可以避免再犯。

## **❄️❄️ `Object.freeze` ❄️❄️ 凍結那時間凍結初遇那一天...**

```javascript
let myObject = {
  a: 1,
  b: 2
};
```

先建立一個看起來很普通的物件，但這不好玩，我們用 `Object.defineProperty` 擴增新的屬性好了：

```javascript
Object.defineProperty(myObject, "c", {
  configurable: false,
  enumerable: true,
  get: () => this.value,
  set: value => {
    this.value = value;
  }
});
```

用 `Object.defineProperty` 建立屬性的好處在於我們可以控制屬性的描述器（Descriptors）： `ACCESSOR` 和 `DATA` ，簡單來說一個是針對物件資料面一個是物件訪問面的一些設定，可參考 MDN[文章](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)。這裡針對 `myObject` 增加了新屬性 `c`， 再用 `Accessor` 設定 `getter` 和 `setter` 的方式。

### setter 是什麼？

在還沒有 `proxy` 可用的年代，我們可以在 setter 做一個介入點，針對該屬性的改變而做某些事情，但這不是我要說的問題，我們再往下看看：

```javascript
console.log(myObject.c); // undefined
```

此時印出來是 `undefined` 沒什麼問題，後來後悔了，希望將 `myObject` 給凍結不讓其他人或其他程式修改，我們可以用 `Object.freeze` 把它給封死。 _`Object.seal()` 跟 `Object.preventExtensions()` 也可以將物件做封裝或防止擴增屬性。_

```javascript
Object.freeze(myObject);
console.log(myObject.c); // undefined
```

這裡也沒什麼問題，`c` 本來就沒設定所以是 `undefined` ，但奇怪的來了

```javascript
myObject.c = 1;
myObject.b = 33;
console.log(myObject.a); // 1
console.log(myObject.b); // 2 - 改不動！是原本的 2
console.log(myObject.c); // 1 - 但它可以被修改！
myObject.c = function(e) {
  return this.a;
};
console.log(myObject.c); // function(e) {return this.a} - 再改～！
myObject.c = 87;
console.log(myObject.c); // 87 - 再改～！
```

這時你可能以為 `Object.defineProperty` > `Object.freeze` ，那在凍結後的 `myObject` 可以在設定 `Object.definePropery` 嗎？

```javascript
Object.defineProperty(myObject, "d", {
  configurable: false,
  enumerable: true,
  get: () => this.value,
  set: value => {
    this.value = value;
  }
});
```

直接跳錯 `Cannot define property d, object is not extensible` 。這也是我們預想到的結果，但是問題來了，如果我真的要 freeze `c` 的屬性，是不是就沒辦法了呢？🤧🤧🤧

---

## **🈚🈚 當 `Object` 裡的 key 是 `Object`，什麼東西消失了... 🈚🈚️**

```javascript
let myObject = {};

let nestedObject1 = {
  a: 1,
  b: 2
};

let nestedObject2 = {
  c: 3,
  d: 4
};

console.log(myObject[nestedObject1]);
console.log(myObject[nestedObject2]);
```

請問上面的 `console.log` 會是什麼結果呢？

```javascript
🔜 undefined
🔜 undefined
```

廢話， `myObject` 根本不存在相對的 key，因為我們還沒賦予任何值給他，那接下來：

```javascript
myObject[nestedObject1] = "value1";
myObject[nestedObject2] = "value2";

console.log(myObject);
```

如果我們直接將相對的 key 賦予值呢，這個`console.log` 會是什麼結果呢？

```javascript
{
  [object Object]: 'value2'
}
```

或許有人會認為應該要有兩個屬性才對，但仔細想想，JS 物件裡的 `key` 是~~沒~~辦法設定為物件的（其實是可以的請參考[下面](#symbol-vs-weakmap-%e8%a7%a3%e6%b1%ba%e6%8a%8a%e7%89%a9%e4%bb%b6%e7%95%b6key%e7%9a%84%e5%95%8f%e9%a1%8c)的說明），所以 JavaScript 會自動轉換成 `[object Object]` 字串當成它的 key。當第一個 key `nestedObject1` 賦予 `value1` 時在設`nestedObject2` 為 `value2` 其實都是針對同一個 `[object Object]`， 最後的 value 會是 `value2`，所以以下兩個 log 都會是同個值：

```javascript
console.log(myObject[nestedObject1]); // value2
console.log(myObject[nestedObject2]); // value2
```

有趣的來了，既然物件會變成 `[object Object]` 字串當 `key` 的話，那下面這段程式碼會怎麼運行呢？

```javascript
myObject[nestedObject1][nestedObject1] = "value3";
console.log(myObject[nestedObject1]); // value2
console.log(myObject[nestedObject1][nestedObject1]); // undefined
```

剛開始接觸 `JavaScript` 的人或許會問為什麼答案不是這樣:

```javascript
{
  [object Object]: {
    [object Object]: 'value3'
  }
}
```

因為在此時物件裡的 `typeof [object Object]` 是 `string` ，也就是 `value2`。因為~~不是~~物件所以沒辦法將 `value2` 設附屬的 key `[object Object]` 。如下列的方式是沒辦法執行的：**（嚴格來說 `String` 也是物件，我們這裡先理解成 String 無法用 [`DotNotation`](https://medium.com/dailyjs/dot-notation-vs-bracket-notation-eedea5fa8572) 賦值 ）**

```javascript
let a;
a["test"] = 123; //"TypeError: Cannot set property 'test' of undefined;

let b = 2;
b["test"] = 123; // 沒報錯
console.log(b); // value是 2， 但 test key消失了 
```

那這時候有人會想， 我們之前例子的 `value3` 到底去了哪？

### 消失的 `value3`

它在 `typeof 'string'` 的 `value2` 裡 賦予 `[object Object]` 卻不像上面的例子報錯，原因是 `typeof string` 下的變數其實也是個物件，擁有 `String.prototype` 所有的屬性和方法可用。

 `('value2')['object Object']` 在JS是合法的寫法，但是 `console.log(myObject[nestedObject1][nestedObject1])` 卻是 `undefined`，那他會在哪裡呢？會在 `myObject[nestedObject1].__proto__` 裡嗎？

```javascript
console.log(typeof myObject[nestedObject1]); // string
console.log(myObject[nestedObject1].__proto__);
// instanceof String.Prototype - [object Object] 會不會在這呢？
```

`myObject[nestedObject1]` 調用了 String.Prototype，它應該會把 `[object Object]: 'value3'` 屬性和值都放進去吧？我們可以用 `Object.hasOwnProperty` 去檢查有沒有相關的key。

```javascript
console.log(myObject[nestedObject1].__proto__.hasOwnProperty('[object Object]'));
console.log(myObject[nestedObject1].__proto__.hasOwnProperty(nestedObject1);
console.log(myObject[nestedObject1].hasOwnProperty('[object Object]'));
console.log(myObject[nestedObject1].hasOwnProperty(nestedObject1));
```

可惜的是上面做法都會回傳 `false`。因為投射在 `__proto__` 物件裡是不能被擴展的：

```javascript
console.log(typeof myObject[nestedObject1]); // string
console.log(Object.isExtensible(myObject[nestedObject1])); //false
```

`Object.isExtensible` 是檢測物件是否可以被擴增，我們這裡檢查了 `myObject[nestedObject1]` 是無法被擴增的。此時物件`[object Object]: 'value3'` 其實沒有報錯就憑空消失了。

這是滿奇怪的，如果不讓擴增原本的 `myObject[nestedObject1]`， JavaScript 應該要報錯才對，但他卻沒這麼做。回頭來看這段 code:

```javascript
let myObject = {};

Object.freeze(myObject);
myObject.a = "1";
console.log(myObject); // {}
```

在 js 裡面如果你用 . `dot notation` 賦予值是不會觸發任何錯誤，我覺得比較好的做法是要在 runtime 就報錯，例如 `Object.defineProperty` 的做法：

```javascript
let myObject = {};

Object.freeze(myObject);
Object.defineProperty(myObject, "a", {
  configurable: false,
  enumerable: true,
  value: "1",
  writable: true
}); // 這行就報錯 TypeError: Cannot define property a, object is not extensible
console.log(myObject);
```

`dot notation` 的做法應該要參照 defineProperty 的報錯方式，才能避免某些變數莫名被 GC 或消失。

### Symbol vs WeakMap 解決把物件當key的問題

回到問題中用物件當key的問題，如果我們要將兩個不同的物件獨立變成某物件的key，可有兩種做法，其中用 `Symbol` ：

```javascript
let myObject = {};
let nestedObject1 = {
  a: 1,
  b: 2
};
let nestedObject2 = {
  a: 1,
  b: 2
};
let myObjectSymbol1 = Symbol(nestedObject1);
let myObjectSymbol2 = Symbol(nestedObject2);

myObject[myObjectSymbol1] = 'value1';
myObject[myObjectSymbol2] = 'value2';

console.log(myObject); // 一個物件擁有兩個獨立Symbol的值    
```
另一種是 `WeakMap` 物件格式:
```javascript
let weakMap = new WeakMap();
let nestedObject1 = {
  a: 1,
  b: 2
};
let nestedObject2 = {
  a: 1,
  b: 2
};

weakMap.set(nestedObject1, 'value1');
weakMap.set(nestedObject2, 'value2');

console.log(weakMap) // WeakMap {{…} => "value2", {…} => "value1"}
console.log(weakMap.get(nestedObject1)); // value1
```
---

## 實在很奇怪，`var` 母仔它到底在幹嘛

新手學JavaScript的前幾個課題應該逃不了提升 `hoisting`，這也是很抽象但卻又不能不懂的JavaScript ~~特別~~功能之一。

來，第一課：

```javascript
console.log(a); // what is this?
```

結果是 `Uncaught ReferenceError: a is not defined`。廢話！都沒跟他說 a 是啥東西。

好我宣告：

```javascript
var a; 
console.log(a); // what is this?
```

結果應該很明確是 `undefined` ，變數被宣告了但卻沒有任何值。

來搬家一下：

```javascript
console.log(a); // what is this?
var a; 
```

咦，搬一下位置後，結果卻是 `undefined` ，為什麼 a 在還沒宣告前log居然是空值？難道他是預言家知道下一行是什麼嗎？

那我先給值總可以了吧：
```javascript
console.log(a); // what is this?
var a = 1; 
```

結果依然是 `undefined`， 'WTF!!'    
其實這就是大家俗稱的 hoisting 也是奇怪的部分，那這章節就結束了...(wei...)

好啦，上面的大概就是大家了解的 `var` 怪怪的地方。所以現在很多人都建議直接用 `let` 和 `const` 就是要防止提升的問題，因為：

```javascript 
var a = 1;
var a = 2;
var a = 3;
```

不管新增幾個都不會報錯，所以你在第1行新增的 `a` 有可能會被第124行的 `a` 給覆蓋。

> 當然在 'use strict' 下是會報錯的

因為有了這特性，所以今天函式也可以用相同的名字，也不會跟你報錯。
```javascript 
function a(){}
var a = 1;
```

他們可以並存，但有趣的是：

```javascript 
function a(){}
var a = 1;
console.log(a); // what is this?
```

這時候的 `a` 會是誰呢？ 如果你說 `a` 那你就對啦。如果調一下位置呢？

```javascript 
var a = 1;
function a(){}
console.log(a); // what is this?
var b = 3;
var c;
```

這時候有人會說應該是函式 `a` 吧？因為他在比較後面，對不起你錯了，結果還是 `1`， 原因一樣跟提升有關，分解圖大概可以透過這樣的方式解釋。

```javascript 
var a // 1. 提升 a
var b // 2. 提升 b
var c // 3. 提升 c
//...不管在哪裡宣告，只要被提升上來，var宣告的都會在最上方，直到沒有被var宣告為止。
function a(){} // 4. 宣告 a 函式

a = 1; // 5. 將 1 賦予 a

console.log(a); // 1
```

以上的兩個案例，基本上跟上面的分解圖是一樣的，就算順序上有差，var宣告的都會被推上去，再來調整一下 log 的順序：

```javascript 
console.log(a); // what is this?
var a = 1;
function a(){}
```

這時候你可能會覺得，啊！應該是 `undefined` ! 因為 `a` 會先提升到上面。  
如果你這樣認為的，那你又錯了，解析圖如下：

```javascript

var a; // 1. 提升這裡沒問題
function a(){} // 2. 函式 function declaration 接在提升的後面
console.log(a); // 3. 這時候的 log a 會是函式本身
a = 1; // 4. 這時候才把 1 賦予 a
```

有趣吧，所以上述的code，在後續印 `a` 會是 1 而不是函式本身。   
再來個加分題：
```javascript 
console.log(a()); // what is this?
var a;
function a(){console.log(a)}
var a = 5;
```
這時候 第一個 a 會是什麼呢？如果你猜的是 `undefined` 的話，那你就對了，因為根據之前的拆解步驟，`a` 會在被呼叫的時候 是 undefined 但是後續呼叫會是 `5`。

拆解圖：
```javascript 
var a; // 1. 提升這裡沒問題
function a(){console.log(a)} // 2. 函式 function declaration 接在提升的後面
console.log(a()); // 3. 呼叫目前 a 的值，目前沒有任何值，所以它會是 'undefined'
a = 5; // 4. 此時把 5 賦予了 a
```

其實到這裡就寫完了，提升這個特性也不是JavaScript奇怪的部分，但是每次在回想起來的時候都覺得它具有滿滿的陷阱，所以以後還是不要用 `var` 會比較好。

> 騙了一個篇幅來寫不是 weird js 的 js，再見

其實還沒結束啦，說到 hoisting就必須了來點 weird part啊，畢竟這些列是 weird 奇怪的奇怪：

```javascript 
for (var i = 0; i < 3; i++ ){
  console.log(i); // what is this?
}
console.log(i);
```
這題應該蠻簡單的，答案就是順序的 `0, 1, 2`，然後因為用 `var` 宣告的關係，第二個log因被提升到上一層而變成`3`，分解圖：

```javascript 
var i; // 1. 提升這裡沒問題 

{
  i = 0; 
  console.log(i); // 2. 目前是 0
  i = i++; //3. for 裡面對這個i 進行了 ++ 的操作
  (i < 3) ? break : continue;
} 
{
  console.log(i); // 4. 目前是 1
  i = i++; //5. for 裡面對這個i 進行了 ++ 的操作
  (i < 3) ? break : continue;
} 
{
  console.log(i);  // 6. 目前是 2
  i = i++; //7. for 裡面對這個i 進行了 ++ 的操作
  (i < 3) ? break : continue; // 8. break 觸發
} 

// 9. 此時全域的i 已經被加了3次，所以 i 目前是 3
```

所以我們在處理for的時候，會希望用let來處理，避免污染上層的變數：
```javascript 
for (let i = 0; i < 3; i++ ){}
console.log(i); //  i is not defined
```

用`var`會被hoisted，我們如果這樣做呢？
```javascript 
for (var i = 0; i < 3; i++ ){
  console.log(i);
  let i = 3;
}
```

猜猜看會有什麼反應，其實此時 `var` 跟 `let` 宣告的 i 已經不是同一個了，而因為進入到for 範疇裡形成了 block scope，所以結果會是 `Cannot access 'i' before initialization`。 分解圖：

```javascript 
var i; // 1. 提升這裡沒問題 

{
  i = 0; // 2. 設定 var 定義的 i 為 0
  console.log(i); // 3. javascript 引擎開始尋找 block scope的i，發現 i 下面，直接噴錯！
  let i = 3;  // 5. 這段不會執行
  i++; // 6. 這段不會執行
} 
// 7. 後續loop不會執行
```

所以這裡很有趣的是，block scope裡若有 let 宣告，就會停止運算報錯，但global i 卻被污染了。若 block scope 裡的 `let` 改成 `var` 又會有不同的結果。

```javascript 
for (var i = 0; i < 3; i++ ){
  console.log(i);
  var i = 3;
}
```

這裡的答案就很明確，因為hoisting的關係，i 在第一個loop就被修改成3，所以執行一次的迴圈。

所以都使用 `let` 就可以避免類似奇怪的問題嗎？ 未必...

```javascript 
for (let i = 0; i < 3; i++ ){
  let i = 3;
  console.log(i); // how many times to be executed
}
```

這裡會有兩個問題，第一個問題是 i 會執行幾次， 二是 i 會印出什麼？

答案是：執行 3 次且都印 3 ...

> WTF??!!

第一次看這個案例的時候我是滿頭問號，for 裡面彷彿有兩層block scope，一個是 loop scope （也就是迴圈執行次數的範疇），一個就是 block scope （迴圈執行內的範疇）。

我自己是不知道有沒有官方的對於這樣的說法，但至少從我看來，我覺得他是蠻奇怪的，如果使用 `let` 會避免提升的問題，那在這迴圈中宣告同樣的變數名稱應該要噴錯才對...

我自己理解的解構圖(不知道是否真的對或錯)：

```javascript 
// for start (在執行迴圈的第一次時)
{
  let i = 0; //(loop scope i) 1. loop scope 下的 i 為 0
} 
{
  //(執行第一次)
  let i = 3; // (block scope i) 2. block scope 下的i 為 3
  console.log(i); // 3. 印 3
}
{
  i = i++; //  (loop scope i) 4. 第一迴圈結束，把 loopscope i + 1;
  (i < 3) ? break : continue; 
}
{
  //(執行第二次)
  let i = 3; // (block scope i) 5. block scope 下的i 為 3
  console.log(i); // 6. 印 3
}
{
  i = i++; //  (loop scope i) 7. 第二迴圈結束，把 loopscope i + 1;
  (i < 3) ? break : continue; 
}
{
  //(執行第三次)
  let i = 3; // (block scope i) 8. block scope 下的i 為 3
  console.log(i); // 9. 印 3
}
{
  i = i++; //  (loop scope i) 10. 第三迴圈結束，把 loopscope i + 1;
  (i < 3) ? break : continue; // 11. break 邏輯觸發
}
```

看到這裡我只能說，不太懂JavaScript的運作邏輯，雖然這樣的設計可能有一個合理的解釋，但我還是覺得，與其設計的那麼複雜，不然就好好提供一個不錯的spec讓大家可以仔細的閱讀即可，不然太多的狀況下，其實要寫出一個沒有蟲蟲的JavaScript還真的滿難的。

如果有大大了解或認為我在解說上有些錯誤的部分，歡迎大家提出自己的意見...
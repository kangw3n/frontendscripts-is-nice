# JavaScript 真的太多東西要學了

因JavaScript的弱型別的關係，所以延伸出非常多的概念，例如Prototype Inheritance，First Class Function, Functional JavaScript等，這裡我先把自己了解的幾個觀念先記錄下來，希望能夠以淺寫易懂的概念讓大家知道。

---
## 你可能不知道的Scope
剛踏入JavaScript時候，對變數的宣告範疇 (Declaration Scope) 都會有些模糊，尤其是當 `hoisting` 的概念被啟動的時候，會讓變數的生命週期變得無法捉摸。

Scoping是什麼？簡單來說，程式的變數在宣告後會處在自己的範疇下，在範疇外的都~~不會~~被取得。在JavaScript裡的範疇可分為: 
1. Lexical Scope 
2. Global Scope 
3. Block Scope 
4. Function Scope 
5. Module Scope

Lexical範疇可算是JavaScript的核心，即內部區塊（如:Block Scope）的可以取得外部區塊的變數，內部可選擇性封閉自己的變數取得性。其他範疇則語義上較直接，這裡就不會簡介這些Scope的差別，有興趣的可以參考 [2018鐵人賽文章](https://ithelp.ithome.com.tw/articles/10194745) 。

Module Scope 是在 `<script type="module"></script>` 出現後的一種新的範疇，他有自己的封閉式範疇，等等會提出一些例子來說明這部分。

接下來就是重頭戲，其實也就是一些簡單的例子：

### 1. 學hoisting過程讓你覺得怪怪的嗎：`global context` vs `global scope`
先來看看簡單的code， （以下code都在全域層撰寫）：

```javascript 
var one = 1;
const two = '1';
let three = true;

console.log(one); // 1
console.log(two); // '1'
console.log(three); // true
```
我們用了JavaScript簡單的三種宣告變數方式，印出來也是在意料內的結果。（如果這裡你看不懂那建議來點基礎JS課程喲～）。 請記得在這寫例子中，我們都是在全域下範疇宣告這些變數，所以 `one, two, three` 基本上在全域的範疇下都可以簡易取得。

在JavaScript的物件結構裡，全域變數就是 `self` , 在瀏覽器環境下即是 `window` ； node 環境下即是 `global` 。

> 💡 `globalThis` 既可以不分環境下取得全域物件，但可惜目前的支援度不高。

```javascript
var one = 1;
const two = '1';

console.log(self.one) // 1
console.log(self.two) // undefined!!!
```
在第一個例子中， `two` 感覺已經是在global物件裡，為何我們不能用 `console.log(self.two)` 取得呢？原因是使用了 `const` 。

這裡有個概念需要釐清：所謂的全域提升，也就是透過 `var` 宣告做hoisting的動作，它即把變數 `one` 帶到了全域範疇 `global scope`， 但在ECMAScript規範裡  `const` 和 `let` 的宣告方式，他們只存在宣告當下的 `lexical scope` 語彙範疇裡：

> 💡 The variables are created when their containing Lexical Environment is instantiated but may not be accessed in any way until the variable’s LexicalBinding is evaluated.

以上的例子中，`two` 宣告在 `global context` 下，並非觸發任何的提升帶到 `global scope`。

用 `console.log(two)` 是在 `global context` 下尋找 `two` 的變數是可行的，但 `console.log(self.two)` 是直接在 `global scope` 尋找 `two` 的屬性，自然就無法搜尋。

> 💡 物件裡不存在的屬性即跳 `undefined` 非 `Throw Error` 錯誤。

這裡也可以提出一個JavaScript裡變數詭異的地方，在 `global context` 下直接賦值和宣告是會有不一樣的物件 configurable 特性 (詳見[Object.defineProperty()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)說明)。 請看以下兩個例子：

```javascript
one = 1; // assign a property to global scope
console.log(one); // 1
console.log(self.one); // 1
```
```javascript
var one = 1; // Define a global context variable and assign a property to global scope
console.log(one); // 1
console.log(self.one); // 1
```

這兩個寫法乍看之下基本上是同樣的結果，但在JavaScript的引擎處理上是會有不一樣的結果：
```javascript
one = 1; // assign a property to global scope
console.log(one); // 1
console.log(self.one); // 1

delete one;
console.log(one); // one is not defined
console.log(self.one); // undefined
```
```javascript
var one = 1; // Define a global context variable and assign a property to global scope
console.log(one); // 1
console.log(self.one); // 1

delete one;
console.log(one); // 1
console.log(self.one); // 1
```

可以看到用賦值的方式是可以刪除屬性 `one` 的，但在使用變數宣告方式是無法刪除變數 `one` 的，原因是因為變數宣告所產生的物件裡的物件屬性特質是無法被修改的 `{configurable: false}`，即使直接刪除self內的值也是得到一樣的結果：

```javascript
var one = 1; // Define a global context variable and assign a property to global scope
console.log(one); // 1
console.log(self.one); // 1

delete self.one;
console.log(one); // 1
console.log(self.one); // 1
```

> ❗❗ 這就是為何從其他語言來的工程師會不太在意care在 JavaScript 使用 `var` 宣告變數，乍看下蠻正常的，實際上他們是有差別的。

`var` 也有一個神奇的地方是，~~~ 沒賦值的變數，並不絕對等於 `undefined` ~~~，試看以下例子：

```javascript 
var one;
console.log(one); // undefined;

var one = 1;
console.log(one); // 1；

var one;
console.log(one); // 1;
```

或許某些人會覺得這蠻正常的，因為賦予 `1` 以後，後續沒賦值當然不會導致 `undefined`。但正常寫程式的時候會碰到在第1行的時候用了 `one` 變數，但過幾天後在撰寫第124行的 `block scope` 重複的使用了 `one` 變數，就或許會導致其他的問題，如以下：

```javascript
var one = 1; 

//... omit some code block and somewhere around line 124:
var blockState = true; // may true or false
if (true) {
  var one; // redefined `one` in a block scope and without assign any value for it.

  if (blockState) { 
    one = 2;
    console.log(one); // if condition apply, one === 2
  } else {
    console.log(one); // if condition not apply, one === 1 not `undefined`
  }
}

//... omit some code block and somewhere around line 1111:
console.log(one) // (blockState) ? 2 : 1;
```

以上的例子是，當在 `block scope` 裡宣告了你以為沒宣告過的相同 `one` 變數，一樣會hoisted到上方，以這例子中會出現的程式蟲蟲是：

> 當 `blockState` 條件不成立時，原本預計第二個log結果會是 `undefined` 卻變成了 `1`；  
> 當 `blockState` 條件成立時，原本預計第三個log結果會是 `1` 卻變成了 `2`。

使用 `var` 在 `block scope` 是會有無法預期的結果，讓我們再看看其他 `block scope` 導致的問題。


### 2. hoisting的處理方式 - Evil in block scope
以下例子讓大家複習一下提升的概念， 以下例子利用了 `block scope` 這個小惡魔範疇做例子的介紹：

```javascript 
var one = 1;

{var one = 2}; // block scope裡設 `2` ，並且透過hoisting修改了上層 `one` 變數
console.log(one); // 2

{const one = 3}; // 並沒提升，3只存在此block scope中
console.log(one); // 2

{
  var one = 4; 
  const one = 4
}; // Throw Error!! block scope裡出現了重複宣告而跳錯

{one: 5} // 利用物件並不會改變上層的值
console.log(one); // 2

```
上面的例子其實也蠻簡單理解的，block scope 並不會保護透過 `var` 宣告的變數，而會自己提升到上一層（即: global)。

> ❗❗ 這就是我們在使用 `for` 迴圈時候需要避免用 `var`的原因。

函式呢？

```javascript
console.log(myFn()); // log from myFn
function myFn() {console.log('log from myFn')}

console.log(myOtherFn); // undefined 因hoisting產生了
var myOtherFn = function() {console.log('log from myOtherFn')}

console.log(myNiceFn()); // myNiceFn is not a function  這時候myNiceFn還是undefined
var myNiceFn = function() {console.log('log from myNiceFn')}
```
上述的例子中也是初學者最懊惱的部分，第一個例子用了 `Function Declaration` 宣告而第二個用了 `Function Expressions`宣告則有不同的結果。

那函式如果在block scope 玩玩看呢？

```javascript 
var one = 1;

{
  one = 4, 
  function otherFn(){ return 'ghost function'}; 
};
console.log(one); // 4 
console.log(otherFn()); // otherFn is not Defined;
```
WHY？依前兩個例子，為何在block scope中這個Fn 不會變成global呢？ 原因在邪惡的 `，`逗點；

逗點在JavaScript裡是做評估 `evaluate` 的動作，也就是把左邊開始的statement做評估，所以將逗點改成 `;` 分號即得到正常的結果。

> 💡 這就是我們在宣告變數的時候盡量避免用逗點做區隔的原因，因為他不會給你任何的錯誤
> 
> ❗ 但值得一提的是，`one` 前面加了宣告變數後就會報錯了 （var one = 4, function....）

所以要真正提升上去的寫法會是:

```javascript
var one = 1;

{
  one = 5, // 用逗點是ok的
  otherFn = function(){return one};
}; 

console.log(otherFn());  // 5
```

用 `const` 一樣有防止提升的問題

```javascript 
{
  const one = 55, 
  otherFn = function(){return one}; 
  console.log('scoped', otherFn()); // scoped 55
}; 

console.log(one, otherFn());  // one, otherFn is not defined;
```

還有一個有趣的JavaScript議題： `closure`

```javascript
let one = 1;

{
  const one = 5; // 用逗點是ok的
  var otherFn = function(){return one};
  console.log(otherFn()); // 5
}; 

console.log(otherFn());  // 5
```
有些人可能會困惑，使用 `var` 會導致提升的問題，既然 `otherFn` 被提升到外層讓它可以被執行，那在全域範疇下執行 `otherFn` ，為何不是取得 `1` 而是 `5`...？那不是 magic ， 而是 `closure` 在作怪了，這篇章就不討論 `closure` 了。


> ⌛ block scope 在使用 `var` 宣告下是不會保護自己的變數... kind of....

> ⌚ `try{}catch(){}` 和 `with` 是例外嗎... kind of....

### 3. with 也沒那麼例外... 當你用 `var` 的時候什麼都會發生

`with` 是JavaScript非常古老的語法，現在應該也沒人再使用它。相關使用方式可參考: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/with 

他基本上有點像函式的範疇： 

```javascript 
var two = 2;

with({two: 22}) {
  console.log(two); // 取得帶入物件的 22
} 

console.log(two); // 2
```

這裡就是基本的函式範疇， `22` 值只存在 with 區域內，即使修改裡面的 `two` 值也不會影響外層的 `two `。

```javascript 
var two = 2;

with({two: 22}) {
  two = 222;
  console.log(two); // 222
} 

console.log(two); // 2 wow 他沒改變外面的two
```

但當沒有帶入任何值的時候，他一樣還是會改變外面的值：

```javascript 
const one = 1; 
var two = 2;

with({}) {
  var two = 222;
  var three = 333;
  let five = 555;
  console.log(two); // 223
  console.log(three); // 333 
  console.log(five); // 555
} 

console.log(two); // 222 被修改咯
console.log(three); //333 被提升咯
console.log(five); // nah 沒這個東西

with({}) {
  var one = 111;
  console.log(one); // one has been declared
} 
```

所以從這幾個例子很明顯可以看到，能不要用 `var` 就不用它, 請用 `let` 和 `const`
。

但這裡有個有趣的例子：

```javascript 
const two = 2;

with({two, three, five} = self) {
  two = 222;
  three = 333;
  five = 555;
  console.log(two); // 223
  console.log(three); // 333
  console.log(five); // 555
} 
```
還記得我們之前用 `const` 來宣告不會對 self 進行任何改變嗎？

這時候我們把global物件結構出來，猜猜看執行會有是什麼？ 答案是... ` Assignment to constant variable.` 錯誤。

從 `self` 結構出來的值，要賦予two本身的值居然不被允許，這也太奇葩了吧 🤷‍♂️？

### 4. 來點潮的 `type="module"`

模組JavaScript已標準化一陣子了，但各瀏覽器的支援度還算蠻一般的，它也有自己的scoping 機制，基本上每個 module 都是封閉式的範疇。

```html
<script>
  var one = '1';
  const two = '2';
</script>

<script type="module">
  console.log(one); // '1'
  console.log(self.one); // '1'
  console.log(two); // '2'
  console.log(self.two); // undefined
</script>
```

上面的例子中跟前面的例子是一樣的，但如果把它改成兩個都是 `module` 的時候就會有不一樣的結果。

```html
<script type="module">
  var one = '1';
  const two = '2';
</script>

<script type="module">
  console.log(one); // Error 因為還沒被宣告
</script>

<script>
  const one = '2'
  console.log(two); // Error 因為還沒被宣告
  console.log(self.two); // undefined 因為在window找不到two的屬性
</script>

<script type="module">
  console.log(one); // '2'
  console.log(self.one); // undefined 
</script>


```

> 💡 總的來說：`module` 會將自己的範疇封閉起來，其他的地方即無法取得。

所以在scoping的議題中，其實有很多非常雕磚的小蟲蟲，即使寫了很久的工程師也未必能夠第一時間解析目前的範疇是什麼，需要了解他們之間的關係是什麼，使用的變數宣告方式等，所以說 

> 🤦‍♂️ 工程師不是人幹的...

---




## Composition 就是 Vue3.0的核心啊 --- todo

## Memoization in JavaScript Js  --- todo

## First Class Function 在程式語言裡面特有的公民權 --- todo
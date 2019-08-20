# 前端正在笑你無能 - Performance Check

這系列主要探討前端的效能，到底需要顧慮效能面還是功能面，一直是前端領域很懊惱折騰的事情。此系列是為了讓工程師能夠快速了解前端該注意的效能面有哪些，以及它的運作原理等。

因此篇有太多與硬體相關的技術面，本人的技能樹有限，難免會有資訊錯誤的部分，歡迎大家提出修改。

## 淺淺談談 JavaScript 的記憶體 
身為前端工程師，應該不太計較JavaScript引擎是怎麼處理記憶體分配 `（Memory Allocation）` 的部分，不同的JavaScript引擎在處理記憶體的部分也略有不同。因有一次上了某知名老師的課，跟老師交流後發現與自己了解的記憶體分配有差，所以稍微深入研究了一下Chrome V8 引擎處理方式，把了解的記錄下來。

若

在JavaScript裡，在開始產生物件時（此`物件`非單純指JavaScript裡的物件）；包含 `object` , `string` , `array` 等都會被賦予記憶體空間，然而因為有 `Garbage Collection` （後續簡稱 GC）機制，這些記憶體將會 `自動` 被清除，也就是開發者不需要特別去將原本被佔據的記憶體做手動的清除。

### 記憶體的生命週期 LifeCycle
雖然說透過GC後，記憶體會`自動`釋放，但這不代表開發者不會遇到記憶體問題，例如它在某種狀況太自動化了，開發者預計會釋放的記憶體卻沒被釋放等，導致記憶體洩漏 `Memory Leak` 的問題。

要避免這問題，我們首先必須了解，大致上都有這三個階段：

> <center> 賦予記憶體 > 使用記憶體 > 釋放記憶體 </center>

1. 賦予記憶體 `（Allocation）` ：宣告變數或賦予初始化值、函式呼叫式 `let date = new Date();`，即被賦予記憶體空間
2. 使用記憶體 `（Use）` ： 指讀取/使用（**Read**）它或修改/寫入（**Write**）變數、或在函式中帶入參數。
3. 釋放記憶體 `（Release）` ： 當變數或屬性沒有被使用、指向 `（pointing）` 、 參考 `（reference）` 時，即透過 GC 把賦予的記憶體位置給釋放。

> 💡 document.getElementById('id') - 使用 `document` 物件裡的方法也屬賦予記憶體的階段 

> ⚠️ 在主流程式語言中，欲釋放記憶體，可透過賦予 `null` 或 `delete obj.myProperty` 即可達到釋放記憶體的效果，但在JavaScript裡這並非絕對，等等在說 `參考` 時會再細說。

```javascript
let a = '1'; //記憶體位置 0xff11
let b = '2'; //記憶體位置 0xff22
let c = {a: 1}; //記憶體位置 0xff33
let d = c; //指向同一個記憶體位置 0xff33
a = b; // 記憶體位置 0xff11 因沒有使用而被釋放，但同時建立了 0xff44 記憶體位置
```
上述例子中，因 `b`裡的值 `2` 賦予 `a`, 使用了`call by value` 的參數傳遞方式拷貝一份新的 `2`，因此原本的 `1` 所佔用的記憶體沒被使用後即被釋放出來。在這系列我只針對記憶體管理而不會細談JavaScript的參數傳遞方式，有興趣的可以參考胡立撰寫的 [深入探討 JavaScript 中的參數傳遞：call by value 還是 reference？](https://blog.techbridge.cc/2018/06/23/javascript-call-by-value-or-reference/)。

由此可見，在記憶體的生命週期三個階段中，釋放記憶體會是整個記憶體管理過程中最複雜的一面。因GC機制，JavaScript引擎需要知道哪些變數是沒被指向/參考或是後續不會再用到的，才會將它給釋放出來。接下來來看看GC是怎麼判定記憶體該釋放與否。

### `Garbage Collection` 演算法
GC要怎麼判定記憶體是否該存在，依靠以下兩種演算法：
1. Reference-counting garbage collection 垃圾回收參考計數
2. Mark-and-sweep algorithm 標記和清理演算法

> 💡 以上中譯取自MDN中文版，文字上的確看不懂在幹嘛，下面會大概解釋一下英文字面上的意義。
#### Reference-counting garbage collection
意義上是指：`透過參考數（References availability）的計算來判定這記憶體是否該被釋放`；在JavaScript記憶體管理概念中，其中一個蠻重要的觀念是物件的指向/參考 `（Reference)` ；該物件是否有被某變數參考：

```javascript
let a = { user: 'kangw3n' }; //記憶體立即被建立
```

上述的例子建立了一個物件屬性 `user` 和值為 `kangw3n` 並賦予 `a` 這個變數上。我們可以理解成：此時該物件 `{ user: 'kangw3n' }` 物件被 `指向` 到變數 `a` 上。

```javascript
a = { user: 'kangw3n2' };
```

接下來我們把 `a` 的值賦予其他值 `{ user: 'kangw3n2' }` , 注意這裡是用覆蓋物件的方式，而不是 `a.user = 'kangw3n2'` 的方式修改。此時某個指向消失了，那就是 `{ user: 'kangw3n' }` 物件。
 
當物件沒有被任何物件給指向/參考時，該物件屬於"可以被回收的垃圾" `garbage collectible`，即等待被回收的物件，以上面的例子物件 `{ user: 'kangw3n2' }` 記憶體位置即成為"可以被回收的垃圾"。

從兩行程式可以看到，因 `{ user: 'kangw3n2' }` 是無法到達 `（unreachable）` 的指向，所以在`垃圾回收參考計數`機制的認知上沒有參考的物件即可以被回收釋放了。

`垃圾回收參考計數`就是依據這邏輯在運行，但是在複雜一些的例子或許會有玄機，往下再看一個例子，若物件被參考了呢？
```javascript
let a = { user: 'kangw3n'};
let b = a;
```
此時物件 `{ user: 'kangw3n'}` 物件被 `指向` 到 `a` 上，`b` 也參考了 `a` 。因JavaScript物件是參考型別 `（references type）` ，所以JavaScript不會針對 `b` 開啟新的一個物件記憶體，即把 `{ user: 'kangw3n' }` 指向到  `a` 和 `b` 變數上。

若將 `a` 內的屬性值修改：

```javascript
a.user: 'kangw3n2';
```
此時 `a` 和 `b` 的值皆為 `{ user: 'kangw3n2' }`，因為他們都指向同一個參考物件，所以利用屬性賦值 `Dot Notation` 方式修改不會讓 JavaScript引擎產生第二個記憶體空間，也不會因為沒有指向而回收任何垃圾。

但在JavaScript裡面奇怪的事情蠻多的：
```javascript
a = { user: 'kangw3n3' };
```
原先 `b` 參考了 `a`， 當時的 `b` 為 `{ user: 'kangw3n2' }`，當修改整個 `a` 指向的物件，或許有人會認為 `{ user: 'kangw3n2'}` 這個值因為沒有被指向了，所以該被回收。但實際上並不是這樣，我們用下列程式碼解釋一下：
```javascript
let a = { user: 'kangw3n' };
let b = a;

a.user = 'kangw3n2'; // a = b = { user: 'kangw3n2' }

a = { user: 'kangw3n3' }; 
// 當執行這段code的時候，JavaScript引擎回去搜尋原本的屬於a的 { user: 'kangw3n2' } 有沒有被其他變數給參考，目前我們知道 b 是參考 a 的，所以必須保有原本的 { user: 'kangw3n2' } 賦予 b 並開啟一個新的記憶體， a 則變成 { user: 'kangw3n3' } 隸屬於另一記憶體空間
```

最後若我們把 `b` 的值也指向 `a` 的話，原本的 `{ user: 'kangw3n2' }` 因沒被指向或參考將被`垃圾回收參考計數`機制認定為"可以被回收的垃圾"。

##### 互相參考產生的迴圈是沒辦法被回收的。
```javascript
function cycle() {
  var a = {};
  var b = {};
  b.a = a;
  a.b = b;
  return {a, b}
}
cycle();
```
`垃圾回收參考計數`機制有個缺陷是，當物件含有互相參考的迴圈，它們是無法立即被回收釋放的。

上面的例子中當 `cycle()` 被執行的後，因為 `a` 和 `b` 互相成為了對方的參考物件，雖然 `cycle()` 結束後本應將它們兩個標記為"可以被回收的垃圾"，但因為`垃圾回收參考計數`機制的原理上被參考的物件是沒辦法將它們回收的，此時就會形成JavaScript裡的記憶體洩漏，所以比較好的方式是做一個物件拷貝的動作，即把 `b.a = Object.assign({}, a)` 方式賦予 `b.a`，我們在這系列不會討論拷貝這部分，但這可以解決參考所導致的`垃圾回收參考計數`的缺陷。

以上演算法為`垃圾回收參考計數`，簡而言之就是有指向就不回收，沒有指向就認定為是垃圾。

#### Mark and Sweep 標記和清理演算法

為了解決`垃圾回收參考計數`機制的缺陷，`標記和清理演算法`使用了另一套邏輯：即尋找物件是否可到達 (reachable)。

`標記和清理演算法`將經過三個步驟：

1. 尋找環境中的根目錄 `（Roots）` ，在JavaScript的世界裡，根目錄是`全域變數`， 在瀏覽器環境是 `window` 物件 而在Node環境則是 `global` 物件。
2. 演算法先檢視根目錄和所有的子階層屬性，並將它們標識成 `active` 即非"可以被回收的垃圾"，相反沒辦法到達的物件節點則標記為"可以被回收的垃圾"。
3. 將被標記為"可以被回收的垃圾"的節點物件清除，把記憶體釋放出來。

![Image of Mark and Sweep](https://miro.medium.com/max/700/1*WVtok3BV0NgU95mpxk9CNg.gif)  
*引用了sessionstack文章內的圖*

參照以下的例子：
```javascript
var a = { a: 1 };
```
我們在瀏覽器 F12開發環境下執行這段code，在沒用任何立即執行函式(IIFE)封包程式碼，`a` 變數將被賦予 `window` 物件中。

> 上述例子使用 `var` 宣告會被hoisted，即到了 `window.a` 上，在這狀況下 屬性屬性/變數 `a` 就不會被回收，因為 `window` 物件是個必須存在的物件。可參考 hoisting (提升) 和 var, let,  const 相關說明。

```javascript
a = null;
```

此時將 `null` 賦予 global 的 `a`， 原本被指向的物件 `{a: 1}` 將會從原本的節點移除， `標記和清理演算法`將從 window 根目錄開始做掃描，若 `{a: 1}` 沒被其他變數或屬性參考，此物件將會被回收。

> 從 2012年開始， JavaScript引擎已開始採用`標記和清理演算法`。之前說的 `cycle()` 的問題既可得到解決，因為互相參考的問題不會被根目錄給記載，但此演算法的缺點是必須要讓欲回收的變數無法被根目錄給記載才能將它回收。

但總結是，這些回收釋放機制都是由JavaScript引擎決定，我們是無法透過JavaScript語法觸發任何的GC機制。

### 記憶體回收該在什麼時候運作，從V8引擎看看

這些演算法在什麼時候運作決定於瀏覽器引擎？GC不可能在瀏覽器高速運作時頻繁執行GC機制，JavaScript引擎基本上會經過以下三點進行GC優化的動作：

1. Generational collection （世代集合）： 可以理解成`解析世代的產物`，將物件分為： `新物件` 和 `舊物件`。當某些物件出現的頻率高，執行速度較短且很快就脫離節點，它們即是 `新物件` ，引擎會快速檢測並回收；當那些物件存留的時間很長，被套用的間距過長，長久下來會被引擎定義為 `舊物件`，引擎將減少對這類型的物件進行檢測，減少不必要的效能。
2. Incremental collection （遞增集合）：當今天物件的量是多的，且節點數是高的，引擎會試圖將物件拆解處理，將相關的節點拆成區塊式分析，減少物件過大所導致的效能的過度利用，再把它們接成同一節點進行分析，從中把不必要的節點回收。
3. Idle-time collection （閒置集合）： 引擎會試著在 CPU 閒置期進行 GC， 減少在高運作時候的執行量。

GC機制有幾個重點，整理一下給大家：
1. 它是沒辦法手動觸發的，我們沒辦法控制或阻止它的運作。
2. 只要該 `物件` 是可以被節點取的 / 到達 `（reachable）` 時，即會保留它的記憶體空間且不會被回收釋放。
3. 可被參考 `（referenced ）` 不代表可被節點取的 / 到達 `（reachable）`，可被參考的物件也有被回收釋放的可能。

GC機制是一個非常複雜且龐大的議題，每個瀏覽器引擎擁有自己處理回收機制的方式，有興趣的人可以研究一下 [Chrome V8引擎處理GC的方式](http://jayconrod.com/posts/55/a-tour-of-v8-garbage-collection)。


### JavaScript的 `Stack` 與 `Heap`
在程式語言世界裡，處理/配置記憶體的方式大致分為一下兩種分法：

1. `Stack` （堆疊）: 由瀏覽器/編譯器自行管理的記憶體位置，它擁有著堆疊的概念，也就是一層層的記憶體，沒被使用的記憶體被回收就會從堆疊中pop出來，它擁有著 `LIFO` （後進先出）的概念：想像成影印機上的一疊紙，第一個被處理的會是最上層的紙。
2. `Heap` （堆積）：在記憶體中無結構的大區域，它們是分散的並無堆疊的概念，在其他語言中它是需要使用者自行回收的，在GC的概念下，它還是會被系統所管理。

那在JavaScript世界裡，什麼東西是存在 `Stack` 什麼東西是存在 `Heap` 呢？ 

說真的我看了好多文章跟V8引擎的文件，它並沒有明確的告訴什麼東西該存到哪個記憶體類類型，而在ECMAScript並沒有規範記憶體配置的部分，所有記憶體機制都由Javascript引擎決定。

所以我的理解是：若JavaScript裡面所有東西都是物件的話，那就是存在 `Heap`；因JavaScript是個動態語言，當他是個動態型別時，弱型別導致它存放在 `Stack` 的意義就沒了，因為型別常被修改的狀況下，要從一堆紙中尋找該記憶體位置再修改它就變得太沒效率了。以我們上面討論的GC機制，的確存在 `Heap` 會是比較好被回收釋放掉。

但JavaScript有事件執行緒的機制，也就是在某個時間點呼叫了什麼函式，而這函式是需要呼叫多個函式做堆疊的動作，他們就會有先後順序的問題，這時候就函式本身就會組成 `stack` 。想要更了解JavaScript的執行緒可以參考 MDN 的 [Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop) 文章或是 Philip Roberts 在2014年 JSConf的Event Loop[演講](https://www.youtube.com/watch?v=8aGhZQkoFbQ)。

> Event Loop內提到的 `heap` 跟 `stack` 與JavaScript是否使用哪種記憶體存取資料或許沒有直接關係。



### 記憶體洩漏 Memory Leak in JavaScript
接下來我們來探討在JavaScript裡面會發生的記憶體洩漏問題。什麼是記憶體洩漏，當你的瀏覽器非常當的時候大概有87%都是記憶體洩漏導致的。沒在用的變數但卻被存取而`沒辦法`被回收釋放，通常都是因為某些撰寫JavaScript的壞習慣：

#### 不必要的全域變數

依據剛剛`標記和清理演算法`中探討的，當我們不小把變數值賦予 `window` 根目錄下，除非你將它設成 `null`，不然這記憶體位置永遠都不會被回收釋放。有時候是無意的，因沒注意而不小心設成全域變數，例如以下的撰寫：

```javascript 
function setLocalState(data) {
  state = data;
}
```
這裡的 `state` 就不小心變成了 `window.state` 了，意外的變成了全域變數。

> 💡 JavaScript裡有個 `immutable` 概念，即盡量不要直接修改參數或變數的值，以上的例子改成 `pure function` 也可以避免記憶體洩漏的問題

> 💡 在JavaScript有個小技巧可以避免不小心設定全域變數的問題：設定 `use-strict;` ；啟用嚴格模式解析 JavaScript，可防止不小心污染全域變數裡的值。

另外還有跟 `this` 執行區域 `context` 有關，如：
```javascript 
function setLocalState(data) {
  this.state = data;
}
setLocalState('global'); // window.state = 'global';

const a = {
  state: null,
  setLocalState: setLocalState
}
a.setLocalState('local'); // window.state = undefined;
```
所以在使用 `this` 時候要小心執行的區域，全域變數不會被回收，除非重新賦予該值或改成 `null`，才能準確的回收不必要的物件。
```javascript
window.state = null;
```
> 💡 使用 `(iife)()` 立即執行函式也可以避免全域變數的污染

#### Clousure
JavaScript裡常用的閉包模式，也可以導致記憶體洩漏，我這裡不細說閉包的運作原理和方式，簡而言之閉包就是記載並存取某個變數而讓這變數範疇是封閉在一個環境中。

其實我覺得閉包是一個最直接讓記憶體無法釋放的撰寫模式，因為記載了某個封閉式的變數，這變數必須要透過深度解析才能知道後續的程式是否有使用或參考，而開發者也容易在無意中撰寫了導致記憶體洩漏的閉包程式。

```javascript
var theThing = null;

var replaceThing = function () {
  var originalThing = theThing;
  var unused = function () {
    if (originalThing) console.log('hi');
  };
  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
        console.log('message');
    }
  };
};

replaceThing();
```
`replaceThing` 執行完後，記憶體無法回收，這個範例比較複雜，我們拆解解釋好了：

1. 當 `replaceThing` 被呼叫後，`theThing` 會取得物件含 `longStr` 和 `someMethod()`。
2. `originalThing` 被 `unused` 閉包作用域給限制，因為第四行的 `originalThing = theThing` ，作用域已被共享，即使 `unused` 永遠不會被調用，但已被 `theThing` 所指向的閉包給參考了。
3. 後續如果沒有 `theThing` 設成 `null`， 閉包內的函式即使沒被使用都不會給GC機制回收。
4. 若之後再調用 `replaceThing` 函式， 閉包的函式只會變得更大，記憶體空間會往上提升。

```javascript
window.onload = function() {
  var obj = document.getElementById("element");
  obj.onclick = function leak() {}; // 這個是兇手
}
```
以上程式碼包含了閉包和循環指向參考：

1. `obj` 包含了 DOM 物件參考，但頁面中的 `<div id="element">button</div>` 也包含了物件 `obj` 的參考，形成了循環。
2. 函式內包含函式會形成閉包，當被調用的時候就會沒辦法得到釋放。

```javascript
window.onload = function() {
  var obj = document.getElementById("element");
  obj.onclick = leak;
}
function leak() {}

// or

window.onload = function() {
  (function() {
    var obj = document.getElementById("element");
    obj.onclick = leak;
  }());
  function leak() {}
}
```
函式定義在函式外或使用立即執行函式即可解除閉包。


#### 利用物件存取DOM標籤
有時候利用物件存取DOM節點會讓程式比較好管理，例如
```javascript
var allEls = {
  button: document.querySelector('button');
  div: document.querySelector('div');
};

function deleteButton() {
  document.body.removeChild(document.querySelector('button'));
}

deleteButton();
```

當 `deleteButton` 被執行後會刪除節點上的 button 按鈕，但 `allEls` 裡還有記載物件的參考，因此 `button` 標籤記憶體參考無法被回收釋放。

#### setInterval is `Evil`
計時器是程式裡常用的功能，但它某種程度也會被閉包的作用域給限制，看看以下例子：

```javascript
var myData = { a: 1 };

setInterval(function() {
  var element = document.getElementById('element');
  if(element) {
    element.innerHTML = JSON.stringify(myData));
  }
}, 1000);
```
1. 上述code中`element`若被刪除 ，計時器是不會停止的。
2. 因計時器沒停止，在函式內調用了 `myData` 值，所以即使 `myData` 之後沒被使用了，它仍然沒辦法被回收釋放。


#### 事件監聽 與 Component Based Framework
事件監聽也是一個常被遺忘且容易導致記憶體洩漏的原因之一。若標籤元素被定義某事件監聽 `（addEventListener）` 時，標籤元素若被移除候要記得移除原本的監聽 `（removeEventListener）`， 因為這例子比較簡單所以就不提供範例了。

回想現代框架都會運用元件化設計，我們可以針對元件進行事件監聽的動作，框架基本上會自動在元件銷毀時候將所運用的監聽給刪除，但往往在撰寫這類型的code時候會忽略原本的JavaScript監聽事件：

```html
<div class="container">
  <button class="length">Check Length</button>
  <div id="#app">
    <Todo v-for="todo in todos" />
  </div>
</div>

<script>
Vue.component('Todo', {
  template: '<button>{{name}} <span @click="removeTodo"> X </span></button>',
  methods: { /* 省略 */ },
  mounted() {
    document.querySelector('length').addEventListener('click', () => {
      alert('目前有' + this.$parent.todos.length  + ' Todo');
    });
  }
});

new Vue({
  el: '#app',
  data: {
    todos: [/* 哆哆哆哆哆哆哆 */]
  }
})
</script>
```
當 `<Todo />` 按下 `removeTodo` 時候，框架會幫我們把 `removeTodo` 的 click事件也一併移除，但是手動監聽執行的 `length` 則不會被移除，而且後續新增進來的 todo 將會覆蓋 N 層的 `call stack`。

> 雖然這案例比較極端也不可能會這麼寫，但單純描述忘了解除監聽這件事情足夠導致記憶體洩漏的問題。


//TODO
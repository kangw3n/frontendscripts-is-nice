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
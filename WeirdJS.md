# JavaScript 怪怪 der

把每天在寫Javascript遇到的怪問題都記錄下來，希望可以避免再犯。

### **❄️❄️ `Object.freeze` ❄️❄️ 好像不凍結了... **
  ```javascript
  let myObject = {
    a: 1,
    b: 2
  };
  ```
  先建立一個看起來很普通的物件，但這不好玩，我們用 `Object.defineProperty` 擴增新的屬性好了：
  ```javascript
  Object.defineProperty(myObject, 'c', {
    configurable: false,
    enumerable: true,
    get: () => this.value,
    set: (value) => {
      this.value = value
    }
  });
  ``` 
  用 `Object.defineProperty` 建立屬性的好處在於我們可以控制屬性的描述器（Descriptors）： `ACCESSOR` 和 `DATA` ，簡單來說一個是針對物件資料面一個是物件訪問面的一些設定，可參考MDN[文章](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)。這裡針對 `myObject` 增加了新屬性 `c`， 再用 `Accessor` 設定 `getter` 和 `setter` 的方式。  
  在還沒有 `proxy` 可用的年代，我們可以在setter做一個介入點，針對該屬性的改變而做某些事情，但這不是我要說的問題，我們再往下看看：

  ```javascript
  console.log(myObject.c) // undefined
  ```

  此時印出來是 `undefined` 沒什麼問題，後來後悔了，希望將 `myObject` 給凍結不讓其他人或其他程式修改，我們可以用 `Object.freeze` 把它給封死。 *`Object.seal()` 跟 `Object.preventExtensions()` 也可以防止物件做封裝或擴增。*

  ```javascript
  Object.freeze(myObject);
  console.log(myObject.c) // undefined
  ```
  這裡也沒什麼問題，`c` 本來就沒設定所以是 `undefined` ，但奇怪的來了

  ```javascript 
  myObject.c = 1;
  myObject.b = 33;
  console.log(myObject.a); // 1
  console.log(myObject.b); // 2 - 改不動！是原本的 2
  console.log(myObject.c); // 1 - 但它可以被修改！
  myObject.c = function(e) {return this.a};
  console.log(myObject.c); // function(e) {return this.a} - 再改～！
  myObject.c = 87;
  console.log(myObject.c); // 87 - 再改～！
  ```
  這時你可能以為 `Object.defineProperty` > `Object.freeze` ，那在凍結後的 `myObject` 可以在設定 `Object.definePropery` 嗎？

  ```javascript 
  Object.defineProperty(myObject, 'd', {
    configurable: false,
    enumerable: true,
    get: () => this.value,
    set: (value) => {
      this.value = value
    }
  });
  ```

  直接跳錯 `Cannot define property d, object is not extensible` 。這也是我們預想到的結果，但是問題來了，如果我真的要 freeze `c` 的屬性，是不是就沒辦法了呢？🤧🤧🤧
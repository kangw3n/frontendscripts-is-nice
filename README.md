# TypeScript 筆記 - 淺談TS裡的Type 

## 目錄

- [TypeScript 筆記 - 淺談TS裡的Type](#typescript-%e7%ad%86%e8%a8%98---%e6%b7%ba%e8%ab%87ts%e8%a3%a1%e7%9a%84type)
  - [目錄](#%e7%9b%ae%e9%8c%84)
  - [Why TypeScript](#why-typescript)
  - [在TypeScript裡的型別: Type](#%e5%9c%a8typescript%e8%a3%a1%e7%9a%84%e5%9e%8b%e5%88%a5-type)
  - [TypeScript的基本型別](#typescript%e7%9a%84%e5%9f%ba%e6%9c%ac%e5%9e%8b%e5%88%a5)
    - [Number](#number)
    - [String](#string)
    - [Boolean](#boolean)
    - [Enums](#enums)
    - [Void](#void)
    - [Null 和 Undefined](#null-%e5%92%8c-undefined)
    - [Any](#any)
    - [Array](#array)
    - [Object](#object)
    - [Tuple](#tuple)
    - [Never](#never)
  - [Interfaces - DuckTyping鴨子在叫](#interfaces---ducktyping%e9%b4%a8%e5%ad%90%e5%9c%a8%e5%8f%ab)
    - [DuckTyping在叫了...](#ducktyping%e5%9c%a8%e5%8f%ab%e4%ba%86)
      - [Type Assertion](#type-assertion)
      - [UnionSet](#unionset)
    - [函式裡的Interfaces](#%e5%87%bd%e5%bc%8f%e8%a3%a1%e7%9a%84interfaces)
  - [TS常遇到的坑](#ts%e5%b8%b8%e9%81%87%e5%88%b0%e7%9a%84%e5%9d%91)
    - [TypeAssertion不能亂用](#typeassertion%e4%b8%8d%e8%83%bd%e4%ba%82%e7%94%a8)
    - [關於預設值這件事](#%e9%97%9c%e6%96%bc%e9%a0%90%e8%a8%ad%e5%80%bc%e9%80%99%e4%bb%b6%e4%ba%8b)
    - [你以為的Union是真的那麼簡單嗎？](#%e4%bd%a0%e4%bb%a5%e7%82%ba%e7%9a%84union%e6%98%af%e7%9c%9f%e7%9a%84%e9%82%a3%e9%ba%bc%e7%b0%a1%e5%96%ae%e5%97%8e)
    - [Array.push() 是什麼，可以吃嗎？](#arraypush-%e6%98%af%e4%bb%80%e9%ba%bc%e5%8f%af%e4%bb%a5%e5%90%83%e5%97%8e)
    - [Never跟陣列是朋友嗎？](#never%e8%b7%9f%e9%99%a3%e5%88%97%e6%98%af%e6%9c%8b%e5%8f%8b%e5%97%8e)
    - [關於Type你又知道多少？](#%e9%97%9c%e6%96%bctype%e4%bd%a0%e5%8f%88%e7%9f%a5%e9%81%93%e5%a4%9a%e5%b0%91)
  - [~~不~~常用的TS功能](#%e4%b8%8d%e5%b8%b8%e7%94%a8%e7%9a%84ts%e5%8a%9f%e8%83%bd)
    - [keyof](#keyof)
    - [Readonly](#readonly)
    - [Partial](#partial)
    - [Required](#required)
    - [Pick](#pick)
    - [Omit](#omit)
    - [Generic](#generic)
    - [Method Overload 重載函式](#method-overload-%e9%87%8d%e8%bc%89%e5%87%bd%e5%bc%8f)
    - [待續](#%e5%be%85%e7%ba%8c)

---


## Why TypeScript

因應JavaScript的趨勢，包含三大框架 **（Angular, React, Vue）** 漸漸開始使用在TypeScript（以下簡稱TS）撰寫他們的函式庫，這篇簡單介紹一下常用的TS使用情境，相關使用方式可參考[官方網站](https://www.typescriptlang.org/samples/index.html)。 *本文部分使用TS3.0版本以上的語法*


## 在TypeScript裡的型別: Type

在TypeScript裡有以下N種宣告型別的方式：
```typescript
let declareType: number; // 宣告Number型別
let declareTypeByInitialize = 1; // 在初始化賦值決定型別

type UnionType = 'left' | 'right'
let declareByType: UnionType; // 使用Type宣告 

let initialType: number | string = 1; //宣告一個String || Number 變數
let baseOnTypeAssertion = <number> initialType // TypeAssertion的變數決定

interface MyObject {a: string}
let declareByInterface: MyObject = {a: 1};

// ...還有很多後續會帶，但最常用的為 declareType這類型，

```

## TypeScript的基本型別

### Number
在TS裡在，與JavaScript一樣，有別於其他語言有浮點數（float，int，double）等，在TS裡通通都是 `Number`，當然也支援十進制以外的數值。宣告如下：

```typescript
let decimalValue: number = 10;
let hexaValue: number = 0xf10b;
```

### String
在TS裡，與JavaScript一樣，欲宣告字串型別，在文字前後加上引號即可 - **單引號('') / 雙引號(" ")**
```typescript
let myStringType: string = 'string with single quotes';
let myStringType2: string = "string with double quote";
```
這裡可以特別補充一下，有別於其他強型別語言（如：Java），TS宣告型別是需要用**小寫**，即 `let myValue: number;` 非 `let myValue: Number` ， 大寫通常都是給自訂的Interface或Type命名使用，當然TS與JavaScript一樣也有一些保留文字，可以參考此[列表](https://github.com/Microsoft/TypeScript/issues/2536#issuecomment-87194347)，宣告的是否特別注意一下。


### Boolean
Boolean值簡單來說就是 `true` 跟 `false` 值，宣告方式：
```typescript
let isVote: boolean = true; 
let isFetch: boolean = !!0;  // false
let isParticipate: boolean = !!+"0"; // false
```
這裡可以TS特別提一下 `0` 和 `1`，某些程式或API會利用 0 或 1 替代 true 或 false，這裡用 `Double NOT (!!)` 做型別的轉換 `Truthy` 或 `Falsy`，如果值是字串時候則用 + （casting） 將字串轉換成數字。相關可參考 MDN 文章 (https://developer.mozilla.org/en-US/docs/Glossary/truthy)

### Enums
Enums 是個抽像型別，中文為 `列舉` 或是 `狀態機`，主要能定義一組固定的常數 `constants`，相關使用時機可以參閱[Enums](https://www.typescriptlang.org/docs/handbook/enums.html)官方說明。 
```typescript
enum GameStatus {
  GameStart = 1,
  GamePause = 2,
  GameStop = 3,
  GameOver = 4,
  GameLoading,
  GameWin,
}

function setGameStatus(name: string, status: GameStatus): void {}

setGameStatus('kangw3n', GameStatus.GameStart)；
```

### Void
Void型別通常都是利用在函式沒有回傳任何值的時候，例如設定狀態或是顯示訊息等，不需要從function取得任何的資料。通常都是用在Function，但也可以用在變數上，但基本上宣告void型別的變數只能賦予 `undefined` 或 `null` 值。

```typescript
function setMessage(message: string): void {
  alert(message);
}

setMessage('你TS真的好厲害歐～');

let declareSomeThingVoid: void;
declareSomeThingVoid = 1; // ERROR 1 不是void型別

```

或許比較常見的例子是 `<a href="javascript:void(0)"> ClickMe </a>` ，其實他們就是有著共同的目的，就是不會回傳任何值，相關 `void` 說明可參考 [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/void)。

### Null 和 Undefined
這兩種型別比較少在用，語義上和JavaScript是一樣的，但他們有個共同的特質是，`null` 和 `undefined`是其他型別的副類型 `(subtype)`。

```typescript
 let nullVal: null = null;
 let nullNumber: number = null; // Allowed
 let undefinedNumber: number = undefined; // Allowed
```

### Any
~~偷偷告訴你這是一個當你覺得TS很複雜很難但又不知道要放什麼型別的時候下它就對了~~，`any` 就是一個非常彈性的型別，它被賦予以上所有的型別都不會報錯，使用的時機通常在還不知道資料格式的時候，可以用 `any` 先來宣告，在 `C#` 裡面就是 `dynamic` keyword

```typescript
let dynamicData: any = 'iAmString'; // 雖然這裡賦予字串型別，但它還是 any 型別
dynamicData.concat(' and iLoveYou'); // 可以直接調用相關的方法
dynamicData = 1;
dynamicData = {};
dynamicData = [];
```

### Array
陣列就是陣列 (**~~詞窮~~**)，在TS裡可以有兩種方式宣告陣列：

```typescript
let myArray: number[] = [1, 2, 3];
let myStringArray: string[] = ['a', 'b', 'c'];
let myWhateverArray: any[] = ['1', 2, {a: 1}];
let myWhateverArray = ['1', 2, {a: 1}]; //當沒提供任何型別則初始化陣列時，此定義跟 any是類似，但初始化會變成tuple

let myGenericArry: Array<number> = [1, 2, 3]; //此方式跟myArray宣告方式是一樣的，在TS裡這種宣告方式為通用型別 `Generic Type `

```

### Object
物件就是物件 (**~~詞窮~~**)，但通常這種定義物件的方式在TS是沒什麼~~屁~~用的，等等在說明 `interface` 時候或許會是更好定義物件的方式。物件型別代表任何不是原始型別`(Non-Primitive Type)`的類別，如 `number`、`string` 、`boolean` 等。相關原始型別請參考[這篇](https://medium.com/@jobboy0101/js%E5%9F%BA%E7%A4%8E-primitive-type-v-s-object-types-f88f7c16f225)。

```typescript
function createUser(user: object): void {
  //...
}

createUser({firstName: 'TEST USER'}); // 只要參數是物件即可
createUser({email: 'fakepeople@fakeemail.com', firstName: 'w3n', url: 'XX'}); // 物件長度有多少根本不重要
createUser(new Function); //咦～搞笑的還真的可以 
```

以最後一個例子而言，只要是物件型別的都可以被帶進去，之後討論 `interface` 時候可以明確將物件定義清楚。

### Tuple
Tuple就是一組資料，一組不可更變的資料, 通常運用在陣列上，當你有明確的型別格式和絕對的長度 `length` 時候可使用。

```typescript
let taiwanYear: [number, string] = [108, '2019']; // 相同長度和絕對型別
let taiwanYearMore: [number, string] = [108, '2019', 100]; // 錯誤，長度應為 2 但實際為 3
taiwanYear[1].substr(1) // 因為index 1 為string所以可以調用相對的方法

type Cordinate3D = [number, number, number];
declare function draw3D(...cordinate: Cordinate3D) : void;

draw3D(10, 10, 20) // 設定 x, y, z軸，且需為數值
let someArray = [10, 20, 30];
draw3D(...someArray) 

// TS 3.0 提供了 可選性 tuple (Optional)

type Point = [number, number?, number?];
const x: Point = [1];
const xy: Point = [1, 2];
const xyz: Point = [1, 2, 3];
```

`?` (Optional Parameter / Properties) 在物件或是interface裡也常會用到，在 `interfaces` 段落會在細說使用時機。在這裡也可以特別提一下，tuple裡的陣列，若使用 N 次 `array.push(1)` 不會報錯，因為TS無法記錄你push了幾次去做型別驗證，使用的時候需要特別小心。



### Never 
`Never` - 從英文的來說很直接，就是 `永遠不會出現不會發生不會實踐` ， 例如 Function 跳錯或無限迴圈，使用時機嘛...恩近乎0。

```typescript
function error (msg: string): never {
  throw new Error(msg);
}

function endlessLoop(): never {
  while(true){
    console.log('Never Ended')
  }
}

let arrayAsNever: [] = []; // 特別注意這裡這樣設定陣列值預設是Never
arrayAsNever.push(123); // Error: Primivite type is not allowed for never type
arrayAsNever.push((() => {throw new Error()})(), (() => {while(true){}})()) 
```

## Interfaces - DuckTyping鴨子在叫
上述說明其實提到了很多次 `interfaces` 這詞，簡單來說就是為物件定義該有的屬性。以之前的 `createUser` 舉例：

```typescript
function createUser(user: object): void {
  //...
}
```

目前 `createUser` 的型別是~~沒身份證的市民~~，但我們可以知道的是~~沒身份證的市民~~的參數必須是物件 `object`，但如果我們要給他適當的身份證，就必須對 `createUser` 的參數賦予某些固定的屬性，例如市民必有年齡，且年齡這變數必須要是數值：

```typescript
function createUser(user: {age: number}): void {
  //...
}

createUser({name: 'kangw3n', age: 30, gender: 'male'});
createUser({name: 'alice', age: '18', gender: 'female'}); // NONO 你謊報年齡了
createUser({name: 'adam', gender: 'male'}); // NONO 你不是女生不需要隱瞞年齡

```
實際上參數有N個，但是我們只要確保age的屬性必須存在，且該型別必須要是數值。其他的都不太重要。~~但這個在TS 1.6版本後就強制需要將相對的欄位指定，不然就會報錯，不過先不管他~~，但若這個參數格式是需要被重複使用的時候，我們就可以把它獨立出來變成  `interface` ：

```typescript
interface HumanBasic {
  age: number
}

function createUser(user: HumanBasic): void {
  //...
}

function createAdmin(admin: HumanBasic): void {
  //...
}
```

好了剛剛說到TS 1.6版本後不存在的屬性會讓TS報錯，所以如果我們有其他不確定存在與否的參數我們可以怎麼做呢？

```typescript
interface HumanBasic {
  age: number;
  user?: string;
  gender?: string;
}

function createUser(user: HumanBasic): void {
  //...
}

createUser({name: 'kangw3n', age: 30, gender: 'male'});
createUser({name: 'adam', age: 30});
```
以上function都不會報錯了，此時使用 `?` 就是可選性，亦指 Optional Properties，代表可有可無的意思，但如果user真的想要謊報年齡呢？ 假設 `年齡可以是數值可以是字串` 的時候怎麼辦？

```typescript
interface HumanBasic {
  age: number | string; 
  user?: string;
  gender?: string;
}
```
這裡將age屬性變成了`union type`，可以是number或是string，當然你也可以放成 `any`。但age通常只會有數值或字串，我們就暫時先維持 `union` 的寫法。下個問題來了，如果今天參數不只有這三組，或許未來還有很多未知的其他屬性該怎麼辦呢？

```typescript
interface HumanBasic {
  age: number | string; 
  user?: string;
  gender?: string;
  [others: string]: any;
}
```
這裡將未知屬性用 `[others: string]: any` 撰寫，代表以後不管增加什麼屬性，只要key是字串，且他的值是 `any` 時就不會報錯，這樣的話是不是就比較彈性了呢？

### DuckTyping在叫了...
> 「當看到一隻鳥走起來像鴨子、游泳起來像鴨子、叫起來也像鴨子，那麼這隻鳥就可以被稱為鴨子。」

其實在interface中可以延伸的就是程式語言裡的鴨子型別，舉例：

```typescript
interface Animal { move; } // move 沒有宣告型別時候預設是 any 

interface Dog extends Animal { woof; }
interface Cat extends Animal { meow; }
interface Duck extends Animal { quack; }

let unknownAnimal: Animal;
if (...condition...) {
  unknownAnimal = {move: 'Dog move', woof: 'Dog Woof'} // Error!
} else if (...condition...) {
  unknownAnimal = {move: 'Cat move', meow: 'Cat meow'} // Error!
} else if {
  unknownAnimal = {move: 'Duck move', quack: 'Duck quack'} // Error!
}
```

`extends` 指承繼，這裡的例子是：狗狗擁有動物的基本移動功能，個別動物擁有自己其他基本的功能。但我們在建立變數時，或許完全不知道這動物到底是什麼，所以預設讓他是 `Animal `型別，但經判斷後賦予它相對的屬性卻會跳錯，原因是Animal沒有其他動物可以調用的屬性。在這種狀況下有幾種解法： 

#### Type Assertion

```typescript
unknownAnimal = {move: 'Dog move', woof: 'Dog Woof'} as Dog
...
unknownAnimal = {move: 'Cat move', meow: 'Cat meow'} as Cat
...
unknownAnimal = {move: 'Duck move', quack: 'Duck quack'} as Duck
```
第一種方式是使用TypeAssertion的方式， TypeAssertion就是在賦予值後再給與相對的型別，利用 `as` keyword 即可。  
這裡也可以順帶一提，除了用 `as` 以外，也可以用先前提到的通用型別去定義它： 

```typescript
unknownAnimal = <Duck>{move: 'Duck move', quack: 'Duck quack'}
```

但由於這寫法跟JSX語法有衝突，建議還是使用 `as` 會比較好。

#### UnionSet

```typescript
let unknownAnimal: Dog | Duck | Cat;
```
第二種方式是使用union的方式，定義unknownAnimal這個變數有可能是這三種型別即可。

### 函式裡的Interfaces
我們也可以針對函式（Function）定義 `interfaces` ，舉剛剛 `createUser` 為例：

```typescript
interface HumanBasic {
  age: number | string; 
  user?: string;
  gender?: string;
  [others: string]: any;
}

interface CreateUserFn {
  (user: HumanBasic, isAdmin: boolean): void
}

let createUser: CreateUserFn; 
createUser = (user: HumanBasic, admin: boolean ): void {
  //...
}

createUser({user: 'TEST USER', age: 1}, true);
```
這裡可以特別提一下，`interface` 裡的參數名稱跟實際上調用時可以不一樣，例如 `interface` 內的第二個參數為 `isAdmin`， 在實際上使用了 `admin` 這個變數名稱。

## TS常遇到的坑

其實這才是這篇的核心，上面都是廢話，話不多說往下看一些情境吧：

### TypeAssertion不能亂用

```typescript
let initialType: number | string = 1;
let baseOnTypeAssertion = <number> initialType
baseOnTypeAssertion = 'string'
```
1. 一開始先設定了initialType為union型別。
2. 當用TypeAssertion強制把initialType轉換成 `number` 的時候，intialType原本的可用String的型別就會被覆蓋掉了，所以在這個狀況下，用TypeAssertion並沒有什麼好處。

### 關於預設值這件事

1. 當沒有對變數指定任何型別，但給予空陣列（Empty Array）的時候，預設型別會是 `any[]`。
    ```typescript
    let arrayAsAny = []; // Auto typing as any[] without any type assigned 
    arrayAsAny.push(1, '2', {a: 1}); // ALLOWED
    ```

2. 當沒有對變數指定任何型別，但給予單一陣列數值時，使用 `Array.push('字串')` 就會報錯，因為已預設幫你轉換成數值型別

    ```typescript
    let arrayAsNumberOnInitialize = [1]; // Auto typing as number[]
    arrayAsNumberOnInitialize.push('2'); // Error : Type number is initialize as number at declaration 
    ```
3. 當沒有對變數指定任何型別，但給予多重型別值的時候，以上問題好像又解決了。

    ```typescript
    let arrayAsMultipleValueOnInitialize = [1, '2']; // Auto typing as (number | string)
    arrayAsMultipleValueOnInitialize.push('2'); // ALLOWED
    arrayAsMultipleValueOnInitialize.push(1); // ALLOWED
    arrayAsMultipleValueOnInitialize = ['1', 2, 3]; // ALLOWED
    ```

4. 當對變數指定任一型別，也賦予相對型別的值後，後續去調整成空陣列是被允許的。

    ```typescript
    let arrayAsExplicitType: number[] = [1, 2]; // Only Number allowed
    arrayAsExplicitType = []; // Empty array is allowed;
    ```

5. 加了預設值就等同於`老婆出嫁後會一生被‘媳婦’這稱號給綁死`一樣，再也沒有自由了...

    ```typescript
    let plainEmptyObject = {};
    plainEmptyObject.a = 1; // Error property 'a' is not exist on {}

    let plainEmptyObject2: object = {};
    plainEmptyObject2.a = 1; // Error property 'a' is not exist on type object
    plainEmptyObject2 = {a: 1} // ALLOWED
    plainEmptyObject2.toString() // ALLOWED WHATTTT???

    let plainEmptyObjectWithoutInitialize;
    plainEmptyObjectWithoutInitialize.a = 1; // ALLOWED
     ```
  其實加上預設值在TS是個比較好的實踐方式，在初始化的過程中就決定變數可省去

### 你以為的Union是真的那麼簡單嗎？

1. 試想：當你沒辦法確認資料型別是哪些時，你會這樣做：

    ```typescript
    let unionType: number | string = 'test'; 
    unionType = 1; // GOOD
    ```
2. 當遇見了陣列時，要設定單一型別值陣列時，你會這樣做：

    ```typescript
    let arrayAsExplicitType: number[] = [1, 2]; // GOOD
    ```

3. 當遇見了陣列時，當你沒辦法確認資料型別是哪些時，你以為可以跟一般基本型別的設法一樣...

    ```typescript
    let arrayAsExplicitTypes: number[] | string[];
    arrayAsExplicitTypes.push('0'); 
    // WHATT? Argument of type '"0"' is not assignable to parameter of type 'number & string'.
    ```

4. 鄭傑正解呢？

    ```typescript
    let arrayAsExplicitTypes1: (number | string)[]; // union type
    arrayAsExplicitTypes1.push('1'); // ALLOWED
    arrayAsExplicitTypes1.push(2); // ALLOWED
    ```

`number[] | string[]` 在下意識撰寫的時候不會認為他們是必須共同存在的。但正確的方式是 `(number | string)[]` 或 ` Array<number | string>` 。

### Array.push() 是什麼，可以吃嗎？

```typescript
let arrayAsExplicitType2: [number, string]; // tuple 
arrayAsExplicitType2.push('1'); // ALLOWED
arrayAsExplicitType2.push('1'); // ALLOWED
arrayAsExplicitType2.push('1'); // ALLOWED
arrayAsExplicitType2 = [1,'2']
```
😧設定了tuple卻可以一直不停的push.... 😤😤😤 

### Never跟陣列是朋友嗎？

```typescript
let arrayAsNever: [] = []; // Type: never[] 
arrayAsNever.push(123); // Error: Primivite type is not allowed for never type 
arrayAsNever.push(() => {throw new Error()}), () => {while(true){}}) // It's not Never Type
arrayAsNever.push((() => {throw new Error()})(), (() => {while(true){}})()) // ALLOWED by execute functions itself
```
設定了空值的陣列，也定義為 `[]` 預設型別會是 `never[]`，當你以為可以直接push原本說的不會完成的函式時，其實你又錯了😭。

### 關於Type你又知道多少？

```typescript
type OnlyCSSPosition  = 'absolute' | 'fixed' | 'static' | 'relative'; 
let myBoxPosition: OnlyCSSPosition = 'initial'; // Error as expected
```
上面的 `type` 跟 `union` 沒什麼問題，只是想告訴你CSS position有這幾個屬性。~~幹你把`sticky`放到哪裡去了~~🐍🐍🐍🐍🐍？

```typescript
interface MyObject {
  a: number
  b: number
}

interface MyObject2 {
  a?: string
}

type MultipleObject = MyObject | MyObject2; //這裡看起來好像蠻簡單的

let multipleUnionObject: MultipleObject;
multipleUnionObject = {a: 1, b: 2}; // ALLOWED
multipleUnionObject = {b: 1}; // Error because it using MyObject as type but 'a' is missing
multipleUnionObject = {a: 1}; // Error because it using MyObject2 as type but value 'a' should be a string
```
使用 `union` 時會有時空錯亂的感覺，如果看仔細應該是不會太難懂，簡單來說就是 `有他沒我`！好啦你以為這樣就夠複雜了嗎？再複雜一點...

```typescript
let objectWithInterfaceByTypeAssertion = <MultipleObject | {c: string}> multipleUnionObject

objectWithInterfaceByTypeAssertion = {c: '1'}; // ALLOWED
objectWithInterfaceByTypeAssertion = {a: '1'}; // ALLOWED
objectWithInterfaceByTypeAssertion = {a: 1, b: 2, c: '1'} // ALLOWED 
```
這裡將 `objectWithInterfaceByTypeAssertion` 變數利用TypeAssertion，加了 `{c: string}` 屬性與原有的 `multipleUnionObject` 型別。   
單純只有c的時候是成立，因為union的關係是可以包含原有的 `MultipleObject` 有的型別加上 c 這個屬性🦁🦁。


## ~~不~~常用的TS功能

### keyof 

`keyof` 語義上就是物件裡面的key，簡單來說就是取得key值當其一型別：

```typescript
interface MyObject {
  a: number
  b: number
}

type ObjectKeyOnly = keyof MyObject;
let a: ObjectKeyOnly; 
a = 'b' // ALLOWED with 'a' | 'b'
a = 'c' // Error

type ArrayKey = keyof [1, 2];
let arrayKey: ArrayKey = 'map' // get all array constructor method
```
值得一提的是陣列裡面除了 `1, 2` 外， 也可以取得所有陣列相關的函式，例如 `forEach, map, push` 等。

### Readonly

```typescript
type BasicTypeObject = {a: number, b: string}
let changeObject: BasicTypeObject = {a: 1, b: '1'};
changeObject.a = 2; // changable afterward

type ReadonlyType = Readonly<BasicTypeObject>
let unchangableObject: ReadonlyType = {a: 1, b: '2'};
unchangableObject.a = 2; // Error not allowed to change after initialized;
```
當為物件加上了 `Readonly` 後，後續是無法修改裡面的值。

### Partial

```typescript
type BasicTypeObject = {a: number, b: string}
let changeObject: BasicTypeObject = {a: 1, b: '1'};

type PartialType = Partial<BasicTypeObject>
let partialObject: PartialType = {a: 1, b: '1'};

changeObject = {a: 1}; // Not allowed
partialObject = {b: '1'}; // Allowed
```
與 `?` 可選性符號相似，把原有的物件型別變成都可選，可控制一些不是自己的type，去覆蓋原有的屬性但保留intellisense。

### Required

```typescript
let requiredFromUnrequiredObject = partialObject as Required<PartialType>
requiredFromUnrequiredObject = {a: 1, b: '1'};
requiredFromUnrequiredObject = {a: 2}; //Error because b is quired from partialObject 
```
與 `partial` 剛好相反，把原本可選填的key變成必填。


### Pick

```typescript
type BasicTypeObject = {a: number, b: string}

let pickOnlyFromObject: Pick<BasicTypeObject, 'a'>
pickOnlyFromObject.a = 1;
pickOnlyFromObject = {}; // not allowed because a is missing
pickOnlyFromObject.b = '2'; // not allowed because b is not available
```
`pick` 語義上就是取得某物件裡的其中一個key當必須存在的格式，必須注意Pick需要帶入第二個參數也就是key的字串。

### Omit

```typescript
type BasicTypeObject = {a: number, b: string}

let omitFromObject: Omit<BasicTypeObject, 'a'>
omitFromObject.a = 1; // not allowed because a is not available
```
`omit` 跟 `pick` 剛好相反，忽略某些值的必填性。

### Generic

```typescript
function returnIdentity<T>(value: T): T {
  return value;
}

let outputString = returnIdentity('string');
let outputNumber = returnIdentity(1);
let wrongOutput = returnIdentity<number>('string') // error because generic expect number but get string instead

interface GenericIdentity {
  <T>(arg: T): T;
}

let getIdentity: GenericIdentity = (arg) => arg;
```

當你需要將相對的參數型別統一的時候，泛型會是一個很好的解決方法，值得一提的是 `returnIdentity('string')` === `returnIdentity<string>('string')` ，TS已自動幫你轉換了。


### Method Overload 重載函式

重載函式 `overloaded function` 在其他語言還蠻流行的，會根據你傳入的參數不同而呼叫相對應的function，原生JavaScript 並未提供這樣的功能，因為無法定義相同的名稱，但在TS提供了類似重載的功能：

```typescript 
interface MyObject {
    a: number
    b: number
}

interface MyObject2 {
    a?: string
}


function overloadFn(x: MyObject): MyObject;
function overloadFn(x: MyObject2): MyObject2;
function overloadFn(x: string): string;
function overloadFn(x: number, y: string): number;
function overloadFn(x: any, y?: any): any { return (y) ? x + y : x; }

overloadFn('a') // string
overloadFn({a: 1, b: 1}) // MyObject
overloadFn(1, 'myString') // SecondArgs
overloadFn({a: 1}) // Error 
overloadFn({a: '1'}) // MyObject2

```
如果仔細看，最後一個fn才是執行context，上面幾個都是宣告此函式可能的格式，讓函式的彈性更大，但可惜的部分是，最後一個fn則需要多做判斷，與其他真正擁有重載函式 `overloaded function` 的語言還是有差別。


### 待續
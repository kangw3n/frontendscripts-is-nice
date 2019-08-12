# TypeScript 筆記 - 淺談TS裡的Type 

## Why TypeScript

因應JavaScript的趨勢，包含目前三大框架 **（Angular, React, Vue）** 都漸漸開始使用在TypeScript（以下簡稱TS）撰寫他們的函式庫，簡單介紹一下常用的TS使用情境，相關使用方式可參考[官方網站](https://www.typescriptlang.org/samples/index.html)。 *本文部分使用TS3.0版本語法*


## 在TypeScript裡的型別: Type

在TypeScript裡有以下N種宣告型別的方式：
```typescript
let declareType: number; // 宣告Number型別
let declareTypeByInitialize = 1; // 在初始化賦值決定型別

type UnionType = 'left' | 'right'
let declareByType: UnionType; // 使用Type宣告 

let initialType: number | string = 1; //宣告一個String || Number 變數
let baseOnTypeAssertion = <number> initialType // TypeAssertion的變數決定

// ...還有很多後續會帶，但最常用的為 declareType這類型，

```

## TypeScript的基本型別

### Number
在TS裡在，與JavaScript一樣，相較於其他語言有浮點數（float，int，double）等，在TS裡通通都是Number，當然也支援十進制以外的數值。宣告如下：

```typescript
  let decimalValue: number = 10;
  let hexaValue: number = 0xf10b;
```
### String
在TS裡，與JavaScript一樣，要宣告字串型別，在文字前後加上引號即可 **單引號('') / 雙引號(" ")**
```typescript
let myStringType: string = 'string with single quotes';
let myStringType2: string = "string with double quote";
```
這裡可以特別補充一下，與其他強型別語言，如Java，宣告型別是需要用**小寫**，大寫通常都是給自訂的Interface或Type命名使用，當然與JavaScriptyiy也有一些保留文字，可以參考此[列表](https://github.com/Microsoft/TypeScript/issues/2536#issuecomment-87194347)，宣告的是否特別注意一下就好。


### Boolean
Boolean值簡單來說就是 `true` 跟 `false` 值，宣告方式：
```typescript
let isVote: boolean = true; 
let isFetch: boolean = !!0;  // false
let isParticipate: boolean = !!+"0"; // false
```
這裡可以特別提一下 `0` 和 `1`，某些程式或API會利用 0 或 1 替代 true 或 false，這裡就能用 `Double NOT (!!)` 做型別的轉換 `Truthy` 或 `Falsy`，如果值是字串時候則用 + 做 cast 將字串轉換成數字。相關可參考 MDN 文章 (https://developer.mozilla.org/en-US/docs/Glossary/truthy)

### Enums
Enums 是個抽像型別，中文為列舉或是狀態機，主要能定義一組固定的常數 `constants`，相關使用時機可以參閱[Enums](https://www.typescriptlang.org/docs/handbook/enums.html)官方說明。 
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
Void型別通常都是利用在涵式沒有回傳任何值的時候，例如設定狀態或是顯示訊息等，不需要從function取得任何的資料。通常都是用在Function，但也可以用在變數上，但基本上宣告void型別的變數只能賦予 `undefined` 或 `null` 值。

```typescript
function setMessage(message: string): void {
  alert(message);
}

setMessage('你TS真的好厲害歐～');

let declareSomeThingVoid: void;
declareSomeThingVoid = 1; // ERROR 1 不是void型別

```

### Null 和 Undefined
這兩種型別比較少在用，語義上和JS是一樣的，但他們有個共同的特質是，`null` 和 `undefined`是其他型別的副類型(subtype)。

```typescript
 let nullVal: null = null;
 let nullNumber: number = null;
 let undefinedNumber: number = undefined;
```

### Any
~~一個當你覺得TS很複雜很難但又不知道要放什麼型別的時候下它就對了~~，any就是一個非常彈性的型別，它可以是以上所有的型別都不會報錯，在你還不知道你的內容是什麼格式的時候，可以用any先來宣告，在 `C#` 裡面就是 `dynamic` keyword

```typescript
let dynamicData: any = 'iAmString';
dynamicData.concat(' and iLoveYou'); //可以直接調用相關的方法
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
let myWhateverArray = ['1', 2, {a: 1}]; //當沒提供任何型別則初始化陣列時，此定義跟any是一樣的

let myGenericArry: Array<number> = [1, 2, 3]; //此方式跟myArray是一樣的，在TS裡這種宣告方式為通用型別 `Generic Type `

```

### Object
物件就是物件 (**~~詞窮~~**)，但通常這種定義物件的方式在JS是沒什麼~~屁~~用的，等等在說明 `interface` 時候或許會是更好定義物件的方式。物件型別代表任何不是原始型別`(Non-Primitive Type)`的類別，如 `number`、`string` 、`boolean` 等。相關原始型別請參考(https://medium.com/@jobboy0101/js%E5%9F%BA%E7%A4%8E-primitive-type-v-s-object-types-f88f7c16f225)。
```typescript
function createUser(user: object): boolean {
  //...
}

createUser({email: 'fakepeople@fakeemail.com', firstName: 'w3n', url: 'XX'});
createUser({firstName: 'TEST USER'}); // 只要參數是物件即可
createUser(new Function); //咦～搞笑的還真的可以 
```

以最後一個例子而言，只要是物件型別的都可以被帶進去，之後我們說`interface`的時候可以將物件定義的更清楚。

### Tuple
Tuple就是一組資料，一組不可更變的資料, 通常運用在陣列上，當你有明確的型別格式和絕對的長度 `length` 時候可使用。

```typescript
let taiwanYear: [number, string] = [108, '2019']; // 相同長度和絕對型別
let taiwanYearMore: [number, string] = [108, '2019', 100]; // 錯誤，長度應為2
taiwanYear[1].substr(1) // 因為index 1 為string所以可以調用相對的方法

type Cordinate3D = [number, number, number];
declare function draw3D(...cordinate: Cordinate3D) : void;

draw3D(10, 10, 20) // 設定 x, y, z軸，且需為數值
draw3D(...someArray) 

// TS 3.0 提供了 可選性 tuple (Optional)

type Point = [number, number?, number?];
const x: Point = [1];
const xy: Point = [1, 2];
const xyz: Point = [1, 2, 3];
```

`?` 可選性在物件或是interface裡也常會用到，在`interface`段落會在細說使用時機。在這裡也可以特別提一下，tuple裡的陣列，若使用 N 次`array.push(1)` 不會報錯，因為TS無法記錄你push了幾次去做型別驗證，使用的時候需要特別小心。


### Never 
Never從英文的來說很直接，就是`永遠不會出現不會發生不會實踐`, 例如 Function 跳錯或無限迴圈，使用時機嘛...恩近乎0。

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

## Interface- DuckTyping
上述的說明其實提到了很多次`interface`這詞，簡單來說就是為物件定義該有的屬性。以之前的`createUser` 舉例:

```typescript
function createUser(user: object): void {
  //...
}
```

這時候的createUser所需要的參數必須是物件，但是物件裡長什麼樣子根本不重要，但我們在createUser有時必須要確認某些屬性，例如某User必有年齡，且年齡這變數必須要是數值: 

```typescript
function createUser(user: {age: number}): void {
  //...
}

createUser({name: 'kangw3n', age: 30, gender: 'male'});
createUser({name: 'adam', age: '18', gender: 'male'}); // NONO 你謊報年齡
createUser({name: 'alice', gender: 'female'}); // NONO 你不是女生不需要隱瞞年齡

```
實際上參數有N個，但是我們只要確保age的屬性必須存在，且該型別必須要是數值。其他的都不太重要。~~但這個在TS1.6版本後就強制需要將相對的欄位指定，不然就會報錯，不過我先不管先這樣說~~, 但若這個參數格式是需要被重複使用的時候，我們就可以把它獨立出來變成 `interface`:

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

好了剛剛說到TS1.6版本後不存在的屬性會讓TS報錯，所以如果我們有其他不確定存在與否的參數我們可以怎麼做呢？

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
```
以上function都不會報錯了，`?` 這個就是可選性的意思，亦指 Optional Properties，代表可有可無的意思，但如果user真的想要謊報年齡呢？`年齡可是數值可是字串`的時候怎麼辦？

```typescript
interface HumanBasic {
  age: number | string; 
  user?: string;
  gender?: string;
}
```
這裡將age屬性變成了`union type`，就是可以是number或是string，當然你也可以放成`any`。但age通常只會有數值或字串，我們就暫時先維持`union`的寫法。下個問題來了，如果今天參數不只是這三個，或許還有其他屬性但我們卻無法預知的時候呢？

```typescript
interface HumanBasic {
  age: number | string; 
  user?: string;
  gender?: string;
  [others: string]: any;
}
```
這裡將未知屬性用`[others: string]: any` 撰寫，代表以後不管增加什麼屬性，只要key是字串，且他的值是`any`時就不會報錯，這樣的話是不是就比較彈性了呢？

### DuckTyping
> 「當看到一隻鳥走起來像鴨子、游泳起來像鴨子、叫起來也像鴨子，那麼這隻鳥就可以被稱為鴨子。」

其實在interface中可以延伸的就是JS裡的鴨子型別，舉例：

```typescript
interface Animal {
  move: string
}

interface Dog extends Animal { woof: string; }
interface Cat extends Animal { meow: string; }
interface Duck extends Animal { quack: string; }

let unknownAnimal: Animal;
if (...) {
  unknownAnimal = {move: 'Dog move', woof: 'Dog Woof'} // Error!
} else if (...) {
  unknownAnimal = {move: 'Cat move', meow: 'Cat meow'} // Error!
} else if {
  unknownAnimal = {move: 'Duck move', quack: 'Duck quack'} // Error!
}
```

`extends`指承繼，狗狗擁有動物的基本移動功能，個別動物擁有自己其他基本的功能。但我們在建立變數時，或許完全不知道這動物到底是什麼，所以預設讓他是`Animal`型別，但在判斷後賦予它相對的屬性卻會跳錯，這裡有幾種做法： 

#### Type Assertion

```typescript
unknownAnimal = {move: 'Dog move', woof: 'Dog Woof'} as Dog
...
unknownAnimal = {move: 'Cat move', meow: 'Cat meow'} as Cat
...
unknownAnimal = {move: 'Duck move', quack: 'Duck quack'} as Duck
```
TypeAssertion就是在賦予值後再給與相對的型別，利用 `as` keyword 即可。這裡也可以順帶一提，除了用 `as` 以外，也可以用先前提到的通用型別去定義它： `<Duck>{move: 'Duck move', quack: 'Duck quack'}`，但由於這寫法跟JSX語法有衝突，建議還是使用`as`會比較好。

#### UnionSet

```typescript
let unknownAnimal: Dog | Duck | Cat;
```
將原本使用union的方式，去定義unknownAnimal這個變數有可能是這三種型別即可。

### 函式裡的Interfaces
我們也可以針對函式定義interfaces，舉剛剛createUser為例：

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
createUser = (user: HumanBasic, admin: boolean ) {
  //...
}

createUser({user: 'TEST USER', age: 1}, true)
```
這裡可以特別提一下，interfaces裡的參數名稱跟實際上調用時可以不一樣，例如interfaces內的第二個參數為`isAdmin`， 在實際上套用時用了`admin`。

## TS常遇到的坑

其實這才是這篇的核心，上面都是廢話，話不多說往下看一些情境吧：

### TypeAssertion不能亂用

```typescript
let initialType: number | string = 1;
let baseOnTypeAssertion = <number> initialType
baseOnTypeAssertion = 'string'

```
1. 一開始先設定了initialType為union型別。
2. 當用TypeAssertion強制把initialType轉換成number的時候，intialType原本的可用String的型別就會被覆蓋掉了，所以在這個狀況下，用TypeAssertion並沒有什麼好處。

### 關於預設值這件事

```typescript
let arrayAsAny = []; // Auto typing as any[] without any type assigned 
arrayAsAny.push(1, '2', {a: 1}); // ALLOWED
```
當沒有對變數指定任何型別但給予空陣列的時候，預設型別會是`any`。

```typescript
let arrayAsNumberOnInitialize = [1]; // Auto typing as number[]
arrayAsNumberOnInitialize.push('2'); // Error : Type number is initialize as number at declaration 
```
當沒有對變數指定任何型別但給予單一陣列數值時，使用push字串就會報錯，因為已預設幫你轉換成數值型別

```typescript
let arrayAsMultipleValueOnInitialize = [1, '2']; // Auto typing as (number | string)
arrayAsMultipleValueOnInitialize.push('2'); // ALLOWED
arrayAsMultipleValueOnInitialize.push(1); // ALLOWED
arrayAsMultipleValueOnInitialize = ['1']; // ALLOWED
```
當沒有對變數指定任何型別但給予多重型別值的時候，問題好像又解決了。

```typescript
let arrayAsExplicitType: number[] = [1, 2]; // Only Number allowed
arrayAsExplicitType = []; // Empty array is allowed;
```
當對變數指定任何型別，也賦予相對型別的值後，後續去調整成空陣列是被允許的。

```typescript
let plainEmptyObject = {};
plainEmptyObject.a = 1; // Error property 'a' is not exist on {}

let plainEmptyObject2: object = {};
plainEmptyObject2.a = 1; // Error property 'a' is not exist on type object

let plainEmptyObject3: object = {};
plainEmptyObject3.toString() // ALLOWED WHATTTT???

let plainEmptyObjectWithoutInitialize;
plainEmptyObjectWithoutInitialize.a = 1; // ALLOWED
```
加了預設值就等同於`老婆出嫁後會一生媳婦這個稱號給綁死`一樣，再也沒有自由了...

### 你以為的Union是真的那麼簡單嗎？

```typescript
// 設定多個型別，你會這樣做
let unionType: number | string = 'test'; 
unionType = 1; // ALLOWED

// 當遇見了陣列時，要設定單一型別值陣列時，你會這樣做
let arrayAsExplicitType: number[] = [1, 2];

// 當遇見了陣列時，若想要可是數值或是字串的陣列，你以為...
let arrayAsExplicitTypes: number[] | string[];
arrayAsExplicitTypes.push('0'); // Error : Argument of type '"0"' is not assignable to parameter of type 'number & string'.

// 鄭傑正解呢？
let arrayAsExplicitTypes1: (number | string)[]; // union type
arrayAsExplicitTypes1.push('1'); // ALLOWED
arrayAsExplicitTypes1.push(2); // ALLOWED

```
`number[] | string[]` 第一次看到的時候怎麼都不會認為他們是必須要共同存在。但正確的方式是 `(number | string)[]` 或 ` Array<number | string>`。

### Array.push() 是什麼，可以吃嗎？

```typescript
let arrayAsExplicitType2: [number, string]; // tuple 
arrayAsExplicitType2.push('1'); // ALLOWED
arrayAsExplicitType2.push('1'); // ALLOWED
arrayAsExplicitType2.push('1'); // ALLOWED
arrayAsExplicitType2 = [1,'2']
```
設定了tuple卻可以一直不停的push....

### Never跟陣列是朋友嗎？
```typescript
let arrayAsNever: [] = []; // Type: never[] 
arrayAsNever.push(123); // Error: Primivite type is not allowed for never type
arrayAsNever.push(() => {throw new Error()}), () => {while(true){}}) // It's not Never Type
arrayAsNever.push((() => {throw new Error()})(), (() => {while(true){}})()) // ALLOWED by execute functions itself
```
設定了空值的陣列，也定義為 `[]` 預設型別會是 `never[]`，當你以為可以直接push原本說的不會完成的函式時，其實你又錯了。

### 關於Type你又知道多少？

```typescript
type OnlyCSSPosition  = 'absolute' | 'fixed' | 'static' | 'relative'; 
let myBoxPosition: OnlyCSSPosition = 'initial'; // Error as expected
```
上面的type跟union沒什麼問題，就只是告訴你CSS position有這幾個屬性。~~幹你把`sticky`放到哪裡去了~~？

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
multipleUnionObject = {b: 1}; //Error because it using MyObject as type but 'a' is missing
multipleUnionObject = {a: 1}; // Error because it using MyObject2 as type but value 'a' should be a string
```
使用union時候會有時空錯亂的感覺，但如果看仔細應該是不會太難懂，簡單來說就是 `有他沒我`！好啦你以為這樣就夠複雜了嗎？再複雜一點...

```typescript
let objectWithInterfaceByTypeAssertion = <MultipleObject | {c: string}> multipleUnionObject

objectWithInterfaceByTypeAssertion = {c: '1'}; // ALLOWED
objectWithInterfaceByTypeAssertion = {a: '1'}; // ALLOWED
objectWithInterfaceByTypeAssertion = {a: 1, b: 2, c: '1'} // ALLOWED 
```
這裡是將 `objectWithInterfaceByTypeAssertion` 變數利用TypeAssertion加了 c 這個屬性和原用上面的 `multipleUnionObject` 原有的型別，單純只有c的時候是成立，但第三行我一開始真的也不太懂為何會成立，但他就是成立了...好啦其實他這樣是對的，因為union的關係他是可以包含原有的 `MultipleObject` 有的型別 加上 c 這個屬性。


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
值得一提的是陣列裡面除了 `1, 2` 外， 也可以取得所有陣列相關的函式，例如 `forEach, map, push` 等

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
與partial剛好相反，把原本可選填的key變成必填


### Pick

```typescript
type BasicTypeObject = {a: number, b: string}

let pickOnlyFromObject: Pick<BasicTypeObject, 'a'>
pickOnlyFromObject.a = 1;
pickOnlyFromObject = {}; // not allowed because a is missing
pickOnlyFromObject.b = '2'; // not allowed because b is not available
```
`pick` 語義上就是取得某物件裡的其中一個key當必須存在的值，必須注意Pick需要帶入第二個參數也就是key的值。

### Omit

```typescript
type BasicTypeObject = {a: number, b: string}

let omitFromObject: Omit<BasicTypeObject, 'a'>
omitFromObject.a = 1; // not allowed because a is not available
```
`omit` 跟 `pick` 也是剛好相反，忽略某些值的必填性。

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

當你需要將相對的參數型別統一的時候，泛型會是一個很好的解決方法，值得一提的是 `returnIdentity('string')` === `returnIdentity<string>('string')` ，TS已自動轉換了。



### 待續

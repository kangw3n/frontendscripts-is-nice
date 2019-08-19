# 跟 Vue 相關的都可以

把一些在 Vue 框架學習過程常用的技巧記錄一下，希望可以幫助一些初學者或是迷路者~~

## 元件溝通 `(Component Communication)`

我們都知道 Vue 是個以元件組成的框架，區塊與功能基本上都會切成元件讓整體應用程式變得更有組織化，這也是現代 JavaScript 框架常使用的設計模式。

元件式設計其實並不困難，但當資料需要透過跨元件傳遞時，通常都會讓初學者有點搞不清楚使用的時機與方法；什麼是跨元件溝通呢？以常用的社群應用程式作為例子好了，在使用某 `F` 開頭的社群應用程式中，當收到新私訊時選單上會出現紅色圈圈提醒，打開私訊小視窗的後該圈圈就會自動消失，這就是基本的跨元件溝通，右下角的小視窗和選單上的 icon 基本上個別的元件，他們之間的資料傳遞需要透過某種機制的傳遞才能達到狀態 `（state）` 的一致性。

### 狀態管理 (State Management)

> 💡 狀態 `(State)` --  應用程式在某個時間點的狀況/值

在現代應用程式中，免不了需要做狀態的儲存。狀態 `(State)` 可以理解成應用程式在某個時間點的狀況，例如：`目前是否正在傳遞資料` 、 `目前是否在等待` 、`目前選單是否被打開` 、 `目前使用者流程已到了哪個階段 (Step)` ，之類的都可以被理解成 `狀態`。這些狀態是否跟其他元件有關？他們之間要怎麼傳遞或綁定，讓狀態是一致的，就是我們今天要探討的議題。

在程式設計模式裡，狀態的溝通方式脫離不了 `觀察者模式` (Observer Pattern)。 雖然這單元介紹的幾種 `元件溝通` 方式未必都屬觀察者模式，但他在程式設計裡是一個常用模式，尤其是面試題裡常考的 `pubsub` 題目，有興趣的可以參考此篇[文章](https://blog.csdn.net/one_girl/article/details/80325210)，我就不再這裡細說了。

### Vue 的狀態與元件溝通

> 這裡會細說設計 Vue 應用程式中狀態和元件的組成，如果你只是想看跨元件溝通的方式可以直接跳到[這裡](#%e8%b7%a8%e5%85%83%e4%bb%b6%e6%ba%9d%e9%80%9a%e7%9a%84%e6%96%b9%e5%bc%8f%e6%9c%89-n-%e7%a8%ae)。

如何在 Vue 中理解狀態和元件，首先以簡單的 `TodoApp` 為例，功能大致可以分為：

1. `input#newTodo` : 使用者欲輸入新的 `todo` input 框。
2. `li.todo[]` : 已存取或執行中的 `todo` 。 _（[]代表是個陣列）_

```html
<!-- component 1 -->
<input id="newTodo" placeholder="Add New Task Here" />

<ul>
  <!-- component 2 -->
  <li class="todo">My Task 1 <span class="delete">x</span></li>
  ...
</ul>
```

這例子中，`state` 會是已儲存的 `li.todo` 以及使用者欲輸入的 `todo`，簡而言之是存取在 `data` 裡的資料都可以是狀態，拆成 Vue 的設計可以是：

```javascript
new Vue({
  el: "#app",
  data: {
    newTodo: "",
    todos: [
      {
        title: "My Task 1",
        isDone: false,
        id: 1
      }
    ]
  },
  methods: {
    deleteTodo(id) {
      this.todos = this.todos.filter(todo => todo.id !== id);
    }
  }
});
```

這裡值得一提的是，所有在 `data` 裡的值都屬 `觀察者模式` 控制，它們~~至少~~是單向綁定的動態 [`（Reactive）`](https://vuejs.org/v2/guide/reactivity.html) 狀態。HTML 可以拆成：

```html
<div id="app">
  <input id="newTodo" v-model="newTodo" placeholder="Add New Task Here" />
  <li
    v-for="(todo, i) in todos"
    :key="i"
    class="todo"
  >
    {{todo.name}} <span class="delete" @click="deleteTodo(todo.id)">x</span>
  </li>
</div>
```

在這案例中，`li.todo[]` 裡有個刪除的函式，按下 `x` 會把自己刪除，在不拆除成 component 的狀況下，基本上功能已經完成了，我們也完成了狀態的管理。但假設 `li.todo[]` 需要在應用程式內其他元件上共用時，`元件` 設計可以幫我們省去撰寫重複且多餘的 code。

要怎麼拆成`元件`呢？ 我們可以將 `TodoApp` 裡的功能都歸類為元件，也就是剛剛說的 `input#newTodo` 和 `li.todo[]` 。拆成元件後 code 可以是：

```html
<div id="app">
  <Newtodo />
  <ul>
    <Todo v-for="(todo, i) in todos" :key="i" />
  </ul>
</div>
<script>
  Vue.component("Newtodo", {
    template: `
    <div>
      <input id="newTodo" v-model="newTodo" placeholder="Add New Task Here">
    </div>`
  });

  Vue.component("Todo", {
    template: `
      <li class="todo" >
        {{todo.name}} 
        <span class="delete" @click="deleteTodo(todo.id)">x</span>
      </li>`,
    methods: {
      deleteTodo(id) {}
    }
  });

  new Vue({
    el: "#app",
    data: {
      newTodo: "",
      todos: [
        {
          title: "My Task 1",
          isDone: false,
          id: 1
        }
      ]
    }
  });
</script>
```
> 💡 這裡在全域 `Vue.component()` 註冊了通用的元件，所以不必在使用的元件上定義 `components: ['Todo', 'Newtodo']` 


我們會發現，拆成元件後，程式就壞掉了，現在元件間有著曖昧的關係，以目前的案例：`#app` 就是爸爸， `<Newtodo />` 和 `<Todo />` 變成是小孩。當~~曖昧~~關係產生的時候，則是~~勇敢告白的時候哦不是~~，是元件溝通出來解救的時候，也就是會讓程式變得複雜的第一步？~~誒~~ 。

後記： `TodoApp` 其實還有很多功能，包含新增/修改 `todo` 等，為了不讓程式變得太複雜，例子中會省略部分的功能，讓大家聚焦在資料傳遞的方式上。

### 跨元件溝通的方式有 N 種

#### 1. `props` 和 `$emit`

這是Vue裡最基本的元件溝通方式，也是在程式不複雜、在不超過一層元件溝通時最好用的方式。以 `<Todo />` 元件為例：

```html
<Todo v-for="(todo, i) in todos" :key="i" :todo="todo" @delete="deleteTodo" />
<script>
  Vue.component("Todo", {
    template: `
      <li class="todo" >
        {{todo.name}} 
        <span class="delete" @click="deleteTodo">x</span>
      </li>`,
    props: ["todo"], // 這裡的props就是從父層帶進來的參數
    methods: {
      deleteTodo() {
        this.$emit("delete", { id: this.todo.id }); // 在子層內定義的函式與父層溝通
      } 
    }
  });
</script>
```

幾個特點整理：

1. `props`：語義上可理解成 `屬性`， 從父層調用的元件帶入特殊的屬性 `todo` ，它是單向資料流，在元件內修改並不會影響到父層。
2. `$emits`： 語義上可理解成 `事件發送`，這跟 JavaScript 的新增[特殊事件](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent/CustomEvent)是一樣的運行邏輯。我們在元件內的 deleteTodo 函式裡 `發送` 了一個事件叫 `delete`， 在調用元件的父層透過 `$on` 監聽事件取得 `@delete` 事件後執行相對的函式。

以我們 `TodoApp` 為例， `props` 跟 `$emit` 其實已經很夠用了。

#### 2. `EventBus` 集中巴士

剛剛有提到，`props` 和 `$emit` 適合一層以內的資料傳遞，但試想假設我們也要將 刪除按鈕 - `X` 也拆成元件呢？這時候就 `deleteTodo` 就會超過一層的資料傳遞了，當然 `props` 和 `$emit` 的方式還是可以解決問題，但是到了 3-4 層後會開始覺得一直在不同的元件上寫了好多的 `props` 和 `$emit`，複雜起來是非常煩人的，這時候需要 `EventBus` 來幫忙了。

`EventBus` 語義上其實很好懂，就是一個匯集所有事件的巴士，特別開了一個新的 Vue 實例 `（instance）` 專門用來處理事件與狀態本身。

```javascript
const bus = new Vue();
```

之後就直接用 `$on` 和 `$emits` 事件傳送資料即可，`<Todo />` 元件只需改一行：

```javascript
Vue.component("Todo", {
  template: `
    <li class="todo">
      {{todo.name}} 
      <span class="delete" @click="deleteTodo">x</span>
    </li>`,
  props: ["todo"],
  methods: {
    deleteTodo() {
      bus.$emit("delete", { id: this.todo.id }); // 原本的this改成透過bus發送事件
    }
  }
});
```

父層只要加個 `mounted` 的監聽事件：

```javascript
new Vue({
  ...//略過
  mounted() {
    bus.$on('delete', ({id}) => {
      this.todos = this.todos.filter(todo => todo.id !== id);
    });
  }
});
```

這樣我們就可以不必在每一個元件加上事件監聽，只要監聽集合巴士的內容即可。不過若在某些狀況需要移除事件則需要： `bus.$off('delete')` 。與 JavaScript 的 `removeEventListener` 是一樣的。

#### 3. Vuex 狀態管理系統

Vuex 借鑒了 Flux 和 Redux 的理念，以 `store` 存取資料的概念，它和EventBus的集合巴士相似，用 `倉庫` 去理解他或許會比較好懂，這倉庫裡包含了所有的狀態的商業邏輯，把需要被監聽的狀態，全權交給 `store` 處理。`Vuex` 主要分為三個概念：

1. State: 狀態的根源
2. Mutation: 執行更改狀態的函式，但要小心 `mutation` 是沒辦法處理非同步事件
3. Action：提交事件的概念，跟 `mutation` 有點類似，但是要記得他是無法更改狀態的 context，但它可以在同一時間同時處理 N 個 `mutation`，也可以處理非同步事件。

Vuex 是個比較進階的概念，其中還包含 `setters` 與 `getters` ，我在這裡就不太深入敘述，有興趣的人可以查看[官方文件](https://vuex.vuejs.org/) 或是 [此篇簡介](https://www.itread01.com/content/1544948592.html)。

若將上述 `<Todo />` 修改為 Vuex 架構，則需將原本在 `data` 裡的 `todos[]` 變數和 `deleteTodo` 函式交由 Vuex 管理，為了讓大家了解Vuex的使用時機，我把新增和修改Todo的程式碼也放進來 ：

> 💡 記得要先引入Vuex的函式庫，請參考適合專案的[安裝方式](https://vuex.vuejs.org/installation.html)。

```javascript
Vue.use(Vuex); 
const store = new Vuex.Store({
  state: {
    todos: [{
      title: 'My Task 1',
      isDone: false,
      id: 1
    }],
    message: '', // API返回錯誤訊息
    isSaving: false, //增加了偵測todo是否在非同步狀況下存取中
  },
  mutations: { 
    SET_TODO(state, payload) {
      state.todos = [...state.todos, payload.todo];
    },
    DELETE_TODO(state, {id}) {
      state.todos = state.todo.filter(todo => todo.id !== id);
    },
    EDIT_TODO(state, {id, title}) {
      state.todos = state.todo.map(todo => {
        if (todo.id === id) todo.title = title;
        return todo;
      });
    },
    SAVING_START(state) {
      state.isSaving = true;
    },
    SAVING_END(state) {
      state.isSaving = false;
    },
    SAVING_FAILED(state, {todos, error}) {
      state.message = error;
      state.todos = [...todos];
    }
  },
  getters: {}, // 這裡沒有多敘述 getters，它概念跟vue的computed是相似的
  actions: { // 欲提交的事件
    editTodo({ commit, state}, todo) { 
      commit('EDIT_TODO', todo);
    },
    setTodo({ commit, state }, todo) { // 非同步或多重commit
      if (state.isSaving) return; //  如果上一個todo仍在存取中，先跳走
      let getOriginalTodos = [...state.todos]; // 拷貝原本的todos[]
      commit('SAVING_START'); // 1️⃣ 先告訴store目前正準備存取

      return fetch('/saveTodo', {body: JSON.stringify(todo), method: 'POST'})
        .then(res => res.json())
        .then(data => {
          if (data.state) commit('SET_TODO', todo); // 2️⃣  儲存成功後則設定新的todo到狀態上
        })
        .catch(error => {
          commit('SAVING_FAILED', {todos: getOriginalTodos, error});
        })
        .finally(() => commit('SAVING_END')); // 3️⃣ 完成儲存

    },
    deleteTodo({ commit, state }, id) { 
      if (state.todos.length) commit('DELETE_TODO', {id});
    }
  }
})
```
在原本的 `#app` 內就變成：

```javascript
new Vue({
  el: "#app",
  data: {
    newTodo: ""
  },
  computed: {
    todos () {
      return store.state.todos; // 換成 computed 去偵測 狀態的修改
    }
  },
  methods: {
    deleteTodo(id) {
      store.dispatch('deleteTodo', id); // 直接呼叫某個action即可
    },
    saveTodo(todo) {
      store.dispatch('setTodo', todo);
    },
    ... /// 其他code略過
  }
});
```
在 `#app` 裡比較重要的改變會是狀態的取得方式，我們改成用 `computed` 的方式監聽 `store` 裡的 `todos` 狀態，在任何元件內都可以呼叫store的 `dispatch` 函式，這樣可以把所有跟狀態相關的函式組織在倉庫內，邏輯切分的清楚。

> 💡 Vuex看似需要更多的程式碼達成相同的功能，但是當功能變得複雜，或跨太多層元件溝通時，Vuex可以扮演著不錯的~~告白~~角色。

以上三個是我們在Vue專案常見的跨元件溝通方式，但其實在Vue裡面還有很多方式可以取得父子層的資料，讓我們繼續往下看看吧。

#### 4. `$parent` 和 `$children`

從名稱上就可以明確的知道： `$parent` 和 `$children` 就是取得父層和子階層的實例 `instance` 。在居多的元件溝通方式裡官方是不建議是用這種的做法，但是我們可以透過 `TodoApp` 來看看這兩種做法的差異。

```html
<div id="app">
  <Newtodo :value="newTodo"></Newtodo> 
  <input type="text" :value="newTodo" @keyup="triggerChild"> 
  <ul>
    <Todo />
  </ul>
</div>
<script>
  Vue.component("Newtodo", {
    template: `
    <div>
      <input id="newTodo" v-model="value" placeholder="Add New Task Here">
    </div>`,
    props: ['value']
  });

  Vue.component("Todo", {
    template: `
      <li class="todo" v-for="(todo, i) in todos" :key="i">
        {{todo.name}} 
        <span class="delete" @click="deleteTodo(todo.id)">x</span>
      </li>`,
    computed: {
      todos() {
        return this.$parent.todos; // 取得父層的某狀態
      }
    }  
  });

  new Vue({
    el: "#app",
    data: {
      newTodo: "",
      todos: [
        {
          title: "My Task 1",
          isDone: false,
          id: 1
        }
      ]
    },
    methods: {
      triggerChild(e) {
        this.$children[0].value = e.target.value; // 透過keyup事件把value值綁定到子階層上
      }
    }
  });
</script>
```

這裡比較特別，我們分成兩個部分解釋好了：

1. `$parent` 的用法：我們設在 `<Todo />` 裡設定了 `computed` 去監聽父層的todos狀態的改變，一旦狀態改變，觸發 `<Todo />` 層 `computed` 更新 View。
2. `$children` 的用法比較特別：從父層裡設定某個 `$children` 的值。此案例裡，在頁面上input綁定了 `newTodo` 狀態， `@keyup` 事件發生的時候會觸發 `this.$children[0].value = e.target.value;`，也就是將子階層內的值賦予input的值。

這個案例實際上單純只是為了示範 `$parent` 和 `$children` 的使用方式，更好的做法是將 `newTodo` 利用 `v-model` 雙向綁定就可以達到想要的效果了。

> 💡 `this.$children` 是陣列 `(Array)` 型別，依據目前view中渲染的數量而定，它的值是動態的，使用的時候要特別小心，當子階層含有非常複雜的結構時， 就會無法正確地取得 `$children` 的 index 值，這是 `$children` 的缺陷。

#### 5. `provide` 和 `inject`

`provide` 語義上就是 `提供` 的意思，我們可以在父層宣告一個可提供給所有需要使用該狀態的子階層，子階層可以透過 `inject` 取得該。我用較簡單裡的例子來說明好了：

```html
<div id="app">
  <Component1></Component1>
</div>
<script>

Vue.component('Component2', {
  template: `
    <div>
      {{value}}
    </div>`,
  inject: ['for'] // 3️⃣ 從祖父層取得for這個變數
  data: {
    value: this.for
  }
})

Vue.component('Component1', {
  template: `
    <div>
      {{value}}
      <Component2></Component2>
    </div>`,
  inject: ['for'] // 2️⃣ 從父層取得for這個變數
  data: {
    value: this.for
  }
});
new Vue({
  el: "#app",
  provide: {
    for: "myValue" // 1️⃣ 提供在這層下所有的子階層的狀態 : for
  }
});
</script>
```
從這個例子我們可以發現，在元件上利用 `provide` 提供給該層級所需要的變數，該層級所有的子階層即可選擇是否需要使用該變數，如果需要則用 `inject` 即可取得。

> 💡 我們在上面的例子可以發現，在 `<Component1 />` 並不需要宣告 `provide` 物件，因為在父層被注射後的元件可以自動帶到下一層級，所以只要在 `<Component2 />` 用 `inject` 依然可以取得 `for` 的變數。但若中間有元件沒有注入 `inject`屬性則就不會再往下傳了。

#### 6. $refs 
在Vue裡面可以透過 `ref` 取得 DOM 物件亦可透過 DOM API 操控該物件，但 `ref` 基本上也可以針對元件本身設定，即可得到元件本身的實例，得到實例後我們就可以取得某些狀態，再用個簡單的例子：

```html
<div id="app">
  <Component1 ref="com"></Component1>
</div>
<script>
  Vue.component('Component1', {
    method: {
      sayHi: () => console.log('sayHi')
    },
    data: {
      myPrivateValue: 98
    }
  });
  new Vue({
    el: '#app',
    mounted() {
      console.log(this.$refs.com.myPrivateValue); // 98
      console.log(this.$refs.com.sayHi) // function() {console.log('sayHi')}
    }
  });
</script>
```
嚴格來說，這是取得子階層變數的一種方式，而且資料流也是單向的，若要雙向的話則需要用 `computed` 去監聽改變， 但應該是不會有人用這種方式傳遞資料。

> 💡 依然還是可以使用 `props` 和 `:value` binding 的方式傳值， `$refs.XXX` 單純是取得該元件的實例。

#### 7. `$attrs` 和 `$listeners`
Vue 2.4.0 版本加入了`$attrs` 和 `$listeners`，它讓我們可以快速的傳遞 `props` 和 `$emit` 發送事件，我們用例子來解釋可能會比較容易。

##### `$attrs`
先來解釋 `$attrs` ， 語義上是標籤上的屬性，簡單來說就是元件上被帶入的 `props` 可以被中途攔截或繼續往下傳遞。 
```html
<!-- 1️⃣ 宣告了給別的屬性提供於子階層 -->
<fake-user name="kangw3n" age="30" title="custom-title"></fake-user>

Vue.component('fake-user', {
  created() {
    console.log(this.$attrs); // 2️⃣ { name: 'kangw3n', age: '30', title="custom-title" }
  }
});

<!-- 3️⃣ 實際頁面渲染的結果 -->
<div name="kangw3n" age="30" title="custom-title"></div>
```
我們可以看到，利用 `$attrs` 可以取得父層傳進來所有的 `props`。

但上述的案例可以發現，被帶進去的 `title` 屬性被當成HTML標籤被渲染出來了，在某些狀況下或是不是我們想要的。因此api上也加 `inheritAttrs` 參數，主要是針對被帶進元件的屬性在渲染的過程中在頁面上給關閉： 
```html
<!-- 1️⃣ 宣告了給別的屬性提供於子階層 -->
<fake-user name="kangw3n" age="30" title="custom-title"></fake-user>

Vue.component('fake-user', {
  inheritAttrs: false, // 2️⃣ 設定 false 會導致在dom渲染上把屬性隱藏
  created() {
    console.log(this.$attrs); // 3️⃣ { name: 'kangw3n', age: '30', title="custom-title" }
  }
});

<!-- 4️⃣ inheritAttrs = false 後實際頁面渲染的結果 -->
<div></div>
```
> 💡 雖然屬性沒渲染在頁面上， `$attrs` 依然可以取得該屬性。

再來，我們可以透過 `props` 設定來攔截不想要提供給實例. `$attrs` 使用的屬性：
```html
<!-- 1️⃣ 宣告了給別的屬性提供於子階層 -->
<fake-user name="kangw3n" age="30"></fake-user>

Vue.component('fake-user', {
  props: ['age'], // 2️⃣ 宣告了自己的props
  created() {
    console.log(this.$attrs); // 3️⃣ { name: 'kangw3n'}
    console.log(this.age); // 4️⃣ 30
  }
});
```
> 💡 此時 this.age 依然可以取得，只是從 $attrs 移除了。

厲害的來了，透過 `v-bind` 可以把子階層綁定該層級的 `$attrs`：
```html
<!-- 1️⃣ 宣告了給別的屬性提供於子階層 -->
<fake-user name="kangw3n" age="30" gender="male"></fake-user>

Vue.component('child', {
  props: ['gender'], // 3️⃣ 攔截 gender
  created() {
    console.log(this.$attrs); // 4️⃣ { name: 'kangw3n' }
  }
});

Vue.component('fake-user', {
  template: '<child v-bind="$attrs"></child>', // 2️⃣ 將 $attrs 綁定到所有的 <child />
  props: ['age'],
  created() {
    console.log(this.$attrs); // { name: 'kangw3n', gender: 'male' }
  }
});
```
由此可見，我們既可以透過 `$attrs` 不斷的往下傳遞內容，且不必個別宣告相對的 `props`， 比起自訂 `props` 好多了。

##### `$listeners`
再來就是 `$listeners`，其實跟 `$attrs` 是相似的概念，只是 `$listener` 是針對事件，也就就是常用的 `$emit`：

> 💡  `$attrs` 和 `$listeners` 單字都是複數結尾，其實就是我們認知 `props` 和 `$emit` 的組合。

```html
<!-- 4️⃣ 主階層只收到 custom 2 的log事件 -->
<div id="app">
  <fake-user 
    @custom1="console.log('custom1')"  
    @custom2="console.log('custom2')"
  ></fake-user>
</div>

<script>
Vue.component('child', {
  template: '<button @click="clickEvent">Click Me</button>',
  methods: {
    clickEvent() {
      this.$emit('custom1'); // 1️⃣ 發送特殊事件 1
      this.$emit('custom2'); // 2️⃣ 發送特殊事件 2
    }
  }
});

Vue.component('fake-user', {
  template: '<child v-on="inputListeners"></child>',
  computed: {
    inputListeners() {
      var vm = this;
      return {
        ...this.$listeners,
        custom1(event) {
          console.log('我在這裡攔截custom1並不讓他往上傳'); // 3️⃣ 把 <child /> 某事件攔截
          // 要往上傳的話可以用 vm.emit('custom1');
        }
      }
    }
  }
});

new Vue({
  el: '#app'
});

</script>
```
上述的code做了這幾件事情：
1. 在 `<child />` 元件發送了兩個特殊事件: `custom1` 和 `custom2`。
2. 在 `<fake-user />` 元件利用 `v-on` 攔截 `$listeners`，透過 `computed` 監聽並修改 `custom1` 的事件。當 `custom1` 被阻攔後沒有做 `vm.$emit('')` 的動作，該事件則完全不會再往上傳遞。
3. 在 `#app` 父層元件選擇在接收到這兩個事件時，只能監聽 `custom2` 發送出來的事件，因為 `custom1` 已在 `<fake-user />` 層被攔截了。

> 💡 如果要原封不動往上傳遞子階層事件，我們可以透過 `v-on="$listeners"`直接傳遞即可。

```html
<!-- 4️⃣ 主階層custom 1 和custom 2 的log事件 -->
<div id="app">
  <fake-user 
    @custom1="console.log('custom1')"  
    @custom2="console.log('custom2')"
  ></fake-user>
</div>

<script>
Vue.component('child', {
  template: '<button @click="clickEvent">Click Me</button>',
  methods: {
    clickEvent() {
      this.$emit('custom1'); // 1️⃣ 發送特殊事件 1
      this.$emit('custom2'); // 2️⃣ 發送特殊事件 2
    }
  }
});

Vue.component('fake-user', {
  template: '<child v-on="$listeners"></child>'  // 3️⃣ 把 <child /> 事件都往上傳遞
});

new Vue({
  el: '#app'
});

</script>
```


從 `$listeners` 就可以把所有子元件相關的事件都往上拋，中間可以依據條件的設定做攔截，修改之間的溝通協定，比起要一層一層傳來的簡單多了，也不需要開啟 `EventBus` 去佔用多餘的實例。

### 跨元件溝通總結，哪個比較好？

> *其實還有另一種資料傳遞方式，則是透過 `localstorage` 瀏覽器端做資料傳遞的動作，但這部分我覺得使用率太低了，而且要把它變成動態的狀態更是麻煩，所以在此就不討論該技術。*

我相信每一種做法都有它的優缺點，這就得看各位大大在建立專案的需求而定了，簡單的話用 `props` 跟 `$emit` 即可，複雜或是大型專案或許使用 `Vuex` 更為方便及維護性高，當然還有其他方法可以參考的方法像是 `inject / provide` 和 `$attrs / $listeners` 也是個不錯的選擇。

*本文參考了對岸的[掘金](https://juejin.im/post/5d267dcdf265da1b957081a3)文章再進行修改，根據自己的經驗加了一些有趣的案例。*

---
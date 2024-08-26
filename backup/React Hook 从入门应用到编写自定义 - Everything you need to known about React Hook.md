## 前言

每次提及 `Hook` 时，只能说得出 `useState` 以及 `useEffect` 来。是得好好总结总结，全面认识 `Hook` 。

本文分两大板块：

1. `Hook` 基础，官方 `Hook` 的使用；
2. 自定义 `Hook` ，常见 `Hook` 的实现。

[点击查看>>>代码托管地址](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fshiyou00%2Ffhooks "https://github.com/shiyou00/fhooks")

## Hook 基础

`React Hooks` 的意思是，组件尽量写成纯函数，如果需要外部功能和副作用，就用钩子把外部代码"钩"进来。 `React Hooks` 就是那些钩子。

看一个简单的例子：

```typescript
function BaseHook(){
    const [name,setName] = useState('jack');

    const handleClick = ()=>{
        setName('frank');
    }

    return (
        <div>
            {name}
            <button onClick={handleClick}>设置姓名</button>
        </div>
    )
}
```

原本函数组件是无状态的，现在利用 `useState Hook` ，你会发现它拥有了类似 `Class` 组件中 `state` 了。

### Hook 解决什么问题

在类组件中有以下几个问题：

* 状态逻辑难复用，通常使用 `render props` （渲染属性）或者 `HOC` （高阶组件）来处理；
* 类组件中到处都是对状态的访问和处理，导致组件难以拆分成更小的组件；
* 事件 `this` 指向问题。

`Hook` 的优点：

* 更好的逻辑复用；
* 副作用的关注点分离；
* `Hook` 在 `function` 组件中使用，不用维护复杂的生命周期，不用担心 `this` 指向问题。

### Hook 使用规则

* 只能在函数内部的最外层调用 `Hook` ，不要在循环、条件判断或者子函数中调用；
* 只能在 `React` 的函数组件中调用 `Hook` ，不要在其他 `JavaScript` 函数中调用。

```typescript
function BaseHook(){
    let name,setName;
    if(true){
        [name,setName] = useState('jack');
    }
    const handleClick = ()=>{
        setName('frank');
    }
    return (
        <div>
            {name}
            <button onClick={handleClick}>设置姓名</button>
        </div>
    )
}
```

编译结果：
![image.png](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e69455812a24337a184b54621e99084~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)
这是 `eslint` 插件： `eslint-plugin-react-hooks` 做的校验。

### useState

`useState()` 用于为函数组件引入状态（ `state` ）。纯函数不能有状态，所以把状态放在钩子里面。

```typescript
const [state, setState] = useState(initialState);
```

特性：

* （1） `initialState` 参数只会在组件的初始化渲染中起作用，后续渲染时会被忽略；
* （2）如果初始 `state` 需要通过复杂计算获得，则可以传入一个函数，在函数中计算并返回初始的 `state` ，此函数只在初始渲染时被调用。

[1] 实例：

```typescript
function UseStateHook(){
    const [count, setCount] = useState(0);

    return (
      <div>
          <p>You clicked {count} times</p>
          <button onClick={() => setCount(count + 1)}>
              Click me
          </button>
      </div>
    )
}
```

代码解析：

* 声明一个叫 `"count"` 的 `state` 变量，初始值为0，后续通过 `setCount` 改变它能让视图重新渲染；
* 初始化时 `count=0` 只会在组件的初始渲染中起作用，后续渲染时会被忽略。

[2] 计算 `state`：

```typescript
function ComputeState(){
    const [count,setCount] = useState(()=>{
        const newCount = Math.random();
        return newCount;
    });
    return (
        <div>
            <p>You clicked {count} times</p>
            <button onClick={() => setCount(count + 1)}>
                Click me ComputeState
            </button>
        </div>
        )
}
```

### useReducer

`React`本身不提供状态管理功能，通常需要使用外部库，这方面最常用的库是 `Redux` 。

`Redux` 的核心概念是，组件发出 `action` 与状态管理器通信。状态管理器收到 `action` 以后，使用 `Reducer` 函数算出新的状态， `Reducer` 函数的形式是 `(state, action) => newState` 。

`useReducer` 是一个用于状态管理的 `Hook Api` 。是 `useState` 的替代方案。
那么 `useReducer` 和 `useState` 的区别是什么呢？答案是 `useState` 是使用 `useReducer` 构建的。

```typescript
const [state, dispatch] = useReducer(reducer, initialState);
```

上面是 `useReducer()` 的基本用法，它接受 `Reducer` 函数和状态的初始值作为参数，返回一个数组。数组的第一个成员是状态的当前值，第二个成员是发送 `action`  的 `dispatch` 函数。

计数器示例：

```typescript
const initialState = 0;

const reducer = (state, action) => {
  switch (action) {
    case 'increment':
      return state + 1
    case 'decrement':
      return state - 1
    case 'reset':
      return initialState
    default:
      return state
  }
}

function UseReducerHook (){
    const [count, dispatch] = useReducer(reducer, initialState);
    return (
        <div>
            <div>Count - {count}</div>
            <button onClick={() => dispatch('increment')}>增加</button>
            <button onClick={() => dispatch('decrement')}>减少</button>
            <button onClick={() => dispatch('reset')}>重置</button>
      </div>
    )
}
```

![count.gif](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d40a4e684ab34f2eb4010c6551fd622e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

由于 `Hooks` 可以提供共享状态和 `Reducer` 函数，所以它在这些方面可以取代 `Redux` 。但是它没法提供中间件（ `middleware` ）这样高级功能。

### useContext

我们知道 `React` 提供了 `context` ，让我们在层级很深的组件中共享状态。在函数组件中就是使用 `useContext` 。

还是上面那个计数器的案例，我们使用 `useContext` 来改造下：

```typescript
const AppContext = React.createContext({});

const initialState = 0;

const reducer = (state, action) => {
  switch (action) {
    case 'increment':
      return state + 1
    case 'decrement':
      return state - 1
    case 'reset':
      return initialState
    default:
      return state
  }
}

function ShowCount(){
    const { count } = useContext(AppContext);
    return (
        <div>Count - {count}</div>
    )
}

function Action(){
    const { dispatch } = useContext(AppContext);
    return (
        <>
            <button onClick={() => dispatch('increment')}>增加</button>
            <button onClick={() => dispatch('decrement')}>减少</button>
            <button onClick={() => dispatch('reset')}>重置</button>
        </>
    )
}

function UseReducerHook (){
    const [count, dispatch] = useReducer(reducer, initialState);

    return (
        <AppContext.Provider value={{
            count,
            dispatch
          }}>
            <div>
                <ShowCount />
                <Action />
            </div>
        </AppContext.Provider>
    )
}
```

这里主要是把展示数字和操纵按钮分离成不同的两个组件，并且在父组件中创建一个 `context` ，下发 `count` 与 `dispatch` 。这样所有子组件，孙子组件都可以共享 `context` 中数据。

### useEffect

`useEffect` 就是一个 `Effect Hook` ，给函数组件增加了操作副作用的能力。它跟 `class` 组件中的 `componentDidMount` 、 `componentDidUpdate`  和 `componentWillUnmount` 具有相同的用途，只不过被合并成了一个 `API` 。

```typescript
useEffect(()  =>  {
  // Async Action
}, [dependencies])
```

* 第一个参数是一个函数，异步操作的代码放在里面。
* 第二个参数是一个数组，用于给出 `Effect` 的依赖项，只要这个数组发生变化， `useEffect()` 就会执行。第二个参数可以省略，这时每次组件渲染时，就会执行 `useEffect()` 。
* `React` 将在 `DOM` 更新之后调用它， `React` 保证了每次运行 `useEffect` 的时候， `DOM` 已经更新完毕。

示例：

```typescript
import React,{useState,useEffect} from 'react';

const Person = ({ personId }) => {
    const [loading, setLoading] = useState(true);
    const [person, setPerson] = useState({});

    useEffect(() => {
      setLoading(true);
      fetch(`https://v1/api/people/${personId}/`) // 这是一个虚拟的请求
        .then(response => response.json())
        .then(data => {
          setPerson(data);
          setLoading(false);
        });
    }, [personId]);

    if (loading === true) {
      return <p>Loading ...</p>;
    }

    return (
      <div>
        <p>You're viewing: {person.name}</p>
        <p>Height: {person.height}</p>
        <p>Mass: {person.mass}</p>
      </div>
    );
  };

function UseEffectHook(){
    const [show, setShow] = useState("1");
    return (
        <>
            <Person personId={show} />
            <div>
                Show:
                <button onClick={() => setShow("1")}>button 1</button>
                <button onClick={() => setShow("2")}>button 2</button>
            </div>
        </>
    )
}

export default UseEffectHook;
```

`Person` 组件参数 `personId` 发生变化， `useEffect()` 就会执行。组件第一次渲染时， `useEffect()` 也会执行。

#### 需要清除的 effect

在 `class` 组件中，我们去监听原生 `DOM` 事件时会在 `componentDidMount` 这个生命周期中去做，因为在这里可以获取到已经挂载的真实 `DOM` 。我们也会在组件卸载的时候去取消事件监听避免内存泄露。那么在 `useEffect` 中该如何实现呢？

```typescript
useEffect(() => {
  function handleClick(status) {
    document.title = `You clicked ${count} times`;
  }

  document.body.addEventListener("click",handleClick,false);

  return function cleanup() {
    document.body.removeEventListener("click",handleClick,false);
  };
});
```

通过在 `useEffect` 中返回一个函数，它便可以清理副作用。

清理规则是：

* **首次渲染不会进行清理，会在下一次渲染，清除上一次的副作用；**
* **卸载阶段也会执行清除操作。*

### useCallback

把内联回调函数及依赖项数组作为参数传入 `useCallback` ，它将返回该回调函数的 `memoized` 版本，该回调函数仅在某个依赖项改变时才会更新。

```typescript
const memoizedCallback = useCallback(
  () => {
    doSomething(a);
  },
  [a],
);
```

通俗来讲当参数 `a` 发生变化时，会返回一个新的函数引用赋值给 `memoizedCallback` 变量，因此这个变量就可以当做 `useEffect` 第二个参数。这样就有效的将逻辑分离出来。

针对上述请求数据例子，我们使用 `useCallback` 改写下：

```typescript
const Person = ({ fetchData }) => {
    const [loading, setLoading] = useState(true);
    const [person, setPerson] = useState({});

    useEffect(() => {
      setLoading(true);
       fetchData().then(response => response.json())
        .then(data => {
          setPerson(data);
          setLoading(false);
        });
    }, [fetchData]); //{2}

    if (loading === true) {
      return <p>Loading ...</p>;
    }

    return (
      <div>
        <p>You're viewing: {person.name}</p>
        <p>Height: {person.height}</p>
        <p>Mass: {person.mass}</p>
      </div>
    );
  };

function UseEffectHook(){
    const [personId, setPersonId] = useState("1");

    const fetchData = useCallback(()=>{
       return fetch(`https://v1/api/people/${personId}/`);
    },[personId]);//{1}

    return (
        <>
            <Person fetchData={fetchData} />
            <div>
                Show:
                <button onClick={() => setPersonId("1")}>button 1</button>
                <button onClick={() => setPersonId("2")}>button 2</button>
            </div>
        </>
    )
}
```

经过 `useCallback` 包装过的函数可以当作普通变量作为 `useEffect` 的依赖。 `useCallback`做的事情，就是在其依赖变化时，返回一个新的函数引用，触发 `useEffect` 的依赖变化，并激活其重新执行。

现在我们不需要在 `useEffect` 依赖中直接对比 `personId` 参数了，而可以直接对比 `fetchData` 函数。 `useEffect` 只要关心 `fetchData` 函数是否变化，而 `fetchData` 参数的变化在 `useCallback` 时关心，能做到：依赖不丢、逻辑内聚，从而容易维护。

### useMemo

把"创建"函数和依赖项数组作为参数传入 `useMemo` ，它仅会在某个依赖项改变时才重新计算 `memoized` 值。 **这种优化有助于避免在每次渲染时都进行高开销的计算**。

跟 `useCallback` 非常类似的功能， `useMemo` 相当于 `Class` 组件的 `pureComponent` 。

我们再接着上述案例使用 `useMemo` 修改下：

```typescript
const fetchData = useMemo(()=>{
	return ()=>fetch(`https://v1/api/people/${personId}/`);
},[personId]);
```

其余都不变，唯独多加了一层 `()=>fn` 结构。

**如果没有提供依赖项数组， `useMemo` 在每次渲染时都会计算新的值。**

### useRef

学习 `useRef` 之前，我们先搞清楚几个问题。

1. `Refs` 是什么；
2. 类组件如何创建 `Refs` ；
3. `forwardRef` ；
4. 函数组件如何创建 `Refs` 。

#### Refs

`Refs` 是一个获取 `DOM` 节点或 `React` 元素实例的工具。在 `React` 中 `Refs` 提供了一种方式，允许用户访问 `DOM` 节点或 `render` 方法中创建的 `React` 元素。

#### 类组件如何创建 ref

* 当 `ref` 属性用于 `HTML` 元素时，构造函数中使用 `React.createRef()` 创建的 `ref` 接收底层 `DOM` 元素作为其 `current` 属性；
* 当 `ref` 属性用于自定义 `class` 组件时， `ref` 对象接收组件的挂载实例作为其 `current` 属性；
* **你不能在函数组件上使用 `ref` 属性**，因为他们没有实例。

```typescript
class Child extends React.Component{
    render() {
        return <div>Child</div>;
    }
}

class ClassRefComp extends React.Component {
    constructor(props) {
      super(props);
      this.myRef = React.createRef();
    }
    componentDidMount(){
        console.log(this.myRef.current);
    }
    render() {
      return (
          <>
            {/* this.myRef.current 获取到 Child 实例 */}
            <Child ref={this.myRef} />
            {/* this.myRef.current 获取 div 元素 */}
            {/* <div ref={this.myRef} /> */}
          </>
      )
    }
}
```

**不能在函数组件上使用 `ref` 属性**

例如下面做会报错：

```typescript
function Child2(){
    return (
        <div>不能在函数组件上使用 ref 属性</div>
    )
}

class ClassRefComp extends React.Component {
    constructor(props) {
      super(props);
      this.myRef = React.createRef();
    }
    componentDidMount(){
        console.log(this.myRef.current);
    }
    render() {
      return (
          <>
            <Child2 ref={this.myRef} />
          </>
      )
    }
}
```

`Child2` 是函数组件，不能使用 `ref` 属性，因为他们没有实例，解决方案：

1. 改成 `class` 组件
2. `React.forwardRef` 进行包装

#### React.forwardRef

```typescript
let Child2 = (props,ref)=>{
    return (
        <div ref={ref}>不能在函数组件上使用 ref 属性</div>
    )
}

Child2 = React.forwardRef(Child2);
```

代码解释：

1. 我们通过调用 `React.createRef` 创建了一个 `React ref` 并将其赋值给 `myRef` 变量；
2. 我们通过指定 `ref` 为 `JSX` 属性，将其向下传递给 `<Child2 ref={this.myRef}>` ；
3. `React` 传递 `ref` 给 `forwardRef` 内函数 `(props, ref) => ...`，作为其第二个参数；
4. 我们向下转发该 `ref` 参数到 `<div ref={ref}>`，将其指定为 `JSX` 属性；
5. 当 `ref` 挂载完成， `ref.current` 将指向 `<div>` DOM 节点。

`React.forwardRef` **解决了，函数组件没有实例，无法像类组件一样可以接收 `ref` 属性的问题**

到这里想必你也应该清楚 `Refs` 是什么以及在类组件中如何使用了，至于它的一些其他用法例如"回调 `Refs` "、"高阶组件中转发 `Refs` "这里就不一一讲解了。

#### 函数组件如何创建ref

`useRef` 返回一个可变的 `ref` 对象，其 `current` 属性被初始化为传入的参数（ `initialValue` ）。返回的 `ref` 对象在组件的整个生命周期内保持不变。

`useRef` 的作用：

* 获取 `DOM` 元素的节点；
* 获取子组件的实例；
* 渲染周期之间共享数据的存储（ `state` 不能存储跨渲染周期的数据，因为 `state` 的保存会触发组件重渲染）。

```typescript
function FuncRefComp(){
    const inputRef = useRef(null);

    const handleChange = ()=>{
        console.log(inputRef.current.value);
    }

    return (
        <input ref={inputRef} type="text" onChange={handleChange} />
    )
}
```

### useImperativeHandle（不常用）

* `useImperativeHandle` 可以让你在使用 `ref` 时，自定义暴露给父组件的实例值，不能让父组件想干嘛就干嘛；
* 在大多数情况下，应当避免使用 `ref` 这样的命令式代码。 `useImperativeHandle` 应当与 `forwardRef` 一起使用；
* 父组件可以使用操作子组件中的多个 `ref` 。

```typescript
useImperativeHandle(ref, createHandle, [deps])
```

示例：

```typescript
function UseImperativeHandleRef(props, ref) {
    const inputRef = useRef();
    useImperativeHandle(ref, () => ({
      focus: () => {
        inputRef.current.focus();
      }
    }));
    return <input ref={inputRef} />;
}

UseImperativeHandleRef = forwardRef(UseImperativeHandleRef);

function UseRefHook(){
    const inputRef = useRef();
    useEffect(()=>{
        console.log(inputRef.current);
    })
    return <UseImperativeHandleRef ref={inputRef} />
}
```

在本例中，渲染 `<UseImperativeHandleRef ref={inputRef} />` 的父组件可以调用 `inputRef.current.focus()`。

### useLayoutEffect（不常用）

* `useEffect` 在全部渲染完毕后才会执行；
* `useLayoutEffect` 会在 浏览器 `layout` 之后， `painting` 之前执行；
* 使用方式同 `useEffect` 相同，但它会在所有的 `DOM` 变更之后同步调用 `effect` ；
* 可以使用它来读取 `DOM` 布局并同步触发重渲染；
* 在浏览器执行绘制之前 `useLayoutEffect` 内部的更新计划将被同步刷新；
* 尽可能使用标准的 `useEffect` 以避免阻塞视图更新。

![image.png](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3349211bcae54a65a7ee28958fdb1689~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

## 自定义 Hook

如果函数的名字以 `use` 开头，并且调用了其他的 `Hook` ，则就称其为一个自定义 `Hook` 。

`Hook` **是一种复用状态逻辑的方式，它不复用 `state` 本身，事实上 `Hook` 的每次调用都有一个完全独立的 `state` 。**

简单的自定义 `Hook` ：

```typescript
function useTitle(title){
    useEffect(()=>{
        document.title = title;
    },[title]);
}
```

使用：

```typescript
function CustomHook (){
    useTitle('my use title');
    return (
        <div>CustomHook</div>
    )
}
```

我们来看一个稍微复杂点的例子，通过自定义 `hook` 实现 `input` 双向数据绑定。

### input 实现双向数据绑定

```typescript
function useBind(initVal){
    let [value,setValue] = useState(initVal);
    let onChange = function(event){
        setValue(event.currentTarget.value);
    }
    return {
        value,
        onChange
    }
}
```

使用：

```typescript
function CustomHook (){
    const valueObj = useBind("");
    return <input {...valueObj} />
}
```

写到这里应该可以感受到 `hook` 的逻辑复用的能力。

对比 `HOC` ，大量使用  `HOC` 的情况下让我们的代码变得嵌套层级非常深，使用自定义 `hook` ，我们可以实现扁平式的状态逻辑复用，而避免了大量的组件嵌套。

既然自定义 `Hook` 这么香，那么有什么优秀的轮子值得我们来深入学习吗？

阿里开源的 `ahooks`  。它是一个 `React Hooks` 库，致力提供常用且高质量的 `Hooks` 。

首先安装： `npm install ahooks --save` 

```typescript
import React from "react";
import { useToggle } from "ahooks";

function AHooks(){
    const [ state, { toggle } ] = useToggle();
    return (
        <div>
        <p>Current Boolean: {String(state)}</p>
        <p>
            <button onClick={() => toggle()}>Toggle</button>
        </p>
        </div>
    );
}

export default AHooks;
```

它的使用可以自行查阅文档，我们今天挑选几个常用 `Hook` 来分析其源码。

### useUpdate

强制组件重新渲染的 `hook` 

```typescript
const useUpdate = () => {
    const [, setState] = useState(0);//{1}

    return useCallback(() => setState((num) => {return num + 1}));//{2}
};
```

分析：

* {1} 定义一个初始状态；
* {2} 调用 `setState` 方法则会导致 `state` 变化，从而刷新组件。

### useUpdateEffect

一个只在依赖更新时执行的 `useEffect hook` 。使用上与 `useEffect` 完全相同，只是它忽略了首次渲染，且只在依赖项更新时运行。

```typescript
const useUpdateEffect = (effect, deps) => {
    const isMounted = useRef(false); //{1}

    useEffect(() => {
      // {2}
      if (!isMounted.current) {
        isMounted.current = true;
      } else {
        return effect();
      }
    }, deps);
 };
```

解析：

* {1} 使用 `useRef` 存一个布尔值；
* {2} 当首次执行 `useEffect` 时，设置其值为 `true` 。再次执行时就可以执行 `effect` 回调函数。

### usePersistFn

持久化 `function` 的 `Hook` ，在某些场景中，你可能会需要用 `useCallback` 记住一个回调，但由于内部函数必须经常重新创建，记忆效果不是很好，导致子组件重复 `render` 。对于超级复杂的子组件，重新渲染会对性能造成影响。通过 `usePersistFn` ，可以保证函数地址永远不会变化。

```typescript
  function usePersistFn(fn) {
    const ref = useRef(() => {
      throw new Error('Cannot call function while rendering.');
    });

    ref.current = fn;

    const persistFn = useCallback(((...args) => ref.current(...args)), [ref]);

    return persistFn;
  }
```

解析： `useCallback` 的第一个参数传入 `ref` ，由于 `ref` 在整个生命周期内是不会发生变化的，因此 `useCallback` 的返回值不会更新。

### useMount and useUnmount

组件挂载和组件卸载生命周期 `Hook` 。

使用示例：

```typescript
const MyComponent = () => {
    useMount(() => {
      console.log('mount'); // 挂载时触发
    });
    useUnmount(() => {
        console.log('unmount'); // 卸载时触发
    });
    return <div>Hello World</div>;
  };

function CustomHook (){
    const [state, { toggle }] = useToggle(false);
    return (
        <div>
            <button type="button" onClick={() => toggle()}>
                {state ? 'unmount' : 'mount'}
            </button>
            {state && <MyComponent />}
        </div>
    )
}
```

源码分析：

```typescript
# mount
const useMount = (fn) => {
  const fnPersist = usePersistFn(fn);

  useEffect(() => {
    if (fnPersist && typeof fnPersist === 'function') {
      fnPersist();
    }
  }, []);
};

# unmount
const useUnmount = (fn) => {
  const fnPersist = usePersistFn(fn);

  useEffect(
    () => () => {
      if (fnPersist && typeof fnPersist === 'function') {
        fnPersist();
      }
    },
    [],
  );
};
```

以上是稍微简单的 `hook` 编写，通过编写这些 `hook` ，至少让我们知道，自定义 `hook` 并没有想象的那么复杂，接下来看几个有点难度的 `hook` 。

### useDebounce

用来处理防抖值的 `Hook` 。

示例： `DebouncedValue` 只会在输入结束 `500ms` 后变化。

```typescript
import React,{useState} from "react";
import {useDebounce} from "./customHooks";

function DebounceHook (){
    const [value, setValue] = useState();
    const debouncedValue = useDebounce(value,  500 );
    return (
        <div>
            <input
                value={value}
                onChange={(e) => setValue(e.target.value) }
                placeholder="Typed value"
                style={{ width: 280 }}
            />
            <p style={{ marginTop: 16 }}>DebouncedValue: {debouncedValue}</p>
        </div>
    )
}

export default DebounceHook;
```

实现：

```typescript
  const useDebounceFn = (fn,wait)=>{
      const _wait =  wait || 0;

      const timer = useRef();
      const fnRef = useRef(fn);
      fnRef.current = fn;

      const cancel = useCallback(()=>{
        if(timer.current){
            clearTimeout(timer.current);
        }
      },[])

      const run = useCallback((...args)=>{
          cancel();
          timer.current = setTimeout(()=>{
            fnRef.current(...args);
          },_wait)
      },[_wait,cancel])

      useEffect(()=> cancel,[]);

      return {
          run,
          cancel
      }
  }

  const useDebounce =(value,wait)=>{
    const [debounced, setDebounced] = useState(value);

    const { run } = useDebounceFn(() => {
        setDebounced(value);
      }, wait);

      useEffect(() => {
        run();
      }, [value]);

      return debounced;
  }
```

解析：

* `useDebounce` 收到 `value` 值变化时，会调用 `useEffect` 钩子；
* `useEffect` 钩子调用 `run` 方法，也就是 `useDebounceFn` 这个钩子返回的；
* `run` 方法中实现了防抖的核心功能。

### useThrottle

用来处理节流值的 `Hook` 。节流的处理，一定时间内只触发一次。

示例： `ThrottledValue` 每隔 `500ms`  变化一次。

```typescript
import React,{useState} from "react";
import {useThrottle} from "./customHooks";

function ThrottleHook (){
    const [value, setValue] = useState();
    const throttledValue = useThrottle(value,  500 );
    return (
        <div>
            <input
                value={value}
                onChange={(e) => setValue(e.target.value)}
                placeholder="Typed value"
                style={{ width: 280 }}
            />
            <p style={{ marginTop: 16 }}>throttledValue: {throttledValue}</p>
        </div>
    )
}

export default ThrottleHook;
```

实现：

```typescript
  const useThrottleFn = (fn,wait)=>{
    const _wait =  wait || 0;
    const timer = useRef();
    const fnRef = useRef(fn);
    fnRef.current = fn;

    const currentArgs = useRef([]);

    const cancel = useCallback(()=>{
      if(timer.current){
          clearTimeout(timer.current);
      }
      timer.current = undefined;
    },[])

    const run = useCallback((...args)=>{
        currentArgs.current = args;
        if(!timer.current){
            timer.current = setTimeout(()=>{
                fnRef.current(...args);
                timer.current = undefined;
            },_wait)
        }
    },[_wait,cancel])

    useEffect(()=> cancel,[]);

    return {
        run,
        cancel
    }
}

const useThrottle =(value,wait)=>{
    const [throttled, setThrottled] = useState(value);

    const { run } = useThrottleFn(() => {
        setThrottled(value);
      }, wait);

      useEffect(() => {
        run();
      }, [value]);

      return throttled;
}
```

解析：

* `useThrottle` 收到 `value` 值变化时，会调用 `useEffect` 钩子；
* `useEffect` 钩子调用 `run` 方法，也就是 `useThrottleFn` 这个钩子返回的；
* `run` 方法中实现了节流的核心方法，每次判断定时器是否开启，如果没有开启则开启定时器执行函数；
* 因此不论输入频率多快，都是每隔一定时间才会触发函数的执行。

## 总结

通过本文，希望您可以快速掌握官方提供的常用 `Hook` 的使用，以及编写自定义 `Hook` 进行逻辑封装。

* [前言](#heading-0 "前言")
* [Hook 基础](#heading-1 "Hook 基础")
  - [Hook 解决什么问题](#heading-2 "Hook 解决什么问题")
  - [Hook 使用规则](#heading-3 "Hook 使用规则")
  - [useState](#heading-4 "useState")
  - [useReducer](#heading-5 "useReducer")
  - [useContext](#heading-6 "useContext")
  - [useEffect](#heading-7 "useEffect")
    + [需要清除的 effect](#heading-8 "需要清除的 effect")
  - [useCallback](#heading-9 "useCallback")
  - [useMemo](#heading-10 "useMemo")
  - [useRef](#heading-11 "useRef")
    + [Refs](#heading-12 "Refs")
    + [类组件如何创建 ref](#heading-13 "类组件如何创建 ref")
    + [React.forwardRef](#heading-14 "React.forwardRef")
    + [函数组件如何创建ref](#heading-15 "函数组件如何创建ref")
  - [useImperativeHandle（不常用）](#heading-16 "useImperativeHandle（不常用）")
  - [useLayoutEffect（不常用）](#heading-17 "useLayoutEffect（不常用）")
* [自定义 Hook](#heading-18 "自定义 Hook")
  - [input 实现双向数据绑定](#heading-19 "input 实现双向数据绑定")
  - [useUpdate](#heading-20 "useUpdate")
  - [useUpdateEffect](#heading-21 "useUpdateEffect")
  - [usePersistFn](#heading-22 "usePersistFn")
  - [useMount and useUnmount](#heading-23 "useMount and useUnmount")
  - [useDebounce](#heading-24 "useDebounce")
  - [useThrottle](#heading-25 "useThrottle")
* [总结](#heading-26 "总结")
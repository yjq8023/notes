### 类组件中的state

#### setState基本用法

```js
setState(obj, callback)
```

+ 第一个参数：当obj为一个对象，则会合并到当前state中。如果是一个函数，那么当前组件的state和props将作为参数，函数返回值用于合并新的state
+ 第二参数：可以理解成setState操作完成后的回掉函数，可以获取到更新后的state值

##### setState调用栈

- 首先，setState 会产生当前更新的优先级（老版本用 expirationTime ，新版本用 lane ）。
- 接下来 React 会从 fiber Root 根部 fiber 向下调和子节点，调和阶段将对比发生更新的地方，更新对比 expirationTime ，找到发生更新的组件，合并 state，然后触发 render 函数，得到新的 UI 视图层，完成 render 阶段。
- 接下来到 commit 阶段，commit 阶段，替换真实 DOM ，完成此次更新流程。
- 此时仍然在 commit 阶段，会执行 setState 中 callback 函数,如上的`()=>{ console.log(this.state.number) }`，到此为止完成了一次 setState 全过程

### 函数组件中的state

##### useState基本用法

```js
const [ name, setName] = useState(defaultValue);
const [ number , setNumber ] = React.useState(()=>{
       /*  props 中 a = 1 state 为 0-1 随机数 ， a = 2 state 为 1 -10随机数 ， 否则，state 为 1 - 100 随机数   */
       if(props.a === 1) return Math.random() 
       if(props.a === 2) return Math.ceil(Math.random() * 10 )
       return Math.ceil(Math.random() * 100 ) 
    })
```

> setState上下文中无法获取当前的state，设置前后两个state会进行浅比较，如果相同，会不发起更新

### 基础原理

这里就要提前聊一下事件系统了。正常 **state 更新**、**UI 交互**，都离不开用户的事件，比如点击事件，表单输入等，React 是采用事件合成的形式，每一个事件都是由 React 事件系统统一调度的，那么 State 批量更新正是和事件系统息息相关的。下面是事件触发的精简源码

```js
function batchedEventUpdates(fn,a){
    /* 开启批量更新  */
   isBatchingEventUpdates = true;
  try {
    /* 这里执行了的事件处理函数， 比如在一次点击事件中触发setState,那么它将在这个函数内执行 */
    return batchedEventUpdatesImpl(fn, a, b);
  } finally {
    /* try 里面 return 不会影响 finally 执行  */
    /* 完成一次事件，批量更新  */
    isBatchingEventUpdates = false;
  }
}
```

可以看到每次事件触发前都会把isBatchingEventUpdates开启，结束后关闭。

这个参数正是表示setState是否批量更新。

> 通过事件触发的setState是批量更新的，也就是异步的。而非事件触发的setState，则是同步的

### 实践与应用

### 
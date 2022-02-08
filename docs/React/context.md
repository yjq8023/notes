### context基本概念

React context是为了解决特定场景下，一个属性想要在子组件中使用，需要经过父组件层层传递。它的功能就是实现无需为每层组件添加props，就能在组件树件进行数据传递。

### context使用

#### 创建context对象

```js
const context = React.createContext(null);
const Provider = context.Provider
const Consumer = context.Consumer
```



#### 提供者

```react
const Provider = context.Provider // 接上诉创建对象。。。
function Component1() {
 const [data, setData] = useState({})
  return  (
    <Provider value={data}> // 重点在这，把提供者作为包裹组件，其内子组件可以作为消费者
      <Component2></Component2>
      <button onClick={() => setData({time: Date.now()})}>set new value</button>
    </Provider>
  )
}
```



#### 消费者

1. ##### contextType

   ```react
   class Component2 extends React.Component{
     render() {
       console.log('this.context');
       console.log(this.context); // 直接使用
       return (
         <div>
           <Component3></Component3>
         </div>
       )
     }
   }
   Component2.contextType = context // 重点在这，把context挂载到contextType属性下，组件内可以直接用context数据
   ```

   

2. ##### useContext

   ```react
   function Component3() {
     const contextValue = useContext(context) // 直接使用useContext获取数据
     return (
       <div>
         <h1>Component3</h1>
         <p>{JSON.stringify(contextValue)}</p>
       </div>
     )
   }
   ```

   

3. ##### Consumer（这里跟前两者有差异，context内容改变时，仅Consumer组件内children函数重渲染，Component4本身不会重渲染）

   ```react
   function Component4() {
     const contextValue = useContext(context)
     return (
       <div>
         // Consumer作为组件使用，其内可以获取到context值
         <Consumer>
           { (contextValue2) => {
             return <p>{JSON.stringify(contextValue2)}</p>
           }}
         </Consumer>
       </div>
     )
   }
   ```

   
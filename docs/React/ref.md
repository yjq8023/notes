### ref基本概念和使用

#### ref对象创建

标准的ref对象如下结构

```js
{
	current: null
}
```

通过createRef跟useRef都可以创建，两者区别在于函数组件没有实例，所以useRef关联的是函数组件的fiber对象

#### ref属性的处理逻辑

1. 值为字符串

   把当前组件关联到this.refs下

2. 值为函数

   函数第一个参数为实例

3. 值为ref对象

   实例挂载到ref对象的current属性下

### ref高阶用法

#### 1. forwardRef

forwardRef的初衷就是解决ref不能跨层传递的问题。forwardRef接受父级元素标记的ref信息，子组件通过props接受上一层的ref。

##### forwardRef跨层级获取ref

```react
// 孙组件
function Son (props){
    const { grandRef } = props
    return <div>
        <div> i am alien </div>
        <span ref={grandRef} >这个是想要获取元素</span>
    </div>
}
// 父组件
class Father extends React.Component{
    constructor(props){
        super(props)
    }
    render(){
        return <div>
            <Son grandRef={this.props.grandRef}  />
        </div>
    }
}
const NewFather = React.forwardRef((props,ref)=> <Father grandRef={ref}  {...props} />)
```



##### forwardRef合并转发ref

通过修改forwardRef.current值，可以做到一个ref对象指向多个想要关联的实例

```react
componentDidMount(){
  const { forwardRef } = this.props // forwardRef传递下来的属性，参考第一个用例
  forwardRef.current={
     form:this.form,      // 给form组件实例 ，绑定给 ref form属性 
     index:this,          // 给index组件实例 ，绑定给 ref index属性 
     button:this.button,  // 给button dom 元素，绑定给 ref button属性 
  }
}
```



##### forwardRef高阶组件转发ref

```react
function HOC(Component){
  class Wrap extends React.Component{
     render(){
        const { forwardedRef ,...otherprops  } = this.props
        return <Component ref={forwardedRef}  {...otherprops}  />
     }
  }
  return  React.forwardRef((props,ref)=> <Wrap forwardedRef={ref} {...props} /> ) 
}
```



#### 2. ref实现组件通信

##### 类组件实现通信

通过ref获取实例后，调用实例内的方法修改数据，达到通信目的

```react
class Son extends React.PureComponent{
    state={
       fatherMes:'',
       sonMes:''
    }
    fatherSay=(fatherMes)=> this.setState({ fatherMes  }) /* 提供给父组件的API */
    render(){
        return <div className="sonbox" ></div>
    }
}
export default function Father(){
    const sonInstance = React.useRef(null) /* 用来获取子组件实例 */
    const toSon =()=> sonInstance.current.fatherSay('fatherMes') /* 调用子组件实例方法，改变子组件state */
    return <div className="box" >
        <Son ref={sonInstance} />
    </div>
}
```



##### 函数组件实现通信 forwardRef + useImperativeHandl

前面说过forwardRef可以让函数组件通过ref绑定到其fiber实例

useImperativeHandle可以修改函数组件内ref指向的内容

```react
// 子组件
function Son (props,ref) {
    const inputRef = useRef(null)
    const [ inputValue , setInputValue ] = useState('')
    useImperativeHandle(ref,()=>{
       const handleRefs = {
           onFocus(){              /* 声明方法用于聚焦input框 */
              inputRef.current.focus()
           },
           onChangeValue(value){   /* 声明方法用于改变input的值 */
               setInputValue(value)
           }
       }
       return handleRefs
    },[])
    return <div>
        <input placeholder="请输入内容"  ref={inputRef}  value={inputValue} />
    </div>
}

const ForwarSon = forwardRef(Son)
// 父组件
class Index extends React.Component{
    cur = null
    handerClick(){
       const { onFocus , onChangeValue } =this.cur
       onFocus() // 让子组件的输入框获取焦点
       onChangeValue('let us learn React!') // 让子组件input  
    }
    render(){
        return <div style={{ marginTop:'50px' }} >
            <ForwarSon ref={cur => (this.cur = cur)} />
            <button onClick={this.handerClick.bind(this)} >操控子组件</button>
        </div>
    }
}
```



#### 3. 函数组件通过ref缓存数据

其实就是利用函数组件ref指向fiber实例的特性，通过fiber对象缓存数据，这样对比使用useState存储数据，好处在于

1. 能直接修改数据，不会造成函数组件冗余的更新作用
2. useEffect ，useMemo内可以直接使用缓存的数据，因为useRef始终指向一个内存空间

```react
const toLearn = [ { type: 1 , mes:'let us learn React' } , { type:2,mes:'let us learn Vue3.0' }  ]
export default function Index({ id }){
    const typeInfo = React.useRef(toLearn[0])
    const changeType = (info)=>{
        typeInfo.current = info /* typeInfo 的改变，不需要视图变化 */
    }
    useEffect(()=>{
       if(typeInfo.current.type===1){
           /* ... */
       }
    },[ id ]) /* 无须将 typeInfo 添加依赖项  */
    return <div>
        {
            toLearn.map(item=> <button key={item.type}  onClick={ changeType.bind(null,item) } >{ item.mes }</button> )
        }
    </div>
}
```



### ref原理

#### ref处理逻辑

ref的执行会在更新时的commit阶段dom更新前后各触发一次分别对应

+ **commitDetachRef**，dom更新前触发，清空ref对象，使其为null
+ **commitAttachRef**，dom更新后触发，赋值真实元素节点给ref

#### ref执行时机

前面说到ref的处理会在react更新阶段触发，但是它不是每次fiber更新时都会触发，它的触发条件有两个

+ fiber初始化时触发
+ ref值改变时触发
### 基本概念

react中，父组件对子组件的调用，绑定在标签上的属性和方法会通过props传递给子组件

```react
/* children 组件 */
function ChidrenComponent(){
    return <div> In this chapter, let's learn about react props ! </div>
}
/* props 接受处理 */
class PropsComponent extends React.Component{
    componentDidMount(){
        console.log(this,'_this')
    }
    render(){
        const {  children , mes , renderName , say ,Component } = this.props
        const renderFunction = children[0]
        const renderComponent = children[1]
        /* 对于子组件，不同的props是怎么被处理 */
        return <div>
            { renderFunction() }
            { mes }
            { renderName() }
            { renderComponent }
            <Component />
            <button onClick={ () => say() } > change content </button>
        </div>
    }
}
/* props 定义绑定 */
class Index extends React.Component{
    state={  
        mes: "hello,React"
    }
    node = null
    say= () =>  this.setState({ mes:'let us learn React!' })
    render(){
        return <div>
            <PropsComponent  
               mes={this.state.mes}  // ① props 作为一个渲染数据源
               say={ this.say  }     // ② props 作为一个回调函数 callback
               Component={ ChidrenComponent } // ③ props 作为一个组件
               renderName={ ()=><div> my name is alien </div> } // ④ props 作为渲染函数
            >
                { ()=> <div>hello,world</div>  } { /* ⑤render props */ }
                <ChidrenComponent />             { /* ⑥render component */ }
            </PropsComponent>
        </div>
    }
}
```

如上看一下 props 可以是什么？

- ① props 作为一个子组件渲染数据源。
- ② props 作为一个通知父组件的回调函数。
- ③ props 作为一个单纯的组件传递。
- ④ props 作为渲染函数。
- ⑤ render props ， 和④的区别是放在了 children 属性上。
- ⑥ render component 插槽组件。
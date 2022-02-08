### 高阶组件概念

所谓高阶组件是指一个组件能以另一个组件为入参，返回一个经过加强的组件。常用于拓展组件功能和分层式构建组件功能。

### 两种不同的高阶组件

#### 属性代理

```react
function Hoc(TargetComponent) {
  return class Advance extends React.Component {
    state = { name: 'test' }
  	render() {
      return <TargetComponent {...this.props} {...this.state}></TargetComponent>
    }
  }
}
```

#### 反向继承

```react
function Hoc(TargetComponent) {
  return class Advance extends TargetComponent {
  	render() {
      const dom = super.render() // 可以直接获取TargetComponent的render内容，进一步处理
    }
  }
}
```

通过对比可以清晰看到两种方式的优缺点

属性代理可以做到跟业务组件完全解耦合，仅通过拓展props，增加渲染前后的逻辑等方式去拓展组件功能。

反向继承需要清晰知道业务组件的逻辑，通过继承业务组件，达到完全控制的目的。相对于属性代理，它的功能更强大。

### 继承静态属性

HOC的本质是返回一个新的组件，那么旧组件上绑定的静态属性会丢失掉，解决这一问题只能是通过手动把静态属性拷贝到新的组件中。方法见仁见智，可以参考使用库 hoist-non-react-statics
---
title: How to Use Classes and Sleep at Night[译]
date: 2018-05-27 19:40:21
tags:
- React
- Class
---

## 到底怎么使用继承？
看到一篇文章，介绍了如果使用继承所必须遵循的原则，故翻译学习之。
标题就是How to Use Classes and Sleep at Night，我想大概意思就是怎么正确的使用类才能在晚上睡得安稳，不然会有很多问题困扰着无法入眠吧。

[原文地址](https://medium.com/@dan_abramov/how-to-use-classes-and-sleep-at-night-9af8de78ccb4)

### 正文开始
在javascript社区中 越来越多人认为ES6的类 不是很棒


<!-- more -->
- 类 掩盖(obscure)了 JS 核心的原型继承
- 类 鼓励继承 但你应该更喜欢组合
- 类 倾向于 将你 锁定在 你想出的第一个糟糕的设计中

我认为很好的是 javascript社区 关注了 使用类和继承(inheritance)所导致的问题，但我担心初学者会感到困惑(confusingly)，因为类都是“坏”的，只是被添加到了语言中。更令人困惑的是，一些库，特别是React，在其文档中用了ES6类。React是否故意(intentionally)遵循“不好的实践”(“bad practices”)?

我认为我们正处在一个怪异(weird)的过渡时期(transitional period)，因为广泛使用类必然是一种坏事，因为它们是有局限的。它们肯定(certainly)比学习每个框架附带的新的特别类系统要好，每个框架都有自己的混合函数(mixins)形式的多继承方式。

如果你喜欢函数式编程，你可能会看到如何从 专有的类系统 转换到 “精简”ES6类(不使用混合函数，不会自动绑定等)，使我们更接近函数式解决方案。

同时(In the meantime), 这里将教你如何正确使用类并且睡得安稳：

- 抵制写你的公共API的类;

（当然导出React组件是个例外，因为他们是直接使用的）你可以始终隐藏工厂方法后面的类。如果你暴露它们，人们会以各种方式继承它们，这对你来说毫无意义，但你可能会在未来打破。避免打破别人的类是很困难的，因为他们可能会使用到你未来也会用到的方法名称，他们可能会读你的私有状态，他们可能会把自己的状态放到你的实例中，他们可能会重写你的方法而不调用超类(super)，如果他们幸运的话可能会正常运行一段时间，但后面可能会出事。也就是说，当基类和用户的派生类之间没有来回交互时，比如React组件，暴露基类可能是一个有效的API选择。


- 不要多次继承;

 继承可以方便地作为捷径使用，继承一次就是um-kay，但不要更深入.继承的问题是，后代对层次结构中每个基类的实现细节都有太多的访问权限，反之亦然(and vice versa).当需求(requirements )改变，重构(refactoring)类层次结构变得非常困难，以至于变成了一个WTF三明治，其中包含了过时的(outdated )需求痕迹(traces)。不要创建类层次结构，而应该考虑创建多个工厂函数。他们可能互相调用对方，调整对方行为。你可以教“基础(base)”工厂函数去接受调整行为的“战略(strategy)”对象，并让其他工厂函数提供它。无论(Regardless)你选择什么方法，重要的部分都是在每一步保持清晰的输入和输出。“你需要重写这个方法”不是一个明确的API，很难成为好的设计。但“你需要提供一个方法作为参数”是明确的，可以帮助你思考

- 不要使用方法进行超级调用;

如果你重写派生(derived )类中的方法，就完全重写它。追踪一个序列 super 调用就像是 追寻了一系列电影恶棍(villain)隐藏在世界各地的笔记,它只有在你看到别人也这么做的时候才有意思。如果你真的需要改变超级方法的结果，或者在做其他事情之前或之后执行(perform)它，那该怎么办？看前面的观点：将你的类变成工厂函数，并清晰地保持他们之间的关系。当你唯一(only)工具是参数和返回值时，更容易发现正确的责任平衡. 中间(intermediate)函数可以具有与“顶级”和“底级”函数不同的（并且更精细的(fine-grained:细粒度)）接口(interfaces)。类不容易提供这样的机制，因为你不能指定“仅用于基类”或“仅用户派生类”的方法，但功能组合使其变得更自然。

- 不要指望别人会用你的类；

即使你选择提供你的类作为公共API, 在接受输入时也更愿意用duck typing([鸭子类型](https://www.cnblogs.com/zzyzz/p/7723272.html)：多态的一种形式)，取代instanceof检查，断言你计划使用的方法的存在，并相信用户做正确的事情。这会使用户区(userland) 扩展和后来的调整更容易，并且消除iframe和不同javascript执行(execution)上下文。


- 学习函数式编程；

它会让你不需要考虑类，所以即使你知道它们的陷阱(pitfalls)，你也不会被迫(compelled)使用它们.

那么React组件如何？
我希望上面的规则 说明了为什么这是一个可怕的想法：
```
import { Component } from 'react';
class BaseButton extends Component {
  componentDidMount() {
    console.log('hey');
  }
  render() {
    return <button>{this.getContent()}</button>;
  }
}
class RedButton extends BaseButton {
  componentDidMount() {
    super.componentDidMount();
    console.log('ho');
  }
  getContent() {
    return 'I am red';
  }
  render() {
    return (
      <div className='red'>
        {super.render()}
      </div>
    );
  }
}
```

然而，下面的代码才是好的。
```
import { Component } from 'react';
class Button extends Component {
  componentDidMount() {
    console.log('hey');
  }
  render() {
    return (
      <div className={this.props.color}>
        <button>{this.props.children}</button>
      </div>
    );
  }
}
class RedButton extends Component {
  componentDidMount() {
    console.log('ho');
  }
  render() {
    return (
      <Button color='red'>
        I am red
      </Button>
    );
  }
}
```

是的，我们使用了可怕的class关键字，但是我们没有创建层级结构，因为我们是在扩展组件。如果你愿意的话，你可以写lint rules。没有必要为了避免在上面的代码中使用class关键字而跳过这些循环。这不是一个真正的问题。

当你想以通用(generic)的方式为组件提供(supply)一些附加(additional)功能时，高阶组件几乎覆盖了我迄今为止遇到的(encountered)所有场景。从技术上讲(Technically)，他们只是更高阶的功能。

在发现高阶组件之后，我还没有感到需要createClass() 风格的mixin，ES7 mixin，[“stamp composition”](https://github.com/stampit-org/react-stampit/blob/master/docs/composition.md)或其他任何组合解决方案。这是另一个赞成使用class的理由。因为你实际上(realistically)不需要任何“更强大”的东西

从React 0.14你可以用纯函数写组件。这完全值得你这么做。任何时候你可以写一个函数而不是一个类就行。

然而，当你需要生命周期钩子或状态时，直到react完成一些纯粹的功能解决方案后，当你没有违反上述规则时，我认为在使用类时没有什么坏处。事实上，从ES6类迁移到纯函数方法比从其他任何方面都更容易。

因此我担心使用 不直接利用React组合模式的“组合解决方案(compositional solutions)”，因为我认为这是在退步的，在概念上更接近于功能模式中没有意义的( don’t make sense)混合模式。



那么，我对React组件的建议是什么？
- 如果你不继承多次 且 不用超级调用(super)，那你可以在你的JS中使用类
- 尽可能将React组件编写为纯函数
- 如果你需要状态或生命周期钩子，请将ES6类用于组件
- 在这种情况下，你只能直接(directly)扩展 React.Component
- 将你的反馈意见发送给React 团队，了解功能状态提案(proposals)

睡得安稳吧~



---

网友 Daniel Lo Nigro 回复：

they might read your private state(他们可能会读你的私有状态)

对于这个问题，在Facebook上，我们将私有成员的名字(定义为“任何带有下划线前缀的成员”)，所以不可能从类以外的地方访问它们，即使是后代也不能碰私有成员。

---
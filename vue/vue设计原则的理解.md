### 谈一下对vue设计原则的理解
> vue官网介绍，核心理念是：JavaScript渐进式框架，易用 灵活 高效

- JavaScript渐进式框架
    vue只提供一个核心库，熟悉vue语法以及模板框架之后就可以迅速开发，在项目过程中需要插件，如router和vuex等库，可以在引入进来，从开发到学习都是一个渐进式过程
- 易用
    vue提供了数据响应式，声明式模版语法和基于配置的组件等功能，可以让开发者在开发过程中者更多的关心核心业务，只需要会写html，css，js就可以编写vue应用
- 灵活
    这点在JavaScript渐进式框架中有提及过，因为vue只提供了核心库，所有的插件以及公共库的应用可以根据我们的需求来，比如element-ui，我们如果只需要其中某些ui组件可以单独拿出来引入
- 高效
    vue采用了虚拟dom和diff算法大幅提高我们项目的性能，并且在vue3中的proxy对响应式数据做出的改进还会进一步提升我们的项目性能
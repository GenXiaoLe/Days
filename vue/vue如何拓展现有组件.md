### vue如何扩展现有组件
1. 使用Vue.mixin进行全局混入
2. 提取组件中的公共方法使用mixin在组件中进行混入
3. 使用slot扩展
    匿名插槽和具名插槽

ps：在混入中，全局混入方法会先执行，然后是组件中的混入，最后是组件中原生的方法
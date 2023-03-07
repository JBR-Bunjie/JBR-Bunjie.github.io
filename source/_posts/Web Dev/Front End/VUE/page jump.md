## this.$router.push

## this.$router.go

```js
// 字符串
this.$router.push('/home/first')
// 对象
this.$router.push({ path: '/home/first' })
// 命名的路由
this.$router.push({ name: 'home', params: { myid: id }})
```



```vue
<!---->
<button @click="goback">返回上页</button>

methods:{
  goback(){
    this.$router.go(-1)
  }
}
```



```vue
<!-- jump to certain page -->
<button @click="goToLink">返回page1</button>

methods:{
  goToLink(){
    this.$router.push('/page1')
}
<!-- == -->
this.$router.push({name:'page1'})
```



1、作用：this.$router.push() 可以通过修改url实现路由跳转。

2、push 后面可以是对象，也可以是字符串：



.Params

由于动态路由也是传递params的，所以在 this.$router.push() 方法中path不能和params一起使用，否则params将无效。需要用name来指定页面。

及通过路由配置的name属性访问

在路由配置文件中定义参数：

通过name获取页面，传递params：

在目标页面通过this.$route.params获取参数：
{
path: ‘/detail/:myid’, // 动态路由
name: ‘home’, // 命名路由
component: Detail
},

## Vue 动态路由 与 命名路由


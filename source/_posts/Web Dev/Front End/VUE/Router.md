# vue-router：404页面路由配置

```js
{
    name:'404',
    path:'/404.html',
    component: resolve => require(['../page/NotFound/view.vue'], resolve),
},
{
     path:'*',
     redirect:{
         name:"404"
     }
}
```


## Problems in Vue projects you may meet



### 1. your Vue is blank

```vue
<template>

<!-- notice! you should know that the router view is the most important label in the parent page, if it doesn't exist, you can only see the blank page! --->
	<router-view/>
	<!-- notice! you should know that the router view is the most important label in the parent page, if it doesn't exist, you can only see the blank page! --->

</template>

<script>
export default {
  name: "defaultPage"
}
</script>

<style scoped>

</style>
```

不管你创建的是第几级路由，总要有一个该级别的路由总页面，并且在这个总页面上要存在 \<router-view\> 标签来指明展示子级页面


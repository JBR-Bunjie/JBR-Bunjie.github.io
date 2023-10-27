---
title: RESTful规范 - 如何写API
date: 2021-7-23 12:23:23
tags:
  - Theroy
categories:
  - Theroy
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# RESTful规范 - 如何写API

1. 如果是对同一个表进行数据操作，应该使用同一条 API，然后根据 method 的不同，进行不同的操作

   ```markdown
   GET/POST/PUT/DELETE/PATCH
   ```

2. 面向资源编程，通过 API 提交的参数最好是名词如 name，尽量少用动词

   ```markdown
   http://www.abc.com/name
   ```

3. 体现版本，在 API 中加入像 v1, v2 这样的版本代号

   ```markdown
   http://www.abc.com/v1/name
   http://www.abc.com/v2/name
   ```

4. 体现 API，让使用者一眼能看出这是 API 而不是 URL，应该在 API 中加入提示：

   ```markdown
   http://www.abc.com/api/v1/name
   http://www.abc.com/api/v2/name
   ```

5. 使用 HTTPS

   ```markdown
   https://www.abc.com/api/v1/name
   https://www.abc.com/api/v2/name
   ```

6. 响应式设置状态码

   ```python
   return HttpResponse('abcde', status=200)
   ```

7. API 的参数里加入筛选条件参数，也可以理解为获取资源优先选择 GET 的方式

   ```markdown
   https://www.abc.com/api/v2/name?page=1&size=10
   ```

8. 返回值的规范，对不同的请求 method，做出相应的响应值如：

   ```markdown
   -> https://www.abc.com/api/v1/name

   - GET: 所有列表
   - POST: 新增的数据

   -> https://www.abc.com/api/v1/name/1

   - GET: 单条数据
   - PUT: 更新，返回更新的数据
   - PATCH: 局部更新，返回更新的数据
   - DELETE: 删除，返回空文档
   ```

9. 返回错误信息，包括错误代号等，让用户直接看出是哪种类型错误

   ```json
   ret {
       code: 1000,
       data: {
           1: {'id': 1, 'title': 'lala'}
       }
   }
   ```

10. 返回的详细信息，应该以字典的形式存放在 data 中
    ```json
    ret {
        code: 1000,
        data: {
            1: {'id': 1, 'title': 'lala', 'detail': 'lalalalala'}
        }
    }
    ```

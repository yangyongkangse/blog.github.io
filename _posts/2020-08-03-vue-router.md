---
layout: post
title: "Vue路由参数的传递和获取"
subtitle: ""
date: 2020-08-03 17:07:22 +0800
author: "YangYongKang"
header-style: text
catalog: true
archive:
    - 前端
    - Vue
    - Router
tags:
    - 前端
    - Vue
    - Router
---


##### 路由传参方式

1. this.$router.push({path:'/about',query:{id:1}}); --类似于get请求,地址栏显示参数
2. this.$router.push({name:'about',query:{id:1}); --类似于get请求,地址栏显示参数
3. this.$router.push({name:'about',params:{id:1}}); --推荐使用,类似于post请求,地址栏不显示参数,没有参数大小限制

##### 查询方式
1. 使用path时,query参数值能正常获取,在地址栏展示参数,类似于get,params参数不能正常获取,值为空.
2. 使用name时,query参数值能正常获取,在地址栏展示参数,类似于get,params参数值能正常获取,地址栏不展示参数,类似于post.

##### 参数获取
1. this.$route.query
2. this.$route.params

##### params方式参数丢失问题解决方案
1. params方式进行路由跳转后进行页面刷新后参数会丢失,可在路由地址后面设置参数规则
如:
```vue
          {
            path: "roleManage/:id",
            name: "roleManage",
            meta: { title: "角色信息维护" },
            component: () => import("@/views/system/role/roleManage.vue")
          }
```
2. 通过localStorage进行处理

# 路由元信息meta

一个路由匹配到的所有路由记录会暴露为 **$route** 对象 (还有在导航守卫中的路由对象) 的 **`$route.matched`** 数组。遍历 **`$route.matched`** 来检查路由记录中的 **`meta`** 字段

```javascript
const router = new VueRouter({
	routes:[
		{
			path:'/foo',
			component:Foo,
			children:[
				path:'bar',
				component:Bar,
				//给未来的路由做权限控制
				meta: { 
					//证明用户访问该组件的时候需要登录
					requiresAuth:true
				}
			]
		}
	]
})

router.beforeEach((to,from,next)=>{
    // to.matched.some(record => record.meta.requiresAuth)
    if(to.meta.requiresAuth){
        //用户点击了博客链接，该用户需要登录
        if(localStorage.getItem('user')){//如果localStorage不为空，表示登录完成
            next();
        }else{//需要登录
            next({
                path:'/login'
            })
        }
    }else{//直接放行
        next();//如果不调用next()页面会卡住
    }
});
```




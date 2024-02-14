## `condition`属性

启动模式配置，仅开发期间生效，用于模拟直达页面的场景，如：进入详情页，直接配置`condition`属性，这样在开发时，页面直接显示特定的页面，就不需要手动去点击，提升开发效率。

```json
"condition": { //模式配置，仅开发期间生效
	"current": 0, //当前激活的模式（list 的索引项）
	"list": [{
			"name": "详情页", //模式名称
			"path": "pages/component/detail/detail", //启动页面，必选
			"query": "id=20" //启动参数，在页面的onLoad函数里面得到。
		},
	]
}
```


# AJAX

> 在不重载全部页面的情况下，实现了对部分网页的更新。AJAX 通过后台加载数据，并在网页上进行显示

## load()

> load() 方法从服务器加载数据，并把返回的数据放入被选元素中

`$(selector).load(URL,data,callback);`

- 必需的 *URL* 参数规定您希望加载的 URL。
- 可选的 *data* 参数规定与请求一同发送的查询字符串键/值对集合。
- 可选的 *callback* 参数是 load() 方法完成后所执行的函数名称。

```javascript
// 把文件 "demo_test.txt" 的内容加载到指定的 div 元素中
$(document).ready(function(){
	$("button").click(function(){
		$("#div1").load("/try/ajax/demo_test.txt");
	});
});
```

* **可以把 jQuery 选择器添加到 URL 参数**

  ```javascript
  // 把 "demo_test.txt" 文件中 id="p1" 的元素的内容，加载到指定的 div 元素中
  $(document).ready(function(){
    $("button").click(function(){
      $("#div1").load("/try/ajax/demo_test.txt #p1");
    });
  });
  ```

**可选的 callback 参数规定当 load() 方法完成后所要允许的回调函数。回调函数可以设置不同的参数：**

- *responseTxt* - 包含调用成功时的结果内容
- *statusTXT* - 包含调用的状态
- *xhr* - 包含 XMLHttpRequest 对象

```javascript
$(document).ready(function(){
    $("button").click(function(){
        $("#div1").load("/try/ajax/demo_test.txt",function(responseTxt,statusTxt,xhr){
            if(statusTxt=="success")
                alert("外部内容加载成功!");
            if(statusTxt=="error")
                alert("Error: "+xhr.status+": "+xhr.statusText);
        });
    });
```

**为了避免多页面情形下的代码重复，可以利用 load() 方法，将重复的部分（例如导航栏）放入单独的文件，使用下列方法进行导入：**

```javascript
//1.当前文件中要插入的地方使用此结构：
<div class="include" file="***.html"></div>

//2.***.html中放入内容，用html格式仅仅因为会有编辑器的书写辅助。。

//3.代码：
$(".include").each(function() {
    if (!!$(this).attr("file")) {
        var $includeObj = $(this);
        $(this).load($(this).attr("file"), function(html) {
            $includeObj.after(html).remove(); //加载的文件内容写入到当前标签后面并移除当前标签
        })
    }
});
// 或者在index文件里只写重复部分，剩下的一股脑放各自单独文件 load() 进来~
```

## get()

> 从指定的资源请求数据

`$.get(URL,callback);`

*callback* 参数是请求成功后所执行的函数名,第一个回调参数存有被请求页面的内容，第二个回调参数存有请求的状态

```javascript
$(document).ready(function(){
    $("button").click(function(){
        $.get("/try/ajax/demo_test.php",function(data,status){
            alert("数据: " + data + "\n状态: " + status);
        });
    });
});
```

## post()

> 向指定的资源提交要处理的数据

`$.post(URL,data,callback);`

- 必需的 *URL* 参数规定您希望请求的 URL。
- 可选的 *data* 参数规定连同请求发送的数据。
- 可选的 *callback* 参数是请求成功后所执行的函数名,第一个回调参数存有被请求的内容，而第二个参数存有请求的状态。

```javascript
$(document).ready(function(){
	$("button").click(function(){
		$.post("/try/ajax/demo_test_post.php",{
			name:"菜鸟教程",
			url:"http://www.runoob.com"
		},
		function(data,status){
			alert("数据: \n" + data + "\n状态: " + status);
		});
	});
});
```

## **GET 和 POST 方法的区别**：

**1、发送的数据数量**

在 GET 中，只能发送有限数量的数据，因为数据是在 URL 中发送的。

在 POST 中，可以发送大量的数据，因为数据是在正文主体中发送的。

**2、安全性**

GET 方法发送的数据不受保护，因为数据在 URL 栏中公开，这增加了漏洞和黑客攻击的风险。

POST 方法发送的数据是安全的，因为数据未在 URL 栏中公开，还可以在其中使用多种编码技术，这使其具有弹性。

**3、加入书签中**

GET 查询的结果可以加入书签中，因为它以 URL 的形式存在；而 POST 查询的结果无法加入书签中。

**4、编码**

在表单中使用 GET 方法时，数据类型中只接受 ASCII 字符。

在表单提交时，POST 方法不绑定表单数据类型，并允许二进制和 ASCII 字符。

**5、可变大小**

GET 方法中的可变大小约为 2000 个字符。

POST 方法最多允许 8 Mb 的可变大小。

**6、缓存**

GET 方法的数据是可缓存的，而 POST 方法的数据是无法缓存的。

**7、主要作用**

GET 方法主要用于获取信息。而 POST 方法主要用于更新数据。
# HTML

## 获得/设置内容 - text()、html() 以及 val()

三个简单实用的用于 DOM 操作的 jQuery 方法：

- text() - 设置或返回所选元素的文本内容
- html() - 设置或返回所选元素的内容（包括 HTML 标记）
- val() - 设置或返回表单字段的值

```javascript
// 获取
$(document).ready(function(){
  $("#btn1").click(function(){
    alert("Text: " + $("#test").text());
  });
  $("#btn2").click(function(){
    alert("HTML: " + $("#test").html());
  });
});
// 设置
$(document).ready(function(){
  $("#btn1").click(function(){
    $("#test1").text("Hello world!");
  });
  $("#btn2").click(function(){
    $("#test2").html("<b>Hello world!</b>");
  });
  $("#btn3").click(function(){
    $("#test3").val("RUNOOB");
  });
});
```

### text()、html() 以及 val() 的回调函数

> 回调函数有两个参数：被选元素列表中当前元素的下标，以及原始（旧的）值。然后以函数新值返回您希望使用的字符串。

```javascript
$(document).ready(function(){
    $("#btn1").click(function(){
        $("#test1").text(function(i,origText){
            return "旧文本: " + origText + " 新文本: Hello world! (index: " + i + ")"; 
        });
    });

    $("#btn2").click(function(){
        $("#test2").html(function(i,origText){
            return "旧 html: " + origText + " 新 html: Hello <b>world!</b> (index: " + i + ")"; 
        });
    });

});
```

## 获取/设置属性 - attr()、prop()

```javascript
// 获取
<a href="//www.runoob.com" id="runoob">菜鸟教程</a>
$("#runoob").attr("href")

// 设置
$(document).ready(function(){
  $("button").click(function(){
    $("#runoob").attr("href","http://www.runoob.com/jquery");
  });
});

// 同时设置多个属性
$(document).ready(function(){
  $("button").click(function(){
    $("#runoob").attr({
      "href" : "http://www.runoob.com/jquery",
      "title" : "jQuery 教程"
    });
	// 通过修改的 title 值来修改链接名称
	title =  $("#runoob").attr('title');
	$("#runoob").html(title);
  });
});
```

**prop()函数的结果:**

   1.如果有相应的属性，返回指定属性值。

   2.如果没有相应的属性，返回值是空字符串。

**attr()函数的结果:**

   1.如果有相应的属性，返回指定属性值。

   2.如果没有相应的属性，返回值是 `undefined`。

对于HTML元素本身就带有的固有属性，在处理时，使用**prop方法 **。

对于HTML元素我们自己自定义的DOM属性，在处理时，使用**attr 方法**。

具有 true 和 false 两个属性的属性，如 checked, selected 或者 disabled 使用prop()

```html
<a href="https://www.runoob.com" target="_self" class="btn">菜鸟教程</a>
```

这个例子里 `a` 元素的 DOM 属性有: **href、target** 和 **class**，这些属性就是 `a` 元素本身就带有的属性，也是 W3C 标准里就包含有这几个属性，或者说在 IDE 里能够智能提示出的属性，这些就叫做固有属性。处理这些属性时，建议使用 **prop** 方法。

```html
<a href="#" id="link1" action="delete" rel="nofollow">删除</a>
```

这个例子里`a` 元素的 DOM 属性有: **href、id** 和 **action**，很明显，前两个是固有属性，而后面一个 **action** 属性是我们自己自定义上去的，`a` 元素本身是没有这个属性的。这种就是自定义的 DOM 属性。处理这些属性时，建议使用 **attr** 方法。

### attr() 的回调函数

回调函数有两个参数：被选元素列表中当前元素的下标，以及原始（旧的）值。然后以函数新值返回您希望使用的字符串。

```javascript
$(document).ready(function(){
    $("button").click(function(){
        $("#runoob").attr("href", function(i, origValue){
            return origValue + "/jquery";
        });
    });
});
```

## 添加元素

​	append() 和 prepend() 、after() 和 before() 方法能够通过参数接收无限数量的新元素

- **append()** - 在被选元素的结尾插入内容（仍然在该元素的内部）

  ```javascript
  $(document).ready(function(){
    $("#btn1").click(function(){
      $("p").append(" <b>追加文本</b>。");
    });
  
    $("#btn2").click(function(){
      $("ol").append("<li>追加列表项</li>");
    });
  });
  
  function appendText(){
  	var txt1="<p>文本-1。</p>";              // 使用 HTML 标签创建文本
  	var txt2=$("<p></p>").text("文本-2。");  // 使用 jQuery 创建文本
  	var txt3=document.createElement("p");
  	txt3.innerHTML="文本-3。";               // 使用 DOM 创建文本 text with DOM
  	$("body").append(txt1,txt2,txt3);        // 追加新元素
  }
  ```

- **prepend() **- 在被选元素的开头插入内容（仍然在该元素的内部）

  ```javascript
  $(document).ready(function(){
  	$("#btn1").click(function(){
  		$("p").prepend("<b>在开头追加文本</b>。 ");
  	});
  	$("#btn2").click(function(){
  		$("ol").prepend("<li>在开头添加列表项</li>");
  	});
  });
  ```

- **after()** - 在被选元素之后插入内容（在元素外面追加）

- **before()** - 在被选元素之前插入内容（在元素外面追加）

  ```javascript
  $(document).ready(function(){
      $("#btn1").click(function(){
          $("img").before("<b>之前</b>");
      });
  
      $("#btn2").click(function(){
          $("img").after("<i>之后</i>");
      });
  });
  
  function afterText(){
  	var txt1="<b>I </b>";                    // 使用 HTML 创建元素
  	var txt2=$("<i></i>").text("love ");     // 使用 jQuery 创建元素
  	var txt3=document.createElement("big");  // 使用 DOM 创建元素
  	txt3.innerHTML="jQuery!";
  	$("img").after(txt1,txt2,txt3);          // 在图片后添加文本
      $("img").after([txt1,txt2,txt3]); // 或者是个数组
  }
  ```

## 删除元素

- **remove() ** - 删除被选元素（及其子元素）

  ```javascript
  $(document).ready(function(){
    $("button").click(function(){
      $("#div1").remove();
    });
  });
  ```

  **过滤被删除的元素:**

  该方法可接受一个参数，允许您对被删元素进行过滤

  ```javascript
  $(document).ready(function(){
    $("button").click(function(){
        // 删除 class="italic" 的所有 <p> 元素
      $("p").remove(".italic");
    });
  });
  ```

  在使用 remove() 的过滤器删除时，不能删除带有过滤器的子元素（过滤器中条件只能作用于同级，不能作用于子元素）。**$(selector)** 语法的返回结果是一个元素的列表，即：将 **$("#div1")** 看作一个列表，**remove()** 中的筛选条件实际上是对这个列表中的元素进行筛选删除，而不会去删除这个列表中不存在的元素（子元素不在这个列表中）:

  ```javascript
  <div id="div1" style="height:100px;width:300px;border:1px solid black;background-color:yellow;">
  	这是 div 中的一些文本。
  	<p class="part">这是在 div 中的一个段落。</p>
  	<p>这是在 div 中的另外一个段落。</p>
  </div>
  <br>
  
  $(document).ready(function(){
  	$("button").click(function(){
  		$("#div1").remove(".part"); // 点击时没有任何反应
  	});
  });
  ```

- **empty() ** - 从被选元素中删除子元素

  ```javascript
  $(document).ready(function(){
    $("button").click(function(){
      $("#div1").empty();
    });
  });
  ```

## 获取并设置 CSS 类

```javascript
addClass('c1 c2 ...' | function(i, c)) -- 添加一个或多个类。

removerClass('c1 c2 ...' | function(i, c)) -- 删除一个或多个类。

toggleClass('c1 c2 ...' | function(i, c), switch?) -- 切换一个或多个类的添加和删除。
```

第一个参数表示要添加或删除的类，既可以用类列表，也可以用函数返回值指定（i 是选择器选中的所有元素中当前对象的索引值，c 是当前对象的类名）。

**switch**: 布尔值，true 表示只添加，false 表示只删除。

- addClass() - 向被选元素添加一个或多个类

  ```javascript
  // 添加类时，也可以选取多个元素
  $(document).ready(function(){
      $("button").click(function(){
          $("h1,h2,p").addClass("blue");
          $("div").addClass("important");
      });
  });
  
  // 在 addClass() 方法中规定多个类
  $(document).ready(function(){
      $("button").click(function(){
          $("body div:first").addClass("important blue");
      });
  });
  ```

- removeClass() - 从被选元素删除一个或多个类

  ```javascript
  // 在不同的元素中删除指定的 class 属性
  $(document).ready(function(){
      $("button").click(function(){
          $("h1,h2,p").removeClass("blue");
      });
  });
  ```

- toggleClass() - 对被选元素进行添加/删除类的切换操作

  ```javascript
  // 对被选元素进行添加/删除类的切换操作
  $(document).ready(function(){
      $("button").click(function(){
          $("h1,h2,p").toggleClass("blue");
      });
  });
  ```

- css() - 设置或返回样式属性

  > css() 方法设置或返回被选元素的一个或多个样式属性

  * **返回指定的 CSS 属性的值**

    css("*propertyname*");

    ```javascript
    $("p").css("background-color")
    
    // 获取指定 p 的背景颜色
    // 但要注意 :nth-child() 选择器默认以 body 作为父标签
    <body>
    <h2>这是一个标题</h2>
    <p style="background-color:#ff0000">这是一个段落。</p>
    <p style="background-color:#00ff00">这是一个段落。</p>
    <p style="background-color:#0000ff">这是一个段落。</p>
    <button>返回第一个 p 元素的 background-color </button>
    </body>
    $("button").click(function(){
        alert("p1背景颜色 = " + $("p:nth-child(2)").css("background-color"));
        alert("p2背景颜色 = " + $("p:nth-child(3)").css("background-color"));
        alert("p3背景颜色 = " + $("p:nth-child(4)").css("background-color"));
    });
    
    // 使用children()方法来获取指定的p
    <div class="getColor">
      <p class="a">first</p>
      <p class="b">second</p>
      <p class="c">third</p>
    </div>
    $(function(){
        $(".color").click(function(){
            alert("b的背景颜色为：" + $(".getColor").children(".b").css("background-color"));
        }); 
    });
    
    // 使用eq()方法获取指定的 p
    $("p").eq(N).css('background-color') // N 是索引号，从 0 开始
    $(function() {
        $("button").click(function() {
           for(var i = 0; i < $("p").length; i++)
            {
                alert($("p").eq(i).css("background-color"));
            }
      });
    });
    ```

  * **设置指定的 CSS 属性**

    `css("propertyname","value");`

    ```javascript
    $("p").css("background-color","yellow");
    ```

  * **设置多个 CSS 属性**

    `css({"propertyname":"value","propertyname":"value",...});`

    ```javascript
    $(document).ready(function(){
        $("button").click(function(){
            $("p").css({"background-color":"yellow","font-size":"200%"});
        });
    });
    ```

## 元素和浏览器窗口的尺寸

![jQuery Dimensions](https://www.runoob.com/images/img_jquerydim.gif)

  - width() ： 设置或返回元素的宽度（不包括内边距、边框或外边距）

  - height() ：设置或返回元素的高度（不包括内边距、边框或外边距）

    ```javascript
    $("button").click(function(){
      var txt="";
      txt+="div 的宽度是: " + $("#div1").width() + "</br>";
      txt+="div 的高度是: " + $("#div1").height();
      $("#div1").html(txt);
    });
    ```

  - innerWidth() : 返回元素的宽度（包括内边距）

  - innerHeight() : 返回元素的高度（包括内边距）

    ```javascript
    $("button").click(function(){
      var txt="";
      txt+="div 宽度，包含内边距: " + $("#div1").innerWidth() + "</br>";
        txt+="div 高度，包含内边距: " + $("#div1").innerHeight();
      $("#div1").html(txt);
    });
    ```

  - outerWidth() : 返回元素的宽度（包括内边距和边框）

  - outerHeight() : 返回元素的高度（包括内边距和边框）

    ```javascript
    $("button").click(function(){
      var txt="";
      txt+="div 宽度，包含内边距和边框: " + $("#div1").outerWidth() + "</br>";
      txt+="div 高度，包含内边距和边框: " + $("#div1").outerHeight();
      $("#div1").html(txt);
    });
    ```

    **注意:**设置了 box-sizing 后，width() 获取的是 css 设置的 width 减去 padding 和 border 的值

    ```css
    .test{width:100px;height:100px;padding:10px;border:10px;box-sizing:border-box;}
    ```

    -  width() 获取为: 60
    -  innerWidth() 获取值为: 80
    -  outWidth() 获取值为: 100

  - outerWidth(true) : 返回元素的宽度（包括内边距、边框和外边距）

  - outerHeight(true) : 返回元素的高度（包括内边距、边框和外边距）
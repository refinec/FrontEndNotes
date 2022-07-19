# css文字滚动

## 一、纯CSS方式

```html
<div class="column">
  <div class="sampleContainer for-css">
    <div class="u-scrollTitleContainer js-scrollContainer">
      <div class="u-scrollTitle">A friend is someone who gives you total freedom to be yourself. - Jim Morrison</div>
    </div>
  </div>
  <br>just CSS3
</div>
```

```css
.column {
  display: inline-block;
  width: auto;
  margin: 0 20px;
  text-align: center;
}

.sampleContainer {
  width: 180px;
  height: 180px;
  background-color: cadetblue;
}

.u-scrollTitleContainer {
  position: relative;
  width: 90%;
  height: 100%;
  text-align: center;
  overflow: hidden;
  margin: 0 auto;
  cursor: pointer;
}

.u-scrollTitle {
  display: inline-block;
  position: relative;
  left: 0;
  top: 45%;
  font-family: Helvetica;
  color: white;
  white-space: nowrap;
}

// 动画
.for-css .u-scrollTitleContainer:hover .u-scrollTitle {
  filter: blur(0);
  -webkit-filter: blur(0);
  -moz-filter: blur(0);
  -o-filter: blur(0);
  animation: marquee 12s linear infinite;
  -webkit-animation: marquee 12s linear infinite;
  -moz-animation: marquee 12s linear infinite;
  -o-animation: marquee 12s linear infinite;
}

@keyframes marquee {
  0% {
    left: 100%;
  }
  100% {
    transform: translateX(-100%);
    -webkit-transform: translateX(-100%);
    -moz-transform: translateX(-100%);
    -o-transform: translateX(-100%);
  }
}
@-webkit-keyframes marquee {
  0% {
    left: 100%;
  }
  100% {
    transform: translateX(-100%);
    -webkit-transform: translateX(-100%);
    -moz-transform: translateX(-100%);
    -o-transform: translateX(-100%);
  }
}
```

## 二、 Jquery方式

```html
<div class="column">
  <div class="sampleContainer for-jquery">
    <div class="u-scrollTitleContainer js-scrollContainer">
      <div class="u-scrollTitle">A friend is someone who gives you total freedom to be yourself. - Jim Morrison</div>
    </div>
  </div>
  <br>just jQuery
</div>
```

```css
.column {
  display: inline-block;
  width: auto;
  margin: 0 20px;
  text-align: center;
}

.sampleContainer {
  width: 180px;
  height: 180px;
  background-color: cadetblue;
}

.u-scrollTitleContainer {
  position: relative;
  width: 90%;
  height: 100%;
  text-align: center;
  overflow: hidden;
  margin: 0 auto;
  cursor: pointer;
}

.u-scrollTitle {
  display: inline-block;
  position: relative;
  left: 0;
  top: 45%;
  font-family: Helvetica;
  color: white;
  white-space: nowrap;
}
```

```js
$(document).ready(function () {
  /* Just jQuery */
  var textInterval;
  $('.for-jquery .js-scrollContainer').hover(function () {
    var el = $(this);
    textInterval = setInterval(function () {
      scrollText(el);
    }, 20);
  }, function () {
    clearInterval(textInterval);
    $('.u-scrollTitle', this).css({
      left: 0
    });
  });
  var scrollText = function (elem) {
    var el = elem.find('.u-scrollTitle'),
        left = el.position().left - 1;
    left = -left > el.width() ? el.width() : left;
    el.css({
      left: left
    });
  };
});
```


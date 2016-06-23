# codeNote

1）事件兼容模型

```javascript
/**
 * Created by PY on 2016/2/24 0024.
 */
    /*主要分为DOM2级非IE模式，DOM0级，IE模式*/
EventUtil = {
    /*添加事件*/
    addHandler: function(element, type, handler) {
        if(element.addEventListener) {
            //非IE DOM2
            element.addEventListener(type, handler, false);
        } else if(element.attachEvent) {
            //IE
            element.attachEvent("on" + type, handler);
        } else {
            //DOM 0级 原型为element.onclick = function(){}; event对象为window.event
            element["on" + type] = handler;
        }
    },

    /*删除事件*/
    removeHandler: function(element, type, handler) {
        if (element.removeEventListener) {
            element.removeEventListener();
        } else if (element.detachEvent) {
            element.detachEvent();
        } else {
            element["on" + type] = null;
        }
    },

    /*获取事件对象*/
    getEvent: function(event) {
        return event ? event : window.event;
    },

    /*获取事件目标*/
    getTarget: function(event) {
          return event.target || event.srcElement;
    },

    /*取消事件默认行为*/
    preventDefault: function(event) {
        if(event.preventDefault) {
            event.preventDefault();
        }  else {
            event.returnValue = false;
        }
    },

    //阻止事件流
    stopPropagation: function(event) {
        if(event.stopPropagation) {
            event.stopPropagation();
        } else {
            //IE不支持事件捕获，因此只能阻止事件冒泡
            event.cancelBubble = true;
        }
    },

    //取得相关元素
    getRelatedTarget: function(event) {
        if(event.relatedTarget) {
            return event.relatedTarget;
        } else if(event.toElement) {
            return event.toElement;
        } else if(event.fromElement) {
            return event.fromElement;
        } else {
            return null;
        }
    },

    //获取鼠标被点击的是哪个键，主键（左键）：0，滚轮按钮：1，鼠标次键（右键）：2
    //除IE8及其更早版本之外的其他浏览器都原生支持DOM模型
    //单独使用能力检测无法确定差异（两种模型有同名的button属性），由于支持DOM版鼠标事件的浏览器可
    //以通过hasFeature()方法来检测，则用hasFeature()进行检测
    getButton: function(event) {
        if(document.implementation.hasFeature("MouseEvent", "2.0")) {
            return event.button;
        } else {
            switch(event.button) {
                case 0:
                case 1:
                case 3:
                case 5:
                case 7:
                    return 0;
                case 2:
                case 6:
                    return 2;
                case 4:
                    return 1;
            }
        }
    },

    //鼠标滚轮事件
    getWheelDelta: function(event) {
        //非FrieFox
        if(event.wheelDelta) {
            var client = function() {
                //呈现引擎
                var engine = {
                  opera: 0,
                    //完整的版本号
                    ver: null
                };
                if(window.opera) {
                    engine.opera = parseFloat(window.opera.version());
                }
            };
            return (client.engine.opera && client.engine.opera < 9.5 ? -event.wheelDelta : event.wheelDelta);
        } else {
            return -event.detail * 40;
        }
    },

    //获取键盘字符编码
    //IE8及之前版本和Opera则是在keyCode中保存字符的ASCII编码
    getCharCode: function(event) {
        if(typeof event.charCode == "number") {
            return event.charCode;
        } else {
            return evetn.keyCode;
        }
    }
};

```

2）测试事件冒泡与捕获
```javascript

/*测试冒泡与捕获*/
    var $input = document.getElementsByTagName("input")[0];
    var $div = document.getElementsByTagName("div")[0];
    var $body = document.getElementsByTagName("body")[0];

    $input.addEventListener('click', function(e){
        this.style.border = "5px solid red";
        console.log("red");
       /* e.stopPropagation();*/
    },false);
    $div.addEventListener('click', function(e){
        this.style.border = "5px solid green";
        console.log("green");
    }, false);
    $body.addEventListener('click', function(e){
        this.style.border = "5px solid yellow";
        console.log("yellow");
    }, false);
    <div>
    <input type="button" value="测试事件冒泡" />
        </div>
```

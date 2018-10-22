## 瞎逼逼

做个原生js插件装逼一下

### 再逼逼

> 部分经常重复的代码抽象出来，写到一个单独的文件中为以后再次使用。再看一下我们的业务逻辑是否可以为团队服务。

插件不是随手就写成的，而是根据自己业务逻辑进行抽象。没有放之四海而皆准的插件，只有对插件，之所以叫做插件，那么就是开箱即用，或者我们只要添加一些配置参数就可以达到我们需要的结果。如果都符合了这些情况，我们才去考虑做一个插件。

### 插件封装的条件
1. 插件自身的作用域与用户当前的作用域相互独立，也就是插件内部的私有变量不能影响使用者的环境变量；
2. 插件需具备默认设置参数；
3. 件除了具备已实现的基本功能外，需提供部分API，使用者可以通过该API修改插件功能的默认参数，从而实现用户自定义插件效果；
4. 插件支持链式调用；
5. 插件需提供监听入口，及针对指定元素进行监听，使得该元素与插件响应达到插件效果。

### 插件的外包装
#### 用函数包装
> 插件就是封装在一个闭包中的函数集
``` javascript
  function add(n1, n2){
     return n1 + n2;
  }
```
* 团队开发中可能存在的问题
> 命名冲突,全局污染


#### 用全局对象包装
* 为了解决这种全局变量污染的问题。这时我们定义一个js对象来接收我们这些工具函数
```javascript
   let plugin = {
      add: function(n1, n2) {},   //加
      sub: function(n1, n2) {},   //减
      mul: function(n1, n2) {}    //乘
   }
```
* 调用：  plugin.add(1, 2)

> 上面的方式，在一定程度上已经解决了全部污染的问题。在团队协作中只要约定好命名规则了，告知其它同学即可。当然不排除别人重新定义并赋值这个值，这时的解决方案
``` javascript
   if(!plugin){  //这里的if条件可以用： (typeof plugin == 'undefined')
      var plugin = {
        // 逻辑
      }
   }

   // 也可以这样写
   var plugin;
   if(!plugin){
      plugin = {
        // ...
      }
   }
```
* 备注：为什么可以在此声明plugin变量？实际上js的解释执行，会把所有声明都提前。如果一个变量已经声明过，后面如果不是在函数内声明的，则没有影响的。所以，就算在别的地方声明 var plugin，也可以在这里再次声明一次
[如何判断Javascript对象是否存在](http://www.ruanyifeng.com/blog/2011/05/how_to_judge_the_existence_of_a_global_object_in_javascript.html)


#### 利用闭包包装
> 上面基本实现了插件的基本功能，不过我们的插件，是定义在全局域里面的。js变量的调用，从全局作用域上查找的速度比私有作用域里面慢的多。所以，我们最好将插件逻辑写在一个私有作用域里面。实现私有作用域，最好的办法就是使用闭包。可以把插件当做一个函数，插件内部的变量及函数的私有变量，为了在调用插件后依旧能使用其功能，闭包的作用就是延长函数(插件)内部变量的生命周期，使得函数可以重复调用，而不影响用户自身作用域。


``` javascript
   ;(function(global, undefined) {
      var plugin = {
        add: function(n1, n2) {... }
        ...
      }
      //最后将插件对象暴露给全局对象
      'plugin' in global && global.plugin = plugin;
   })(window);
```

>1. 在定义插件之前添加一个分号，可以解决js合并时可能会产生的错误问题；
> 2. undefined在老一辈的浏览器是不被支持的，直接使用会报错，js框架要考虑到兼容性，因此增加一个形参undefined，就算有人把外面的undefined定义看，里面的undefined依然不受影响；
> 3. 把window对象作为参数传入，是避免了函数执行的时候到外部去查找
* 直接传window对象进去，还是不妥当，插件不一定用于浏览器上，所以我们不传参数，直接取当前的全局this对象作为顶级对象用。
``` javascript
  ;(function(global, undefined) {
    "use strict"  //使用严格模式检查，使语法更规范
    var _global;
    var plugin = {
      add: function(n1, n2) { ... }
      ...
    }
    //最后将插件对象暴露给全局对象
    _global = (function() {return this || (0, eval)('this');}());
    !('plugin' in _global) && (_global.plugin = plugin);
  }());
```
* 如此，我们不需要传入任何参数，并且解决了插件对环境的依事性。如此我们的插件可以在任何宿主环境上运行了。

> 上面的代码段中有段奇怪的表达式：(0, eval)('this')，实际上(0,eval)是一个表达式，这个表达式执行之后的结果就是eval这一句相当于执行eval('this')的意思，详细解释看此篇：[(0,eval)('this')释义](https://www.jianshu.com/p/205a4033010a)或者看一下这篇[(0,eval)('this')](http://www.cnblogs.com/qianlegeqian/p/3950044.html)

立即自执行函数，有两种写法：

```javascript
   // 写法一
   (function() {})
```

```javascript
  //写法二
  （function(){} ()）
```
* 两种写法没区别，都是正确的写法，第二张更像一个整体

> **知识点**
> js里面()括号就是将代码结构变成表达式，被包在()里面的变成表达式之后，则就会立即执行，js中的代码表达式有很多种

``` javascript

    void function(){...}();

    // 或者
    !function foo(){...}();

    //或者
    +function foot(){...}();
````
当然，我们不推荐你这么用，而且乱用可能会产生一些歧义。


#### 使用模块化的规范包装
大型开发，多人开发，会产生多个文件，每个人负责一个小功能，那么如何才能将所有人开发的代码集合起来呢？要实现协同开发插件，需具备如下条件
* 每个功能互相之间的依赖必须明确，则必须严格按照依赖的顺序进行合并或者加载
* 每个子功能分别都要是一个闭包，并且将公共的接口暴露到共享域也及即使一个被主函数暴露的公共对象

关键如何实现，有多种办法，最笨的办法就是按顺序加载js
``` javascript
    <script type="text/javascript" src="part1.js"></script>
    <script type="text/javascript" src="part2.js"></script>
    <script type="text/javascript" src="part3.js"></script>
    ...
    <script type="text/javascript" src="main.js"></script>
```
但是不推荐这么做，这样做与我们所追求的插件的封装性相背。
加载器，比如[require](https://github.com/requirejs/requirejs)、[seajs](https://github.com/seajs/seajs)，或者也可以类似Node的方式进行加载，不过在浏览器端，我们还得利用打包器实现模块加载，比如[browserify](https://github.com/browserify/browserify)。

为了实现插件的模块化并且让我们的插件也是一个模块，我们就得让我们的插件也实现模块化的机制
实际上，只要判断是否存在加载器，我们就是使用加载器，如果不存在加载器。我们就使用顶级域对象

``` javascript
    if(typeof module !== "undefined" && module.exports) {
      module.exports = plugin;
    } else if (type define === "function" && define.amd) {
      define(function(){return plugin;});
    } else {
      _globals.plugin = plugin;
    }
```

这样我们的完整插件的样子应该是这样的：

``` javascript
    // plugin.js
    ;(function(undefined) {
        "use strict"
        var _global;
        var plugin = {
          add: function(n1, n2) { return n1 + n2; }, //加
          sub: function(n1, n2) { return n1 - n2; }, //减
          mul: function(n1, n2) { return n1 * n2; }, //乘
          div: function(n1, n2) { return n1 / n2; }, //除
          sur: function(n1, n2) { return n1 % n2; } //余
        }
        //最后将插件对象暴露给全局对象
        _global = (function(){return this || (0, eval)('this');}());
        if(typeof module !=="undefined" && module.exports) {
            module.exports = plugin;
        } else if(typeof define === "function" && define.amd) {
            define(function(){return plugin;});
        } else {
            !('plugin' in _global) && (_global.plugin = plugin);
        }
    }())

    // 引入插件之后，则可以直接使用plugin对象
    with(plugin) {
      console.log(add(2, 1)) //3
      console.log(sub(2, 1)) //1
      console.log(mul(2,1)) // 2
      console.log(div(2,1)) // 2
      console.log(sur(2,1)) // 0
    }

````

### 插件的API
#### 插件的默认参数

我们知道，函数是可以设置默认参数，而不管我们是否传有参数，我们都应该返回一个值以告诉用户我做了怎样的处理

``` javascript
    function add(param) {
      var args = !!param ?  Array.prototype.slice.call(arguments) : [];
      return args.reduce(function(pre, cur){
        return pre + cur;
      }, 0);
    }

    console.log(add()) //不传参，结果输出0，则这里已经设置了默认了参数的空数组
    console.log(add(1, 2, 3, 4, 5)) //传参，结果输出15
```

则作为一个健壮的js插件，我们应该吧一些基本的状态参数添加到我们需要的插件上去。
假设还是上面的加减乘除余的需求，我们如何实现插件的默认参数呢？ 道理是一样的

```javascript
    //plugin.js
    ;(function(undefined) {
      "use strict"
      var _global;

      function result(args, fn) {
        var argsArr = Array.prototype.slice.call(args);
        if(argsArr.length > 0) {
          return argsArr.reduce(fn);
        } else {
          return 0;
        }
      }

      var plugin = {
          add: function() {
              return result(arguments, function(pre, cur) {
                  return pre + cur;
              });
          },
          sub: function() {
              return result(arguments, function(pre, cur) {
                  return pre - cur;
              });
          },
          mul: function() {
              return result(arguments, function(pre, cur) {
                  return pre * cur;
              });
          },
          div: function() {
              return result(arguments, function(pre, cur) {
                  return pre / cur;
              });
          },
          sur: function() {
              return result(arguments, function(pre, cur) {
                  return pre % cur;
              });
          }
      }

      // 将插件对象暴露给全局对象
      _global = (function(){ return this || (0, eval)('this'); }());
      if(typeof module !=="undefined" && module.exports) {
          module.exports = plugin;
      } else if (typeof define === "function" && define.amd) {
          define(function() {return plugin;});
      } else {
          !('plugin' in _global) && (_global.plugin = plugin);
      }

    }());

    //输出结果
    with(plugin){
        console.log(add());
        console.log(add(2, 1)); //3
    }
```

* 参考 [with 用法](https://www.cnblogs.com/benchan2015/p/5057314.html)

实际上，插件都有自己的默认参数，就以我们最为常见的表单验证插件为例 [validate.js](https://github.com/rickharrison/validate.js/blob/master/validate.js)


#### 插件的钩子
设计一下插件，参数或者其逻辑肯定不是写死的，我们得像函数一样，得让用户提供自己的参数去实现用户的需求。则我们的插件需要提供一个修改默认参数的入口。[API]

> 我们常将容易被修改和变动的方法或属性统称为**钩子(Hook)**,方法则直接叫**钩子函数**，
实际上，我们知道插件可以像一条绳子上挂东西，也可以拿掉挂的东西，那么一个插件，实际上就是个形象上的**链**。不过我们上面的所有钩子都是挂在对象上的，用于实现链并不是很理想。

#### 插件的链式调用（利用当前对象）
插件并非都是链式调用的，有时候，我们只是用钩子来实现一个计算并返回结果，取得运算结果就可以了。但是有些时候，我们用钩子并不需要其返回结果。我们只利用其实现我们的业务逻辑，为了代码简洁与方便，我们常常将插件的调用按链式的方式进行调用。
最常见的jquery的链式调用如下：
``` javascript
  $(<id>).show().css('color','red').width(100).height(100)...
```
将链式调用运用到我们的插件中，将业务结构改为：
``` javascript
    var plugin = {
        add: function(n1, n2) { return this; },
        sub: function(n1, n2) { return this; },
        ...    
    }
```
我们只要将插件的当前对象this直接返回，则在下一个方法中，同样可以引用插件对象plugin的其它钩子方法，然后调用得时候就可以使用链式了
```javascript
    plugin.add().sub().mul().div()  
```
显然这样做并没有什么意义。我们这里的每一个钩子函数都只是用来计算并且获取返回值而已。而链式调用本身的意义是用来处理业务逻辑的。

#### 插件的链式调用（利用原型链）
Js中，万物皆对象，所有对象都是继承自原型。JS在创建对象（不论是普通对象还是函数对象）的时候，都有一个叫__proto__的内容属性，用于指向创建它的函数对象的原型对象prototype。

更改插件将plugin写成一个构造函数，我们将插件名换为Calculate避免因为Plugin大写的时候与window对象中的API冲突。
```javascript
    function Calculate() {}
    Calculate.prototype.add = function() {return this;}
    Calculate.prototype.sub = function() {return this;}
    Calculate.prototype.mul = function() {return this;}
```
假设我们的插件是对初始化参数进行运算并输出结果，则
```javascript
    // plugins.js
    ;(function(undefined) {
        "use strict"
        var _global;
        
        function result(args, type) {
            var argsArr = Array.prototype.slice.call(args);
            if(argsArr.length == 0) return 0;
            switch(type) {
                case 1:  return argsArr.reduce(function(p, c) {return p + c;});
                case 2:  return argsArr.reduce(function(p, c) {return p - c;});
                case 3:  return argsArr.reduce(function(p, c) {return p * c;});
                case 4:  return argsArr.reduce(function(p, c) {return p / c;});
                case 5:  return argsArr.reduce(function(p, c) {return p % c;});
                default: return 0;
          }
      }

      function Calculate() {}
      Calculate.prototype.add = function() { console.log(result(arguments,1)); return this; }
      Calculate.prototype.sub = function() { console.log(result(arguments,2)); return this; }
      Calculate.prototype.mul = function() { console.log(result(arguments,3)); return this; }
      Calculate.prototype.div = function() { console.log(result(arguments,4)); return this; }
      Calculate.prototype.sur = function() { console.log(result(arguments,5)); return this; }

    // 最后将插件对象暴露给全局对象
   _global = (function(){ return this || (0, eval)('this'); }());
   if (typeof module !== "undefined" && module.exports) {
         module.exports = Calculate;
    } else if (typeof define === "function" && define.amd) {
        define(function(){return Calculate;});
    } else {
        !('Calculate' in _global) && (_global.Calculate = Calculate);
    }
  }());
```
调用插件
``` javascript
    var plugin = new Calculate();
    plugin
        .add(2, 1)
        .sub(2, 1);
   // 结果
   // 3
   // 1
```

#### 编写UI组件
一般情况，如果一个js仅仅是处理一个逻辑，我们称之为插件，但如果与dom和css有关系并且具备一定的交互性，一般叫组件。
利用原型链，可以将一些UI层面的业务代码封装在一个小组件，并利用js实现组件的交互性




###  参考资料
1. [原生JavaScript插件编写指南](https://link.jianshu.com/?t=http%3A%2F%2Fgeocld.github.io%2F2016%2F03%2F10%2Fjavascript_plugin%2F)
2. [js原型链](https://www.cnblogs.com/alichengyin/p/4852616.html)

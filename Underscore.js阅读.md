#Underscore.js-源代码阅读笔记

这是我的第一次阅读源代码,因此选择了一个简单的类库:Underscore.js.

选择的主要原因也是因为这个类库短小精悍,都是一些静态函数,既容易阅读,也能学习技巧.

然后说明下阅读思路:我会先从0.1版本开始读,然后再读1.0版本,最后升级到最新版本(1.8.3)的阅读.

原因是所有的库在刚刚开始设计的时候,目的性总是最明确的,但是后来随着需要兼容和增强会增加很多代码.但是最核心的思路还是在最初就确定了.

所以直接从第一个版本开始阅读就能容易能够搞清什么是核心功能代码.然后慢慢升级去思考为什么他们需要去考虑这些问题.不然刚刚开始就阅读最新版本可能会一头乱码,理不清头绪;

很明显的是代码数量:
>[Underscore.js v0.1版:400+行](https://github.com/jashkenas/underscore/releases/tag/0.1.0)
>
>[Underscore.js v1.0版:700+行](https://github.com/jashkenas/underscore/releases/tag/1.0.0)
>
>[Underscore.js v1.8.3版:1600+行]((https://github.com/jashkenas/underscore/releases/tag/1.8.3)
>)
>

所以功能可能还是那么多,但是需要考虑的问题或者模式变多了,代码量也乘倍数增长.因此我会选择从最早的版本开始阅读;


##Underscore.js v0.1版

###总体结构
不得不说,最初版本的underscore.js确实十分简单,可以说简单到简陋了:

~~~javascript
window._ = {
	VERSION:"0.1.0",
	each:function(){},
	.....
}
~~~
相当于在window的环境下注册了一个"\_"变量,然后赋值一个对象;这个对象衔接了不少的函数方法,然后就通过"\_"方法来调用
>_.each([1,2,3],console.log);

Underscore.js的全部的函数分成了

* Collection Functionns
* Array Functions
* Function Functions
* Object Functions
* Utility Functions

接下来会依此来介绍这些函数的实现思路:

###Collection Functions
首先,什么是**Collection Functions**?

####1.each

##### 设计思路

所以思路是首先试探是否存在forEach,然后是数组那么自己遍历,如果有each那么执行;最后是一个对象,那么使用回掉函数处理对象包装成的list;

~~~javascript
each : function(obj, iterator, context) {
  var index = 0;
  try {
    if (obj.forEach) { //如果是数组,那么可能会有forEach原生函数,直接调用;
      obj.forEach(iterator, context);
    } else if (obj.length) { //可能是arguments类数组的对象,那么自己遍历;
      for (var i=0; i<obj.length; i++) iterator.call(context, obj[i], i);
    } else if (obj.each) { //可能是以前版本的underscore的each函数
      obj.each(function(value) { iterator.call(context, value, index++); });
    } else {
      var i = 0;
      for (var key in obj) { //如果不限定hasOwnproperty,可能会出现原型链上面的属性;
        var value = obj[key], pair = [key, value];
        pair.key = key; 
        //往list里面放属性之后会变 pair =[ 'key', 'value', key: 'key' ]
        pair.value = value; 
        iterator.call(context, pair, i++);
        //传入的第一个参数是这个对象,第二个参数是循环到什么地方了;
      }
    }
  } catch(e) {
    if (e != '__break__') throw e;
  }
  return obj;
}
~~~

##### Test

~~~javascript
1._.each([1,2],function(ele,i,list){console.log(ele,i,list)});
    1 0 [1,2]
    2 1 [1,2]
    
~~~





#### 2.map

map这个函数在ES6存在了




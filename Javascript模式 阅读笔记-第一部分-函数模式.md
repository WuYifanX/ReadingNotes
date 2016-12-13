#Javascript模式 阅读笔记-第一部分-函数模式
总的来说,javascript模式是一本力荐的js进阶书,书里面涉及了很多在学习javascript过程中会碰到的坑,然后提供了很不错的解决方法.虽然很多人吐槽这本书的翻译,但是糟糕的翻译还是无法掩盖这是一本好书的事实.
###1.关于new的一些事情
随处可见的是使用new操作符的构造函数

~~~javascript
var People = function(){ this.type = "human"}; //构造函数
var aPerson = new People();			//用new操作符来实例化
~~~
所以当在new操作符下的构造函数究竟发生了什么?

~~~javascript
//new操作发生的事情
var this = Object.create(People.prototype); //创建新的对象,然后继承自parent.prototype;
this.type = "human"; 		//属性和方法被加入到this下;
return this;				 //如果构造函数没有返回,则自己返回this;
~~~

现在有一个问题,如果用户在使用构造函数的时候,漏掉了new操作符,那么this就会被指向window/global上,从而出现错误;

~~~javascript
var aPerson = People(); //这里的this指向window
console.log(aPerson.type); //undefined
console.log(window.type);  //'human'
~~~
如果漏掉了new操作符可能会导致比较严重的问题.因此也会有了所谓的安全模式:

~~~javascript
var People = function(cfg){
	if(!(this instanceof People)){ //核心代码
		return new People(cfg);		//做一个简单的检测
	}
	.....//该干嘛干嘛
}
~~~
所以就算漏掉了new操作符,代码的检查机制也会帮你new,不会出现绑定的问题;

###2.使用回调模式来解耦和提高效率
回调函数是解耦的神器,如果你的函数内部有一个部分会频繁变化,可以考虑把这些部分封装到一个函数里面作为回调函数传入,这样就可以解除不同函数之间的耦合,并且会方便修改;

~~~javascript
var function complexFunction(){
	function A(); //变化少的部分
	function B(); //变化多的部分
	function C()  //变化少的部分
}

//重构之后:
var function complexFunction(fn){
	function A();
	fn();
	function C();
}
complexFunction(function B(){
	//变化很大的部分
});
~~~
这样就可以把会经常变化的部分从不变的函数部分中解耦出来;而且书中的例子更好,同样可以学习参考下:

~~~javascript
//原始代码:第一个先在node里面创建数据
var findNodes =function(){
	//大概的作用是在循环里面重复处理数据
	var i = 100000,
		node = [],
		found;
	while(i){
		i -= 1;
		node.push(found);
	}
	return nodes;
}
//这个函数的作用是循环数组来让他消失;为了解耦,所以分开了写;
var hide = function(nodes){
	var i =0, max = nodes.length;
	for(;i<max;i+=1){
		nodes[i].style.display = "none";
	};
};
hide(findNodes());
~~~
这样写的问题在于做这个事情需要循环两次,效率其实并不高;所以考虑用下面的方法重构:



~~~javascript
//重构:提高效率和解耦
var findNodes = function(callback){
	var i =100000,
		node=[],
		found;
	if( typeof callback !== "function"){
		callback = false; //这里单独判断是为了放在循环之外
	}
	while(i){
		i -=1;
		
		//这里执行很复杂的过程
		
		if(callback){
			callback(found);
		}
		nodes.push(found);
	}
}

var hide = function(node){
	node.style.display = "none";
};

findNodes(hide);
~~~
所以通过回调函数的会让函数解耦,特别是在循环体系之内;

###3.自定义函数的使用
如果一个函数有一些准备工作要做,但是这些工作也许只需要执行一次就好了,那么你需要用自定义函数来提高性能;

~~~javascript
var count = function(){
	var number = 0;
	console.log("Init Done!") //完成只会执行一次的初始化
	count = function(){
		//这里用新的函数覆盖原函数
		number++;
		console.log(number);
	}
}
Test:
count(); //Init Done;
count(); //1
count(); //2
~~~
所以完成了第一次的初始化,然后完成了一些初始化的工作.这里可以和下面的技巧一起联合使用;

###4.初始化分支选择
如果你开头就需要判断一些东西,然后之后这个判断都不需要执行,那么你可能需要使用这种方法;

说起来比较抽象,但是这个比较常见,比如说浏览器需要知道你正在使用的浏览器,但是这个信息一般只会有一次判断就可以了,比如说下面的方法效率就很低;

~~~javascript
var utils = {
	addListener:function(el,type,fn){
		if(typeof window.addEventListener ==="function"){
			el.addEventListener(type,fn,false);
		}else if(typeof document.attachEvent ==="function"){ // 判断这是IE
			el.attachEvent("on"+type,fn);
		}
	}
}
~~~
但是实际上这种信息我只需要判断一次就好;但是如果写在了utils里面,那么每次都需要执行;所以我们可以优化成为:

~~~javascript
var utils = { //这里只是简单的设置一个接口,实际的执行留在后面的初始化分支里面
	addListener:null
};
//
if(typeof window.addEventListener === "function"){
	utils.addListener = function(type,fn,false){
		el.addEventListener(type,fn,false);
	}
}else(typeof document.attachEvent ==="function"){
	utils.addListener = function(type,fn,false){
		el.attachEvent("on"+type,fn);
	}
}
~~~

###5.备忘模式
函数是一个对象,那么也就是可以在属性里面添加一些信息.比如说数组的length,函数的name属性就是本来就存在的;

那么我们也可以使用这个特性来给某些计算开销很大的函数,增加一个缓存,如果下次还需要计算这个值,就可以直接返回缓存中的量;

~~~javascript
var func = function(param){
	if(!func.cache[param]){
		var result = {};
		//做一些很复杂的计算
		func.cache[param] = result;
	}
	return func.cache[param];
};
myFunc.cache = {};
~~~

###6.配置模式
当你在做一个通用组件的时候,有时候你也需要留出一定的配置项出来让用户来设置;

~~~javascript
var Widget = function(cfg){
	cfg = cfg||{}; //如果传入了就用,没有的话就传入{};
	cfg.name && this.setName(cfg.name);
	cfg.theme && this.setTheme(cfg.theme);
	......//一系列的配置项
}
~~~
大概的思路就是说如果用户有传入配置项,那么就使用配置项.


###7.构造函数中的静态函数
之前学js的继承的时候,一直很揪心的问题就是,到底什么会继承,什么不会呢?这里刚刚做一个总结

~~~javascript
    var Gadget = function(){
        this.say1 = function(){ console.log("1~")};
        var say2= function(){ console.log("2~")};
    };
    Gadget.prototype.say3 = function(){console.log("3")};
    Gadget.say4 = function(){console.log("4")};

	//This is a test;
    var gadget = new Gadget();
    gadget.say1(); //1 通过this继承方法效率不高,但是实际上是可以使用的
    gadget.say3(); //3 最好的还是通过prototype的方法继承.正常访问;
    
    Gadget.say2(); //报错!在Gadget里面设置的会无法访问;
    Gadget.say4();// 4,正常使用,因此通过属性的方法添加到Gadget里面的是可以作为Gadget静态方法调用的
~~~





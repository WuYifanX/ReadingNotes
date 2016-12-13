#Javascript面向对象开发

##设计模式-工厂模式
创造工厂模式是一种`创建性模式`,也就是一种创建对象的最佳实践.首先我们需要理解:
###为什么我们需要工厂模式?

想象一个场景:如果你要求去买一些东西:`板烧鸡腿汉堡`,`可乐`和`薯条`,那么人们会非常自然的跑去麦当劳去购买对吧.

为什么我们会想到去麦当劳呢?因为这些东西都是一类食物,然后麦当劳作为一个'工厂',可以一条龙的提供给消费者,如果没有麦当劳,那么我们需要分别去可乐,薯条和板烧鸡腿汉堡的店面去分别买这些食物,那么购买效率会很低.所以可以说麦当劳就是一个销售食物的工厂模式.

所以我们可以这样理解工厂模式,把相关的多个类(薯条,可乐等)提供一个统一入口的一个模式,让你从一个入口就可以获得多个类,提高工作效率.

但是对于工厂模式也会有三种类型的实现方式,分别是:简单工厂模式,工厂方法模式和抽象工厂模式.它们分别是在各自基础上有一定的改进.

###简单工厂模式
也被叫做静态工厂模式(Simple Factory Patter),主要用于创建`同一类的对象`.

适用情况:如果被要求写一些球类的实现,那么一般情况的话我们会这样实现:

~~~javascript
var Football = function(){
	this.name = "football";
	....
}
var Basketball = function(){
	this.name = "basketball";
	....
}
~~~

这种写法有一些缺陷:`返回了多个类,不便于他人使用.`因此我们考虑把这些类似的类封装到一个工厂里面,也就有了我们的简单工厂模式:

~~~javascript
var BallFactory;
!(function(){
    BallFactory = function(type,cfg){
        var Football = function(cfg){
            this.name = 'football';
            console.log(this.name + " in the prototype");
        };
        Football.prototype = {
            call:function(){console.log(this.name)},
            sell:function(){}
        };
        var Basketball = function(cfg) {
            this.name = "basketball";
            console.log(this.name);
        };

        var cfg = cfg ||{};
        switch(type){
            case 'football':
                return new Football(cfg);
                break;
            case 'basketball':
                return new Basketball(cfg);
                break;
        }
    };
})();

var aFootball = BallFactory('football');
aFootball.call();
~~~

因此使用BallFactory把这些内容包裹起来给其他人使用就会避免返回了多个类的问题,所以这就是简单工厂模式想解决的问题:
`统一创建的入口`.

当然在实际的使用过程中,我们会使用一些其它技巧:

如果工厂的产品中有很多重复部分,那么我们需要把重复的部分抽象出来成为共同的部分,不同的部分放入switch里面:

~~~javascript
var GetChildren = function(cfg){
	cfg = cfg||{};
	this.name = cfg.name;
	this.height = cfg.height;
	this.speak = function(){};
	
	//抽离出不同的部分
	switch(cfg.gender){
		case "boy":
			this.gender = cfg.gender;
			this.moustouch = ....
			.....; //特有部分
			break; 
		case "girl":
			this.gender = cfg.gender;
			.......
			break;
	}
	return this;
}
var aBoy = GetChildren({.....}); 
~~~


###工厂方法模式

在简单工厂模式的基础上,我们已经解决了入口不统一的问题,但是还有一个问题没有解决:

加入一个新的类需要修改多部分:首先我们需要在BallFactory工厂内部加入如何实现,然后加到switch部分;所以这是一次修改的需求,我们需要修改多个地方.

那么我们看能不能尝试更抽象一点,**尽可能减少需要修改的地方**;

~~~javascript
var BallFactory = function(type,cfg){
    this.name = cfg.name; //共同的部分放在这里
    return this[type](cfg);
};

BallFactory.prototype = {
    football:function(cfg){
        console.log("这里加入和football相关的独特内容" +this.name);
    },
    basketball:function(cfg) {
        console.log("这里加入和basketball相关的独特内容" +this.name);
    }
};
var aBall = new BallFactory("football",{name:"football"}); //football in the prototype

~~~

所以这里加入了一个`return this[type](cfg)`的方法自动代替了之前的switch的方法.以后需要加入内容只需要修改BallFactory的prototype就可以了.

当然我们还可以添加一种`安全模式`来解决如果不在构造函数前面加上new的话,会报错的问题.解决的思路其实是把**new封装在构造函数之内**:

~~~javascript
var BallFactory = function(type,cfg){
    if(!(this instanceof BallFactory)){
        return new BallFactory(type,cfg); //多一行判断即:如果没有带new,我自己帮你new一个返回就好;
    }
    this.name = cfg.name;
    return this[type](cfg);
};

var aBall = BallFactory("football",{name:"football"}); //这里如果掉了new也会正常执行;
~~~


所以,我们可以看到的是,我们使用**简单工厂模式解决了入口不统一的问题**,然后使用**工厂模式解决了修改地点不统一的问题**

###抽象工厂模式

一般来说,抽象工厂在大型项目的使用更多,大概的思路是在父类里面设计好接口(没有具体实现),具体的实现等到了子类再重写.

这里借用一个张容铭的<JavaScript设计模式>的一个例子:

~~~javascript
var Car = function(){};
Car.prototype = {
    getPrice:function(){ throw new Error("抽象方法不能调用")},
    getSpeed:function(){ throw new Error("抽象方法不能调用")}
};
//这里使用Object.create()继承,子类到父类中会多一个中间过渡函数Temp(){};防止在子类的prototype覆盖父类;可见参考资料
aBMW = Object.create(Car.prototype);
    
aBMW.getPrice();  // 抽象方法不能调用
aBMW.getPrice = function(){
    console.log("I am getting price");
};
aBMW.getPrice(); //I am getting price
~~~

父类定义好接口,具体实现延迟到子类才实现.

所以总结来说:


**简单工厂模式解决了入口不统一的问题**,

**工厂模式解决了修改地点不统一的问题**,

**抽象工厂模式解决了子类实现不规范的问题**



##参考资料

[Javascript设计模式 张容铭](https://book.douban.com/subject/26589719/)

[深入理解JavaScript系列（28）：设计模式之工厂模式](http://www.cnblogs.com/TomXu/archive/2012/02/23/2353389.html)

[工厂模式](http://www.runoob.com/design-pattern/factory-pattern.html)

[Object.create()使用方式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)
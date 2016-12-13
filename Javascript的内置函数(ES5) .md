#Javascript的内置函数(ES5)
整理了一些`Javascript语言精粹`的方法一章的整理出的ES5的内置方法和ES6标准入门的ES6方法;

整理这些作用一方面是更好的理解,另一方面是对于类数组arguments,可以使用原型链的调用即可;

##ES5中的一些函数
###Array
#####1.array.concat(item....)
>var a = [1,2,3];
>
>b = a.concat([4,5,6]);

输出的b为a+b的结果[1,2,3,4,5,6];因此array.concat的作用是连接数组,当然item可以是数字也会加入到里面;

#####2.array.join(separator)
作用是把一个数组变成一个字符串,中间使用separator连接;
>var a = [1,2,3];
>
>b = a.join(""); //b = "123"
>
>c = a.join(","); //c = "1,2,3"
>

#####3.array.pop() && array.push()
数组的标准操作,大概的思路是返回和添加一个元素到最后一个;

然后push返回的是加入之后的数组长度;pop返回的是数组的被踢出的值;

共同的特性就是都会改变数组原来的数值;

#####4.array.reverse();
数组原地逆转,然后返回这个逆转的数组;

#####5.array.shift();
移除数组头元素,并且返回,速度并没有pop快;
#####6.array.unshift();
把数组加到头元素,返回新的数组的长度;

#####7.array.slice(start,end);
类似于python的切片;返回的是浅复制的数组
#####8.array.splice(start,deleteCount,item..)
从start的点开始删除deleteCount个数的元素,然后再这个地点加入item元素;返回删除的元素

>var a= [1,2,3];
>
>var b = a.splice(1,2,[1,2,3]) // a = [1,[1,2,3]]; b = [2,3]

#####9.array.sort(comparefn);
如果不加入comparafn的情况下,sort函数会按照字符串的形式来排序;

~~~javascript
> a = [1,22,33,12,112]
[ 1, 22, 33, 12, 112 ]
> a.sort()
[ 1, 112, 12, 22, 33 ]
~~~
因此我们在使用sort函数的时候,我们应该使用comparefn;下面说下这个函数的作用机制:

比较函数comparefn接受两个参数:a,b;

然后comparefn会返回一个数值,如果返回的值大于0,那么就调换顺序,如果小于0,那么就不会调换顺序;

~~~javascript
aList = [1,2,3];
ab = function(a,b){return a-b};
ba = function(a,b){return b-a};

aList.sort(ab); //[1,2,3]
aList.sort(ba);	//[3,2,1] 因为依此遍历b-a,然后大于0就调换位置;
~~~
然后语言精粹里面加入了更加复杂的一些比较函数:

~~~javascript
var m = ['aa','bb','a',4,8,15,16,23,42];
m.sort(function(a,b){
    if(a===b){ 
        return 0;
    }
    if(typeof a ===typeof b){
        return a < b? -1:1;
    }
    
    //'string' > 'number',会让number在前面
    return typeof a < typeof b? -1:1;
});


// m为 4,8,15,16,23,42,a,aa,bb

~~~
然后继续拓展一个更加高级的比较方法,用来比较对象里面的属性的排序:

~~~javascript
var s = [
    {first:"Joe",last:"Besser"},
    {first:"Moe",last:"Howard"},
    {first:"Joe",last:"Derita"},
    {first:"Curly",last:"Howard"},
    {first:12,last:"Howard"}

];

//这里的direction为 1或者-1.其中-1为正序(升高排列);
var by = function(name,direction){
    return function(o,p){
        var a,b;
        //下面是为了首先判断是object,而且不是null;
        if(typeof o === "object" && typeof p ==="object" && o&& p){
            a =o[name];
            b = p[name];
            if( a=== b){
                return 0;
            }
            if(typeof a === typeof b){
                return a<b ? direction:-direction; 
                //意思是如果是统一的类型,那么就正序排列
            }
            return typeof a < typeof b? direction:-direction;
        }
    }
};
s.sort(by("first",-1));
~~~

如果还需要比较第二个last的排序,那么可以继续拓展,改变上面如果a===b的时候排序方法:

~~~javascript

var by = function(name,minor,direction){
	.....//不变的部分
	if(a===b){
		return typeof minor ==="function"? minor(o,p):0;
	}
	.....//不变
}


//相当于递归了一次调用方法是
s.sort(by("first",by("last"),-1));
~~~
当然我也在这里碰到了一个问题

~~~javascript
//如果只是调用一次排序函数,那么这样是没问题的
s.sort(by("first",-1));
//如果两个同时调用:
s.sort(by("first",1));
s.sort(by("first",-1));
//返回的对象的排序方式是一模一样的;
//所以这样的话,也就是第二个方法其实并没有奏效;
//查了下似乎是回调机制问题,以后再研究;
~~~

###function

#####1.function.apply(thisArg,argArray);
其实call和apply很像,call只是apply的语法糖写法;

apply的作用还是显式的绑定this到函数上面;

详细的可见你不知道的Javascript上册

###Object
#####1.object.hasOwnProperty(name)
最有用的地方就是遍历对象的属性了

###Number
#####1.number.toExponential(fractionDigits)
转化为字符串,用指数形式表示,并且选择保留小数点后几位的问题

~~~javascrpt
> Math.PI.toExponential(2)
'3.14e+0'
> Math.PI.toExponential(3)
'3.142e+0'
> Math.PI.toExponential(5)
'3.14159e+0'
~~~

#####2.number.toFixed(fractionDigits)
转化为字符串,并且选择保留小数点后几位的问题

~~~javascript
> Math.PI.toFixed(2)
'3.14'
> Math.PI.toFixed(3)
'3.142'
~~~

#####3.number.toString(radix)
数字转化为字符串,radix是控制多少进制

~~~javascript
> Math.PI.toString()
'3.141592653589793'
> Math.PI.toString(2)
'11.001001000011111101101010100010001000010110100011'
> Math.PI.toString(8)
'3.1103755242102643'
~~~

###String

#####1.string.indexOf(searchString,position)
寻找string里面的字符串里面的字符串,而且从position的位置开始;没找到返回-1

~~~javascript
> var text = "Mississippi"
> text.indexOf("ss")
2
> text.indexOf("ss",2)
2
> text.indexOf("ss",3)
5
> text.indexOf("ss",6)
-1
~~~

#####2.string.match(regexp)
和正则表达式匹配..以后再更新


#####3.string.replace(searchValue,replaceValue)
replace方法对string进行查找和替换操作,并且返回一个新的字符串;

searchValue可以是正则表达式,如果searchvalue是普通的字符串,那么只会替换第一个

~~~javascript
> text
'Mississippi'
> text.replace("ss","SS")
'MiSSissippi'
> text
'Mississippi'
> text.replace("ss","SS").replace("ss","SS")
'MiSSiSSippi'
~~~
下面是和正则表达式的匹配:


#####4.string.search(regexp)
string的search方法和indexOf方法很相似,但是只能接受正则表达式对象;

#####5.string.slice(start,end)
复制string的一个部分来构造新的字符串,这个方法和array.slice()很类似;

~~~javascript
> text
'Mississippi'
> text.slice(1)
'ississippi'
> text.slice(1,3) //end - start 的个数差是需要书要输出的个数
'is'
~~~

#####6.string.split(separator,limit)
把字符串用separator分解成为一个数组,然后这个separator可以是正则表达式对象,
limits表示了这里需要多少个;

~~~javascript
> a = "123456789"
'123456789'
> a.split("")
[ '1', '2', '3', '4', '5', '6', '7', '8', '9' ]
> a.split("",5)
[ '1', '2', '3', '4', '5' ]

> f = "a|b|c|"
'a|b|c|'
> f.split(/\|/)
[ 'a', 'b', 'c', '' ]

~~~

#####7.string.toLowerCase()/toUpperCase()
两个方法分别返回一个新的字符串,string被转化成为大写和小写的格式


###参考资料

[JavaScript语言精粹](https://book.douban.com/subject/3590768/)
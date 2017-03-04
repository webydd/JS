### 第一周 第三天（this和面向对象）
>如果函数名字和变量重名，在预解释阶段以最后一个函数为准；当代码执行到变量赋值的那一行，以这个变量为准

    fn(); // 2
    function fn(){ console.log(1); }
    fn(); // 2
    var fn = 5; // 代码执行到这一行fn这个变量再也不会代表打印2这个函数了
    typeof fn == 'function' && fn(); //  fn is not a function
    // cannot set property 'className' of undefined , lis[i].className = 'active'
    function fn(){ console.log(2); }
    if(typeof fn == 'function'){ fn(); } // 不打印

####1、(函数中)this用法 :
        1 绑定事件函数中的this就是绑定的那个元素 => 事件绑定给谁this就是谁
        2 函数执行时候(函数中的this)就是调用这个函数的宿主 => '.'前面是谁这个函数中的this就是谁
          ps : 只要有函数包含this那么this就一定已经发生了改变
          ps : 只有函数执行的那一刻才能确定函数中的this   比如 : fn()  obj.fn()
          ps : fn() 和 obj.fn() 执行的是同一个函数，由于执行方式不同导致this不同
      3 自运行函数中的this就是window   (非严格模式下)
       4  定时器函数中的this一般是window
       5 构造函数中的this是实例
       6 call和apply&bind都可以强制修改this
       7 回调函数中的this一般也是window

>- 实例一

     var obj = {
        name : 'zf',
        fn : function () {
            console.log(this);
        }
    };

    obj.fn(); // 这样执行fn函数中的this是obj => 也就是'.'前面的这个对象

>- 实例二
  

    function fn(){
        console.log(this);
    }
    var o = {
        fn : function (){
            console.log(this); // o
            fn(); // window
        }
    };
    o.fn(); 
>- 练习一：利用this 和作用域释放练习

    var num = 5; // window.num = 5 // 6 // 7
    var obj = {
        num : 4, // 5
        fn : (function (num){
            // num : undefined
            num++; // num : NaN
            this.num++; // window.num++ => 全局变量num从5修改成了6
            var num = 2; // num : 2  3  4
            return function (){
                num++;
                this.num++; // window.num++   obj.num++
                console.log(num);
            }
        })()
    };
    // 全局变量已经从5修改成了6，自运行函数作用域没有释放里的私有num=2
    var fn = obj.fn; // 全局fn和obj.fn共同引用同一个函数
    fn(); //3 把自运行函数中那个私有变量num=2修改成了3，由于this是window，还把全局变量num从6修改成了7
    obj.fn(); //4 由于和fn是同一个函数，所以同样也会把上一级也就是自运行函数中的num从3修改成了4,由于this是obj,那么把obj.num从4修改成了5
    console.log(this.num,obj.num); // 7 5
>- 练习二

    var num = 1; // window.num = 1 , 3  5
    var obj = {num : 2}; // obj.num = 2  4
    obj.fn = (function (num){
        // var num = 0;  -1  0  -1 0
        this.num += 2; // window.num += 2;
        num--;
        return function (n){
            this.num += 2;
            num--;
            console.log(n + ++num); // 1  2
        }
    })(this.num); // 1
    var fn = obj.fn;
    fn(1); // 1
    obj.fn(2); // 2
    console.log(num,obj.num); // 5 4
  >- 练习三
  

    var length = 5;
    var fn = function (){

        console.log(this.length);
    };

    var main = {
        fn : function (fn){

            fn(); // fn是获取的就是全局fn函数。那么在这里执行fn函数中的this是window。打印的就是window.len => 5

            arguments[0](); //1   <=> arguments.length=1

        },
        length : 10
    };

    main.fn(fn);

####2、面向对象
**一、单例模式：**
其实就是一个对象，解决命名冲突的问题，模块化方便代码管理  常用
                命名空间 => nameSpace
                ps: 单例模式下每个属性和属性之间的调用使用this就可以
                比如 : 在guonei.fn1中调用fn2就直接在fn1函数中this.fn2即可

     var guonei = {
        num : 100, //
        fn1 : function (){
            //guonei.fn2();
            this.fn2(); // this => guonei => guonei.fn2()
        },
        fn2 : function (){
            this.num;
        },
        fx : function (){
            this.fn2();
        }
    };

    guonei.fn1();
 **二、构造函数（类）**
 >内置类 Number String Boolean Object Array Function ...
        实例 :  具体到类中的某一个。每一个实例都是一个对象数据类型
             ps:  每一个数组都是Array这个类的一个实例
             ps : 每一个函数都是Function这个类的一个实例， 无论是实名还是匿名自运行
             ps : 咱们每一个同学都是Chinese中国人这个类的一个实例
             ps : instanceof 专门判断一个实例是否属于一个类,返回布尔值
             Array instanceof Function 说明Array这个类也是一个函数
             ps : 任何一个引用数据类型都是Object这个类的实例
             
>- 如何定义一个类 ： function Tab(){} 定义函数相同
            ps : 构造一个类尽量使用首字母大写
             构造函数中的this就是当前实例 => 如果想在实例添加私有属性 this.属性名
             函数函数会默认返回一个实例，但是不能再函数中再写return一个引用数据类型
    
# **EJack函数式高精度科学计算器**
EJack是一个可编程的科学计算器，除了支持基本的科学运算外，我们还支持变量绑定、Lambda演算、函数声明、递归、条件语句等等功能。
**注意：如果不能通过编译，请将numeric_cfg.h中的常数CALSTACK_NBYTES_MAX修改为65536**

---
## 基本的科学函数与算数计算

我们允许您在算数表达式中使用基本的四则运算和括号，以组成复杂的表达式：
>　　2*(5-8)/6+9  

我们支持以下科学函数：  
>　　sin()、cos()、tan()、arcsin()、arccos()、arctan()、exp()、x^y、ln()、log2()、lg()、nlg(y)、sqrt()、abs()、fact()  

除此之外，我们也做了双曲函数等等，但是未被启用（纯粹是我懒），这些函数你可以在scientific.c中找到他们的实现。

每个科学函数中，你可以自由的嵌套其他函数，不过请不要在一个算数表达式中输入空格，这很有可能引起解析错误（我也不知道会不会出bug，实际上我为了消除空格做了很多处理，但不保证它总是可用的）

---
## 变量的绑定
我们允许你自定义变量，变量名varName不超过16个字母，且只允许由字母组成。我们的变量定义形式如下：  
>　　　let varName = expression  

这种形式下，expression可以是任何合法表达式（因为无论什么表达式都存在返回值，且返回值是Numeric和Closure中的一种）

我们也允许你如此操作：  
>　　let [varName = exp1] in exp2  
这样定义的变量只会作用在exp2中，而不能在其他地方使用。
例：  
>let x = 1 in x^2  
->2  
x  
->Error

另外一种形式的let是letrec，它被创造的主要目的是声明一个可以递归的函数。let声明的函数不支持递归，letrec是他的替代品，但letrec暂时不能使用letrec in （其实可以，但被我注释了，我觉得做的不够完善）。
例：  
>let f = $x{if x == 1 then 1 else x*f[x-1]}  
->Closure  
f[5]  
->120  

无论是何种形式的let，都可以自由嵌套，他们的返回值都是最后的expression的计算结果。

---
## lambda表达式、单元函数与闭包
我们的函数都是有闭包（可以理解为词法作用域）的函数，我们通过让EJack支持lambda表达式来实现单元函数。
你可以这样书写以创建一个有闭包的lambda表达式:
>$x{ x+1 }  
->Closure  

其中，‘$’号后面是这个函数的形式参数（就暂时当做是形式参数吧），花括号内的部分是返回值。

你也可以在一个函数内部声明变量，这样的变量是局部变量，不会对闭包外产生影响。
>$x{let [y = 1] in let [z = 5] in x+y+z}  
->Closure  
 $x{let [y = 1] in let [z = 5] in x+y+z }[4]  
->10 

---
## 函数调用
函数调用可以通过三种方式进行，其一是使用let in，这种方式我们已经介绍过了，其二则是通过闭包定义后直接加上'[value]'的形式，第三种则是通过函数名调用。
> $x{x+5}[5]  
->10
let f = $x{x+5}
->Closure   
f[10]  
->15  
---
## 函数是一等公民(First Class Function)
我们的函数作为一等公民，与普通变量享有相同的地位。我们可以自由的使用一个函数作为函数的参数传入、返回值返回，因此我们可以实现很多非常强大的功能。借此我们可以通过单元函数实现多元函数和函数返回函数接受参数继续返回函数这种奇怪操作。
**Example：**
>let g = $y{y^2}
let t = $z{z^3}  
->Closure    
let f = $x{if x==0 then g else t }  
->Closure    
f[0][3]  
->9  
f[1][3]  
->27  
let h = $x{ $y{ $z{x+y+z}}}  
->Closure
h[2][3][5]  
->10
---
## 条件语句
我们支持一种类似三目运算符“ ? :”的if语句。  
语法：if condition then exp1 else exp2
其中，condition为E1 boolOpeartion E2。E1、E2均为合法表达式。  
例：
>letrec f = $x{if x==1 then 1 else x*f[x-1] }  
->Closure
f[5]
->120

---
## 微积分
我们的EJack支持微分的运算（积分其实也能算，但是太慢了，被我砍掉了），具体的语法如下：
>diff f 2  

例:
>let f = $x{x^2}  
diff f 4  
->8
---
## 其他一些被我砍掉的功能
### **AST抽象语法树**
这东西应该是一个解释器的核心部分，然而我当时没看懂这东西该如何实现（因为我那时还不知道什么是树），等我了解了这个AST该如何实现的时候，我已经写了几千行了，因此最终我们的解释器实际上是用字符串解析的方式完成的，把字符串看成一颗AST，然后解释器就是树遍历算法。这样做肯定有很大弊端，每次我们调用闭包的时候，都要重新解析闭包内的字符串，这非常Naive。（没事很快我就会重构）
### **FHQ Treap**
这是一种非旋平衡树，在这里是用来更好的实现环境的一种结构。然而没有做出来的理由还是我数据结构没上过，不知道如何写这东西（现在可能都不会）。  
但是这很重要，也很高效，下次重构肯定会把它加上。
### **TUPLE**
也就是元组，一个不可更改的列表，形如:(1,2),(1,(2,3))这样。  
砍掉的理由是:我忘了做了(大哭)。  
元组是非常好的一个东西，有了这个东西，我们就可以进一步下面的三个方法。
### **MAP、REDUCE、FILTER**
这三个非常典型的高阶函数，由于我没做元组，所以都被我砍掉了。事实上，元组和这三个玩意实现都不困难(但是我懒的去debug)，我会在这个暑假完成这项内容。
## **模拟Class**
这是我2019-4-11的突发奇想，我希望给闭包加一个类似结构操作符一样的东西，让我们可以通过f->var的形式，访问到闭包内部的变量，再让闭包支持一个"add x 3 to f"形式的操作,允许我们向一个闭包的环境里面在外部添加变量。然后我们可以设计一个闭包，让他具备几个成员变量，由于函数也是变量，所以我们可以给这个闭包一个constructor，这个函数会根据传入的参数和闭包内的成员变量值返回一个新的函数。   
这样，在某种意义上，我们就实现了一个类似于Class的东西。（当然，都是我乱讲的，能不能实现还不好说）
## **WHILE**
while这种东西大家都懂，这是挺重要的一种结构，本来我已经做好了，后来我突然发现对于一个计算器来说，while不是什么必要的东西，而且我发现如果放弃While，我依然可以图灵完备，而且只需要一条语句就足够图灵完备了，故砍掉，未来可能还会加回来。
## **doNothing**
这用来表示一个语句被规约后的最终结果。doNothing顾名思义就是什么都不做，它自然也是不可规约的。因为我放弃了多条语句和while，同时希望所有的语句返回值都是有意义的，因此砍掉，未来可能也会加回来。
## **分号（列表）**
在<计算的本质>这本书中，我曾经学习过实现多条语句的一种方法，就是将多个语句通过分号组合成一个列表，支持了列表也就是支持了多语句，这个不难，我最开始就已经写入了，但最终还是因为我觉得多语句现在做不是很合适（毕竟我们是计算器），故砍掉（其实没砍干净，语句结束还是可以用分号的，可能也可以在一行中输入多个分号隔开的句子）。
## **面向对象**
这个实现起来很困难，我们的结构决定我们最好不要去碰OOP这种想法，但是我很想做，可能暑假会做吧。

---
---
以上是这个EJack的基本功能，因为我本人是负责解释器部分的，对高精度和科学函数的实现不是很熟悉，这两个部分请您们看我们的技术报告。
  
本项目作者及分工：
                                        
                                        解释器：狄志鹏
                                        高精度：王浩任
                                        科学函数：林隆中

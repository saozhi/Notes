# gcc与g++

*Utuntu 16.04自带gcc与g++*

## gcc

1. 编译C文件

> gcc file.c
> 默认生成a.out文件

1. 执行a.out文件

> ./a.out

1. 其他参数和命令

- gcc -Wall file.c -o b //这里的Wall（W必须大写）  表示warning all
  b 表示生成可执行文件名字 （默认生成a.out）
- gcc --help

## g++

1. 编译Cpp文件

> gcc file.cpp
> 默认生成a.out文件

1. 执行a.out文件

> ./a.out

1. 其他参数和命令

- g++ -Wall file.cpp -o b //这里的Wall（W必须大写）  表示warning all
  b 表示生成可执行文件名字 （默认生成a.out）
- g++ --help

## 误区一：gcc只能编译c代码，g++只能编译c++代码

两者都可以，但是请注意：

1. 后缀为.c的，gcc把它当作是C程序，而g++当作是c++程序；后缀为.cpp的，两者都会认为是c++程序，注意，虽然c++是c的超集，但是两者对语法的要求是有区别的，
   例如：

   ```c
   #include <stdio.h>
   int main(int argc, char* argv[]) 
   {
      if(argv == 0)
    return; 
     printString(argv);
     return;
   }
   int printString(char* string)
    { 
    sprintf(string, "This is a test./n");
   }
   ```

   如果按照C的语法规则，OK，没问题，但是，一旦把后缀改为cpp，立刻报三个错：“printString未定义”；“cannot convert `char**' to`char*”；”return-statement with no value“；分别对应前面红色标注的部分。可见C++的语法规则更加严谨一些。

2. 编译阶段，g++会调用gcc，对于c++代码，两者是等价的，但是因为gcc命令不能自动和C＋＋程序使用的库联接，所以通常用g++来完成链接，为了统一起见，干脆编译/链接统统用g++了，这就给人一种错觉，好像cpp程序只能用g++似的。

## 误区二：gcc不会定义__cplusplus宏，而g++会

实际上，这个宏只是标志着编译器将会把代码按C还是C++语法来解释，如上所述，如果后缀为.c，并且采用gcc编译器，则该宏就是未定义的，否则，就是已定义。

## 误区三：编译只能用gcc，链接只能用g++

严格来说，这句话不算错误，但是它混淆了概念，应该这样说：编译可以用gcc/g++，而链接可以用g++或者gcc -lstdc++。因为gcc命令不能自动和C＋＋程序使用的库联接，所以通常使用g++来完成联接。但在编译阶段，g++会自动调用gcc，二者等价。

## 误区四：extern "C"与gcc/g++有关系

实际上并无关系，无论是gcc还是g++，用extern "c"时，都是以C的命名方式来为symbol命名，否则，都以c++方式命名。试验如下：me.h：

```c
extern "C" 
void CppPrintf(void); 
```

me.cpp:

```c++
#include <iostream>
#include "me.h"
using namespace std;
void CppPrintf(void)
{    
 cout << "Hello/n";
} 
```

test.cpp:

```c++
#include <stdlib.h>
#include <stdio.h>
#include "me.h"    
int main(void)
{ 
   CppPrintf();    return 0;
} 
```

1. 先给me.h加上extern "C"，看用gcc和g++命名有什么不同

   ```shell
   [root@root G++]# g++ -S me.cpp
   [root@root G++]# less me.s
   .globl _Z9CppPrintfv        //注意此函数的命名
   .type   CppPrintf, @function
   [root@root GCC]# gcc -S me.cpp
   [root@root GCC]# less me.s
   .globl _Z9CppPrintfv        //注意此函数的命名
   .type   CppPrintf, @function完全相同！
   ```

   

   2.去掉me.h中extern "C"，看用gcc和g++命名有什么不同

   ```shell
   [root@root GCC]# gcc -S me.cpp
   [root@root GCC]# less me.s
   .globl _Z9CppPrintfv        //注意此函数的命名
   .type   _Z9CppPrintfv, @function
   [root@root G++]# g++ -S me.cpp
   [root@root G++]# less me.s.globl _Z9CppPrintfv        //注意此函数的命名
   .type   _Z9CppPrintfv, @function完全相同！
   ```

> 作者：五秋木
> 链接：https://www.jianshu.com/p/2eade5c996ad
> 来源：简书
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。







# 句柄(类)

- 句柄类（[智能指针](https://www.baidu.com/s?wd=智能指针&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)smart point）是存储指向动态分配（堆）对象指针的类。除了能够在适当的时间自动删除指向的对象外，工作机制很像C++的内置指针。[智能指针](https://www.baidu.com/s?wd=智能指针&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)在面对异常的时候格外有用，因为能够确保正确的销毁动态分配的对象。也可以用于跟踪被多用户共享的动态分配对象。
- 在C++中一个通用的技术是定义包装（cover）类或句柄（handle）类，也称[智能指针](https://www.baidu.com/s?wd=智能指针&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)。句柄类存储和管理版基类指针。指针所指向权对象的类型可以变化，它既可以指向基类类型对象又可以指向派生类型对象。用户通过句柄类访问继承层次的操作。因为句柄类使用指针执行操作，虚成员的行为将在运行时根据句柄实际绑定的对象类型而变化，即实现c++运行时动态绑定。故句柄用户可以获得动态行为但无需操心指针的管理。







# printf输出string类

string 是C++中的字符串。 字符串对象是一种特殊类型的容器，专门设计来操作的字符序列。 不像传统的c-strings,只是在数组中的一个字符序列，我们称之为字符数组，而C++字符串对象属于一个类。

eg.

```c++
#include"cstdio"
#include"string"
#include"algorithm"
#include"iostream"

using namespace std;
int main(){
    string a[5];
    for(int i=0;i<=2;i++)
        scanf("%s",a[i].c_str()); //c_str()函数把a[i]转换成c-string
    sort(a,a+3);

    printf("%s\n",a[0].c_str());

    return 0;
} 
```

如上： 
 **c_str()** 函数,就可以把string类型得到等效字符数组， 然后就可以用scanf()/printf()函数，进行输入/出。



# 命名空间

在C语言中只有一个全局作用域：
1.C语言中所有的全局标识符共享一个作用域
2.标识符之间可能发生冲突

C++中提出了命名空间的概念：
1.命名空间将全局作用域分成不同的部分，
2.不同命名空间中的标识符可以同名而不会发生冲突
3.命名空间可以发生嵌套
4.全局作用域也叫默认命名空间

```c++
    namespace Name
    {
    	namespace Internal
    	{
    		/*...*/
    	}
    	/*...*/
    }
```

C++命名空间的使用：

| 使用整个命名空间：using namespace name;        |
| ---------------------------------------------- |
| 使用命名空间中的变量：using name**::**variable |
| 使用默认命名空间中的变量: **::**variable       |



# 数据结构

## 双端队列

std::deque是双端队列。

双端队列（Double-ended queue，缩写为Deque）是一个大小可以动态变化（Dynamic size）且可以在两端扩展或收缩的顺序容器。顺序容器中的元素按照严格的线性顺序排序。可以通过元素在序列中的位置访问对应的元素。

相比向量，双端队列不保证内部的元素是按连续的存储空间存储的，因此，不允许对指针直接做偏移操作来直接访问元素。



## 向量

### 向量赋值

eg.

```c++
// 如下, 定义消息变量 point_cloud_rviz_
sensor_msgs::PointCloud     point_cloud_rviz_;
```

上面消息结构如下：

<img src="/home/liwenzhi/.config/Typora/typora-user-images/image-20200730154040252.png" alt="image-20200730154040252" style="zoom:80%;" />

我们要在Rviz上显示点，需要用到中间 points 消息

points消息为数组，查看在代码中定义

![image-20200730154335345](/home/liwenzhi/.config/Typora/typora-user-images/image-20200730154335345.png)

显然，points是一个向量，成员为 geometyr_msgs/Point32 类型的消息：

![image-20200730154550617](/home/liwenzhi/.config/Typora/typora-user-images/image-20200730154550617.png)

消息 结构：

sensor_msgs/PointCloud

—— std_msgs/Header header

—— geometry_msgs/Point32[] points
		—— float32 x
		—— float32 y
		—— float32 z

—— sensor_msgs/ChannelFloat32[] channels



**注意**，往变量 point_cloud_rviz_ 中写消息 points 时，不能直接赋值x/y/z，而应该整体push一个消息：

```c++
sensor_msgs::PointCloud     point_cloud_rviz_;	// 定义PointCloud消息变量   


// 数据源为charging_pose_.pose.position, geometry_msgs/Point 类型
// float64 x
// float64 y
// float64 z

//

// 错误, vector成员不是int/double这种的单个变量，不能直接单个赋值
point_cloud_rviz_.points[0].x = (double)(charging_pose_.pose.position.x);


// 正确，先构造一个geometyr_msgs::Point32类型的变量，赋值成员，然后push到Topic变量point_cloud_rviz_的points成员
geometry_msgs::Point32      points_rviz_;		// 定义PointCloud中的一个msg变量

points_rviz_.x = (double)(charging_pose_.pose.position.x);
points_rviz_.y = (double)(charging_pose_.pose.position.y);
points_rviz_.z = (double)(charging_pose_.pose.position.z);

point_cloud_rviz_.points.push_back(points_rviz_);
```







# 函数



## atoll函数

将字符串转化为long long类型变量

| atoi | 将字符串转化为int类型变量       |
| ---- | ------------------------------- |
| atol | 将字符串转化为long类型变量      |
| atol | 将字符串转化为long long类型变量 |
| atof | 将字符串转化为double类型变量    |

这些函数的转化过程，都是将一个字符串的可读部分取到变量中

遇到不可读的部分，则直接终止读取



## 函数声明后跟 throw()

成员函数声明后面跟上throw()，表示告诉类的使用者：我的这个方法不会抛出异常，所以，在使用该方法的时候，不必把它至于 try/catch 异常处理块中。

```c++
void fun() throw() 				//表示fun不允许抛出任何异常，即fun是异常安全的。

void fun() throw(...) 			//表示fun可以抛出任何形式的异常。

void fun() throw(exceptionType)	//表示fun只能抛出exceptionType类型的异常。
```



## 函数 - Maths

fabs()		示绝对值

| 函数名 | 功能                        | 头文件                                                     |
| ------ | --------------------------- | ---------------------------------------------------------- |
| abs    | 对 **整数** 求绝对值        | stdlib.h  或  \<cstdlib\>  <br>#include \<cmath>也可以使用 |
| fabs   | **double float **型求绝对值 | \<cmath\>                                                  |
|        |                             |                                                            |



## boost::bind

用于将一个接口适配为另一个接口,使得函数的接口更加通用

boost::bind是标准库函数std::bind1st和std::bind2nd的一种泛化形式。其可以支持函数对象、函数、函数指针、成员函数指针，并且绑定任意参数到某个指定值上或者将输入参数传入任意位置。

bind 是一组重载的函数模板. 用来向一个函数(或函数对象)绑定某些参数. 
bind的返回值是一个函数对象. 

**bind**接收的**第一个参数**必须是一个**可调用的对象f**，包括**函数**、**函数指针**、**函数对象**、和**成员函数指针**，之后bind**最多接受9个参数**，**参数数量**必须与**f的参数数量相等**，这些参数被传递给f作为入参。 绑定完成后，**bind**会**返回**一个**函数对象**，它内部**保存了f的拷贝**，具有**operator()**，**返回值类型**被**自动推导**为**f的返回类型**。在发生调用时这个**函数对象**将把之前**存储的参数**转发给**f**完成调用。

### 普通函数

```c++
//假如给定以下函数
int f(int a, int b)
{
    return a + b;
}
 
int g(int a, int b, int c)
{
    return a + b + c;
}
```

可以绑定所有参数，如：

bind(f, 1, 2)等价于f(1, 2); bind(g, 1, 2, 3)等价于g(1, 2, 3);

也可以选择性地绑定参数，如：

bind(f, _1, 5)(x)等价于f(x, 5)，其中_1是一个占位符，表示用第一个参数来替换;

bind(f, _2, _1)(x, y)等价于f(y, x);

bind(g, _1, 9, _1)(x)等价于g(x, 9, x);

bind(g, _3, _3, _3)(x, y, z)等价于g(z, z, z);

————————————————

传入bind函数的参数一般为变量的copy，如：

int i = 5;

bind(f, i, _1);

如果想传入变量的引用，可以使用boost::ref和boost::cref，如：

int i = 5;

bind(f, ref(i), _1);

bind(f, cref(i), _1);

————————————————

> 版权声明：本文为CSDN博主「amoscykl」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
> 原文链接：https://blog.csdn.net/amoscykl/java/article/details/83386081



### 非静态成员函数

假如有：

```c++
 struct A {
  void func(int x, int y) {
   cout << x << "," << y << endl;
  }
 };
 
 A a; 
 A* pa = new A; //指针
 boost::shared_ptr<A> ptr_a(pa);  //智能指针.
```

现在要向像 A::func 这样的非静态成员函数绑定. 
若A::func有n个参数, 则 bind 要有 n+2 个参数: **指向成员函数fun的指针**,  **绑定到this的对象**, n个参数.

```c++
 boost::bind(&A::func, a, 3, 4)();    //输出 3, 4
 boost::bind(&A::func, pa, 3, 4)();   //输出 3, 4
 boost::bind(&A::func, ptr_a, 3, 4)();//输出 3, 4
 //同样可以用 _1 这样的占位符. 如:
 boost::bind(&A::func, _1, 3, 4)(ptr_a);//输出 3, 4
```

可以看出. 不论传递给bind 的第2个参数是 对象、对象指针、还是智能指针， bind函数都能够正常工作（直接使用this指针也可以）。 



---

其它参考

https://blog.csdn.net/adcxf/article/details/3970116

https://blog.csdn.net/weixin_30501857/article/details/94902420



# 权限控制符

private：

私有控制符。这类成员只能被本类中的成员函数和类的友元函数访问。

protected：

受保护控制符。这类成员可以被本类中的成员函数和类的友元函数访问，也可以被派生类的成员函数和类的友元函数访问。

public：

共有控制符。这类成员可以被本类中的成员函数和类的友元函数访问，也可以被类作用域内的其他函数引用。

virtual:

C++通过虚函数实现多态."无论发送消息的对象属于什么类，它们均发送具有同一形式的消息，对消息的处理方式可能随接手消息的对象而变"的处理方式被称为多态性。而虚函数是通过Virtual关键字来限定的。





# 指针



## shared_ptr

要确保用 new 动态分配的内存空间在程序的各条执行路径都能被释放是一件麻烦的事情。C++ 11 模板库的 \<memory\> 头文件中定义的智能指针，即 shared _ptr 模板，就是用来部分解决这个问题的。命名空间为 **std**。

只要将 new 运算符返回的指针 p 交给一个 shared_ptr 对象“托管”，就不必担心在哪里写`delete p`语句——实际上根本不需要编写这条语句，托管 p 的 shared_ptr 对象在消亡时会自动执行`delete p`。而且，该 shared_ptr 对象能像指针 p —样使用，即假设托管 p 的 shared_ptr 对象叫作 ptr，那么 *ptr 就是 p 指向的对象。



通过 shared_ptr 的构造函数，可以让 shared_ptr 对象托管一个 new 运算符返回的指针，写法如下：

```c++
shared_ptr<T> ptr(new T); // T 可以是 int、char、类等各种类型
```

此后，ptr 就可以像 T* 类型的指针一样使用，即 *ptr 就是用 new 动态分配的那个对象。

多个 shared_ptr 对象可以共同托管一个指针 p，当所有曾经托管 p 的 shared_ptr 对象都解除了对其的托管时，就会执行`delete p`。

e.g.

```c++
    #include <iostream>
    #include <memory>
    using namespace std;
    class A
    {
    public:
        int i;
        A(int n):i(n) { };
        ~A() { cout << i << " " << "destructed" << endl; }
    };
    int main()
    {
        shared_ptr<A> sp1(new A(2)); //A(2)由sp1托管，
        shared_ptr<A> sp2(sp1);      //A(2)同时交由sp2托管
        
        shared_ptr<A> sp3;
        sp3 = sp2;  				 //A(2)同时交由sp3托管
        
        cout << sp1->i << "," << sp2->i <<"," << sp3->i << endl;
        
        A * p = sp3.get();			// get返回托管的指针，p 指向 A(2)
        cout << p->i << endl;       //输出 2
        
        sp1.reset(new A(3));    	// reset导致托管新的指针, 此时sp1托管A(3)
        sp2.reset(new A(4));   		// sp2托管A(4)
        
        cout << sp1->i << endl; 	//输出 3
        
        sp3.reset(new A(5));    	// sp3托管A(5),A(2)无人托管，被delete
        cout << "end" << endl;
        return 0;
    }
/*
*  输出：
*  2,2,2
*  2
*  3
*  2 destructed
*  end
*  5 destructed
*  4 destructed
*  3 destructed
*/

```



## this

this 是 C++ 中的一个关键字，也是一个 const 指针，它指向当前对象(所谓当前对象，是指正在使用的对象)，通过它可以访问当前对象的所有成员。

```c++
//this指向Dock::DockManager
//DockManager是Dock命名空间下的一个类
sync_->registerCallback(boost::bind(&DockManager::syncCb, this, _1, _2, _3));
```

- this 只能用在类的内部，通过 this 可以访问类的所有成员，包括 private、protected、public 属性的。注意，this 是一个指针，要用`->`来访问成员变量或成员函数。

- this 虽然用在类的内部，但是只有在对象被创建以后才会给 this 赋值（因此不能在 static 成员函数中使用），并且这个赋值的过程是编译器自动完成的，不需要用户干预，用户也不能显式地给 this 赋值。
- 友元函数没有 **this** 指针，因为友元不是类的成员。只有成员函数才有 **this** 指针。
- this 是 const 指针，它的值是不能被修改的，一切企图修改该指针的操作，如赋值、递增、递减等都是不允许的。



在 C++ 中，每一个对象都能通过 **this** 指针来访问自己的地址。

this 作为隐式形参，本质上是成员函数的局部变量，所以只能用在成员函数的内部，并且只有在通过对象调用成员函数时才给 this 赋值。



# 循环控制

## 范围for (c++11)

参考：https://blog.csdn.net/hailong0715/article/details/54172848

这种语句遍历给定**序列中**个元素并对序列中每一个值执行某种操作

语法：

for(元素类型 元素对象：容器对象)
{
		循环体
}

eg.1

```c++
// 类中定义signal_rcv_成员为一个uchar型容器, 在此resize分配空间
signal_rcv_.resize(ir->data.size());
// 范围 for 初始化容器所有元素为0
for(auto &i: signal_rcv_)
{
    i = 0;
}
```



eg.2

```c++
std::vector<int> vec {1,2,3,4,5,6,7,8,9,10};
	for (auto n :vec)
		std::cout << n;
 
	int arr[10] = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
	for (auto n : arr)
		std::cout << n;
```

我们首先遍历容器的内容，然后给容器内的元素每个都加1，然后再遍历一次容器的内容，示例的输出结果如下：

```cpp
修改前
12345678910

修改后
12345678910
```

可以看到，我们遍历后对容器内元素的加1操作并没有生效，修改后的输出仍然和原来的元素值一样，因此可以看出，这种遍历方法，如果传入的迭代参数类型为非引用时，做的是<span style="color:red">值拷贝</span>，所以修改无效。

如果要遍历容器内的元素的同时又要修改元素的值该怎么做呢，方法就是将遍历的变量声明为引用类型

```c++
	std::vector<int> vec {1,2,3,4,5,6,7,8,9,10};
	cout << "修改前" << endl;
	for (auto& n :vec)
		std::cout << n++;
 
	cout << endl;
	cout << "修改后" << endl;
	for (auto j : vec)
		std::cout << j;
```

输出结果为：

```cpp
修改前
12345678910

修改后
234567891011
```



# goto

C++中使用goto语句可以跳到指定的函数末端，在使用g++编译时，要注意在**goto语句出现之后不允许出现新申明的变量**，所以需要申明变量需要放在所有goto语句之前。（VisutalStudio编译无此问题）。

```c++
#include <iostream>
void Test(int m)
{
	int i = m;
	if (i > 10) goto res;
	int j = i;
res:
	std::cout<<"m > 10"<<std::endl;
}

int _tmain()
{
	Test(4);
	return 0;
}
```


此时使用g++编译报错：

root@ubuntu:/home/Temp# g++ -c temp.cpp
temp.cpp: In function ‘void Test(int)’:
temp.cpp:12:1: error: jump to label ‘res’ [-fpermissive]
 res:
 ^
temp.cpp:7:19: error:   from here [-fpermissive]
  if (i > 10) goto res;
                   ^
temp.cpp:10:6: error:   crosses initialization of ‘int j’
  int j = i;

将Test方法中代码做如下修改即可：

```c++
void Test(int m)
{
	int i = m;
	int j;
	if (i > 10) goto res;
 		j = i;
res:
	std::cout<<"m > 10"<<std::endl;
}
```





# 类型转换



## reinterpret_cast 转换

通过重新解释底层位模式在类型间转换。

### 语法

```c++
reinterpret_cast < 新类型 > (表达式)
```

与 static_cast 不同，但与 const_cast 类似，reinterpret_cast 表达式不会编译成任何 CPU 指令（除非在整数和指针间转换，或在指针表示依赖其类型的不明架构上）。它纯粹是一个编译时指令，指示编译器将 **表达式** 视为如同具有 **新类型** 类型一样处理。



### tips

 reinterpretcast要求转换前后的类型所占用内存大小一致，否则将引发编译时错误。
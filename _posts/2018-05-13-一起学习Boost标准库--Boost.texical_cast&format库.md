---
layout:     post                    	# 使用的布局（不需要改）
title:     Boost.texical_cast&format库               # 标题 
subtitle:  一起学习Boost标准库 	#副标题
date:       2018-05-13              # 时间
author:     iceman                      # 作者
header-img: img/post-bg-universe.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Boost	
    - C++
---

今天接着介绍有关字符串表示相关的两个boost库：

- lexical_cast	将数值转换成字符串
- format      字符串输出格式化

首先，介绍下**lexical_cast** ，闻其名，知其意。类似C中的*atoi* 函数，可以进行字符串与整数/浮点数之间的字面转换

# Boost::lexical_cast库

## 前期准备

lexical_cast库位于boost命名空间下，使用需要引入头文件

```c++
#include <boost/lexical_cast.hpp>
using namespace boost;
```



## 函数声明

lexical_cast使用类似C++标类型操作符的形式进行通用的语法，其声明如下：

```c++
// 1. 标准形式，转换数字和字符串
template <typename Target, typename Source>
inline Target lexical_cast(const Source &arg);

// 2. 转换C字符串
template <typename Target>
inline Target lexical_cast(const char* chars, std::size_t count);
inline Target lexical_cast(const unsigned char* chars, std::size_t count);
inline Target lexical_cast(const signed char* chars, std::size_t count);
inline Target lexical_cast(const wchar_t* chars, std::size_t count);
inline Target lexical_cast(const char16_t* chars, std::size_t count);
inline Target lexical_cast(const char32_t* chars, std::size_t count);
```

- 第一种形式有两个模版参数，*Target* 为需要转换的目标类型，通常是数字类型或者std::string，第二个参数*Source* 则不用写，可以通过函数参数推到出来，调用形式如下

```c++
lexical_cast<int>("123");
lexical_cast<double>("234.123");
lexical_cast<std::string>(567.789);
```

- 第二种形式主要用来处理C字符串，支持多种类型,只接受一个模板参数Target,指明转换后的目标类型，函数参数*chars* 和*count* 则标记了要转换的字符串范围 

```c++
const char* double_str = "123.456";
double d1 = lexical_cast<double>(double_str, strlen(double_str));
```



## 使用样例

通过上文的介绍，您大概已经知道如何使用了，此处，我们就通过一个简单Demo来使用lexical_cast库

```c++
#include <boost/lexical_cast.hpp>
#include <iostream>
#include <string>
#include <iomanip>
using namespace std;
using namespace boost;

int main()
{
	int age = lexical_cast<int>("29");
	int money = lexical_cast<long>("10000000");
	float pai_f = lexical_cast<float>("3.1415926535");
	double pai_d = lexical_cast<double>("3.14159265358979323846264338324990", 20);

	cout << "age: " << age << endl
		<< "money: " << money << endl
		<< setiosflags(ios::fixed)
		<< "pai_f: " << setprecision(8) << pai_f << endl
		<< "pai_d: " << setprecision(16) << pai_d << endl;


	cout << lexical_cast<string>(age) << endl
		<< lexical_cast<string>(money) << endl
		<< lexical_cast<string>(pai_d) << endl;

	return 0;
}
```

Output:

```c++
age: 29
money: 10000000
pai_f: 3.14159274
pai_d: 3.1415926535897931
29
10000000
3.1415926535897931
```

**注意：** 使用lexical_cast时要注意，转换成数字的字符串中只能有数字和小数点，不能出现字母或其他非数字字符，同时也不支持高级的格式控制，如果要进行复杂的格式控制可以使用**std::stringstram** 和 **boost::format** (后面介绍该库)

## 错误处理

当lexcial_cast无法执行转换操作的时候会抛出异常**bad_lexical_cast** ,他是**std::bad_cast** 的派生类，此处以上文中*注意* 来说明

```c++
	try
	{
		int age = lexical_cast<int>("0x64");
		//int age = lexical_cast<int>("100L");
		//bool f = lexical_cast<bool>("false");
		cout << "age: " << age << endl;
	}
	catch (bad_lexical_cast &e)
	{
		cout << "cast error: " << e.what() << endl;
	}
```

Output:

```c++
cast error: bad lexical cast: source type value could not be interpreted as target
```

如果每次都通过异常捕获来处理，就比较麻烦了，还好lexical_cast为我们想到了，再命名空间**boost::conversion** 提供方法*try_lexical_convert()* 函数来避免抛出异常，通过返回bool值来表示是否转换成功。具体使用如下：

```c++
using namespace boost::conversion;
int x = 0;
if (!try_lexical_convert<int>("0x64", x))
	cout << "convert failed" << endl;
else
	cout << "x: " << x << endl;
```

# Boost.format库

C++标准库提供了强大的输入输出流处理，可以通过设置输出各种各样的格式，精度控制、填充、对齐等。但是唯一缺点就是太复杂了，真心记不住这么多，还是怀恋**printf()** 可以通过规定的样式输出想要的格式。虽然C++中可以继续使用**printf()** 但它缺乏类型安全检查等其他缺点，重点就是boost.format库实现了类似于**printf()** 的格式化对象，可以把参数格式化到一个字符串，而且是类型安全的，是一个header-only 的函数库，只要准备好头文件，不用预先编译就可以使用了,最主要的是用着还挺顺手。

## 前期准备

format库位于boost命名空间中，需引入头文件：

```C++
#include <boost/format.hpp>
using namespace boost;
```

## 类声明

```c++
template <class Ch, 
        class Tr = BOOST_IO_STD char_traits<Ch>, class Alloc = std::allocator<Ch> >
    class basic_format;

typedef basic_format<char >     format;

template<class Ch, class Tr, class Alloc>
class basic_format
{
public:
	explicit basic_format(const Ch* str = NULL);
	explicit basic_format(const string_type& s);
	basic_format(const basic_format& x);
	basic_format& operator= (const basic_format& x);

	basic_format& clear();       // 清空缓存
	basic_format& parse(const string_type&); // 重新格式化

	size_type   size() const;    // 获取字符串长度
	string_type str()  const;    // 获取格式化后的字符串

	template<class T>
	basic_format&   operator%(const T& x); //重载操作符%

	template<class Ch2, class Tr2, class Alloc2>
	friend std::basic_ostream<Ch2, Tr2> &
		operator<<(std::basic_ostream<Ch2, Tr2> &,
		const basic_format<Ch2, Tr2, Alloc2>&);	//流输出
}; // class basic_format
```

- str： 返回format对象内部已经格式化号的字符串
- size ： 返回format对象格式化号的字符串长度，可以直接从str()返回的string.size()
- parse ：重新格式化，清空format对象内部缓存，改用一个新的格式化字符串，如果只是想清空缓存，则使用clear()，它把format对象恢复到初始化状态
- operator% ： 可以接受待格式化的任意参数，%输入的参数个数必须等于格式化字符串中要求的个数，过多或者过少都会抛出异常
- operator<<：重载流输入操作符，可以直接输出格式化好的字符串

不过要注意的是，透过operator%传给boost::format对象的变量是会储存在对象内部的，所以可以分批的传入变数；但是如果变量的数量不符合的话，在编译阶段虽然不会出现错误，可是到了执行阶段还是会让程序崩溃，所以在使用上必须小心一点。 不过，在有输出后，是可以再重新传入新的变量、重复使用同一个boost::format 对象的。 

## 格式化语法

format基本继承了printf的格式化语法，格式化选项以%开始，后面是格式规则

``` 
%［标志］［输出最少宽度］［．精度］［长度］类型
```

详细请参考[printf格式输出](http://www.cplusplus.com/reference/cstdio/printf/)

除了支持printf格式化外，还新增了格式：

- %|spec|	：与printf格式选项功能相同，但是两边增加竖线分隔，更好的区分格式化选项与普通字符
- %N%       ：标记第N个参数，相当于占位符，不带任何其他格式化的选项

通过以下使用竖线分隔，更加清楚明了格式化参数

```c++
format fmt("%05d\n%-8.3f\n% 10s\n%05X\n");
format fmt("%|05d|\n%|-8.3f|\n%| 10s|\n%|05X|\n");
```

##  使用样例

```c++
#include <boost/format.hpp>
#include <iostream>
using namespace std;
using namespace boost;

int main()
{
	cout << "-------------"<<endl;
	format fmt("%d + %d = %d");
	fmt % 2 % 3 % 5;
	cout << fmt.str() << endl;

	cout << "-------------"<<endl;
	format fmt2("%05d\n%-8.3f\n% 10s\n%05X\n");
	fmt2 % 123;
    fmt2 % 456.7;
    fmt2 %"boost" % 100;
	cout << fmt2 << endl;

	cout << "-------------"<<endl;
	cout << boost::format("x=%1%,  y=%2% ,z= %3%") % "format" % 40.2 % 134 << endl;
	cout << "-------------"<<endl;
    return 0;
}
```

 Output:

```c++
-------------
2 + 3 = 5
-------------
00123
456.700
     boost
00064

-------------
x=format,  y=40.2 ,z= 134
-------------
```


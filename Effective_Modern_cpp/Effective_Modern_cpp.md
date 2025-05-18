# effective modern c++

# 1.左值右值

C++所有的形参都是左值

左值可以取地址，右值不可以

左值通过复制构造函数构造，右值通过移动构造函数构造

```cpp
class Widget {
public:
 Widget(Widget&& rhs)
 {
   index = rhs.index;
   std::cout << "move construct !" << std::endl;
 }
 Widget()
 {
   index = 0;
 }
 void setIndex(int index)\
 {
   this->index = index;
 }
 void printIndex()
 {
   std::cout << "index: " << index << std::endl;
 }
private:
 int index;
};

void testMoveConstruct()
{
   Widget w1, w2;
   w2.setIndex(2);
   w1.printIndex();
   w2.printIndex();
   Widget w3(std::move(w2));
   w3.printIndex();
}
```

输出结果

```shellsession
index: 0
index: 2
move construct !
index: 2
```

# 2.模版型别推导

万能引用---在模版函数中声明T&&

在非模版函数中如果声明T&&则为右值引用

```cpp
template<typename T>
void f(T&& param)

const int& c=27;
f(c) // 左值引用

f(27) // 右值引用
```

之所以被称为万能引用，是因为该函数接口在传入左值引用的时候会把参数推导为左值引用，而传入右值引用的时候会把参数推导成右值引用。如上代码所示。



## 2.1.param是指针或者引用，但是不是万能引用

```cpp
template<typename T>
void f1(T&)
{
  std::cout << typeid(param).name() << std::endl;
}

int c = 27;
const int cl = 27;
const int& cr = 27;

f1(c);   // int&
f1(cl);  // const int&
f1(cr);  // const int&
```

如果在模板函数形参中加入const修饰

```cpp
template<typename T>
void f1(const T&)

int c = 27;
const int cl = 27;
const int& cr = 27;

f1(c);   // const int&
f1(cl);  // const int&
f1(cr);  // const int&
f1(27);  // 编译不过，原因在于左值引用不兼容右值
```

## 2.2.param是万能指针或引用

```cpp
template<typename T>
void f0(T&&)
{
  std::cout << typeid(param).name() << std::endl;
}

int c = 27;
const int cl = 27;
const int& cr = 27;

f0(c);   // int&
f0(cl);  // const int&
f0(cr);  // const int&
f0(27); // int&&
```

也就是说如果是万能引用，只是增加了对右值的支持，像27这种临时变量可以使用右值引用减少拷贝。

## 2.3.param既不是指针也不是引用

```cpp
template<typename T>
void f2(T param)
{
  std::cout << typeid(param).name() << std::endl;
}  

f2(c);  // int
f2(cl);  // int
f2(cr);  // int
f2(27);  // int
```

因为这是拷贝，拷贝之后就没有那些限制符的限制了。



而如果对于字符串

```cpp
template<typename T>
void f0(T&& param)
{
  std::cout << typeid(param).name() << std::endl;
}

template<typename T>
void f1(T& param)
{
  std::cout << typeid(param).name() << std::endl;
}

template<typename T>
void f2(T param)
{
  std::cout << typeid(param).name() << std::endl;
}   

 const char p[] = "asaasas";
 const char* ptr = p;

 f0(p);  // const char[8]&
 f1(p);  // const char[8]&
 f2(p);  // const char*
 f0(ptr);  //const char* &
 f1(ptr);  // const char* &
 f2(ptr);  // const char*
```

也就是说对于字符数组，带引用的模板函数会将其转成左值引用，而对于指向该字符串的指针，带引用的模板函数会把指针本身作为左值引用。对于传值拷贝的函数，都是直接的const指针。



书中对于函数实参的描述根据我的实测是错误的

## 2.4.完美转发跟万能引用的关系

万能引用虽然能够自动把传进来的实参转成对应左值引用或者右值引用，但是没办法把这些引用直接匹配到对应函数中。这时候就需要使用完美转发来转发参数。完美转发的意思就是你是左值引用就转发到左值引用的函数，你是右值引用就转发到右值引用的函数。

```cpp
void process(int& x) {
    std::cout << "Lvalue reference: " << x << std::endl;
}

void process(int&& x) {
    std::cout << "Rvalue reference: " << x << std::endl;
}

template<typename T>
void wrapper(T&& arg) {
	process(std::forward<T>(arg));  // 完美转发
}

void testForward()
{
    int a = 10;
    wrapper(a);         // a是左值，传入到左值引用的形参函数
    wrapper(20);        // 20是右值，传入到右值引用的形参函数
}
```



# 3.auto型别推导

auto型别推导基本等价于模板型别推导。只是在进行初始化的时候如果使用{}，则会使用列表初始化std::initializer\_list。而std::initializer\_list本身也是一个模板，是需要推导类型的。



当在函数返回值或者输入形参使auto，其使用的是模板型别推导而不是正常的auto型别推导模式



```cpp
 auto x = 28;        // int
 const auto y = x;   // const int
 const auto& z = x;  // const int&

 auto&& uref1 = x;   // int
 auto&& uref2 = y;   // const int&
 auto&& uref3 = 27;  // int&&
```

由于auto在常规状态就是模板型别推导。所以auto&& 就是模板型别推导的万能引用。而“=”右侧的变量就是传进模板函数的参数。型别推导规则完全遵守模板型别推导规则。



当使用{}初始化auto的变量时，

```cpp
auto x = {1,2 3};
```

x变量的型别是std::initializer\_list\<int>,int是通过大括号里面的型别推导出来的。



不能通过编译的例子

```cpp
void auto func()
{
  return {1};
}

void auto func1()
{
return 1;
}
```

在这个例子里面func函数是无法编译通过的，因为返回的列表初始化类型std::initializer\_list\<int>,但是作为函数返回值auto类型推导走的是模板类型推导，它不能识别这个类型。

# 4.decltype

这个函数用于获得变量的类型。在比较新的版本中decltype的作用在弱化，auto的语义在增强而很多场景不再需要decltype的辅助。



```cpp
template <typename Container,typename Index>
auto authAndAccess(Container&& c, Index i)
{
  return std::forward<Container>(c)[i];
}

std::vector<int> v = { 1,2,3,4 };
auto x = authAndAccess(v, 1);
std::cout << x << std::endl;

decltype(x) z = 10;
decltype((x)) y = x;
```

authAndAccess函数输入一个容器和索引，然后根据万能引用返回对应容器索引的引用。返回类型是自动推导出来的。

decltype在输入变量，会推导输入变量类型，如果输入表达式，返回表达式返回值的左值引用。(x)就是一个表达式，所以y的类型为int&。



# 5.查看类型推导的方法

IDE会推断出类型信息，但是现实中可能存在错误。

编写出错误代码让编译器检查，通过报错信息查看，虽然是一种办法，但是基于模板元的代码一个报错就是一大坨。

运行时输出typeid(x).name()

这是标准库的实现，但是对于不同编译器，输出是依赖于具体编译器实现的。而且函数解算过程中会删除一些类型的修饰符



# 6.优先使用auto而非显示型别声明

直接用auto声明变量类型比较省事。因为复杂类型带模板的都很长。这样做也有一些额外好处。



```cpp
    int x = 10;
    auto y;        // 编译不过
    auto z = 10;   // 可以编译过
```

上面的差异可以帮助在声明完变量后检查变量是否初始化，未初始化编译不过去。



auto的另外一个好处是使用auto的lambda表达式的大小小于function的



```cpp
bool big(int x, int y)
{
  return x > y;
}

std::function<bool(int x, int y)> func;
func = big;

auto func2 = [](int x, int y)->bool {
  return x > y;
};

std::cout << sizeof(func) << std::endl;     // return 64
std::cout << sizeof(func2) << std::endl;    // return 1
```

另外有时候对于复杂数据类型，自己手写的类型不一定准确，如果写的是错误的，编译器做类型推断会把做一些拷贝等行为，而程序员可能发现不来导致性能变差而不知道。



```cpp
std::unordered_map<std::string,int> m;

for(const std::pair<std::string, int>& p:m)
{
  ...
}
```

问题在于std::unordered\_map的键是const std::string。所以在进行遍历的时候，每一个p都是引用的拷贝出来的临时变量而非原始容器中的数据。

对应的使用auto就可以规避这种问题

```cpp
for(const auto& p:m)
{
  ...
}
```

# 7.使用auto做自动类型推导存在的坑

```cpp
std::vector<bool> v;
v.push_back(true);
v.push_back(false);

for (const auto& p : v)
{
  std::cout << p << std::endl;
  bool b = p;  // 或者这样auto b = static_cast<bool>(p);

  std::cout << typeid(b).name() << std::endl;
  std::cout << typeid(p).name() << std::endl;
}
```

输出结果

```shellscript
1
bool
class std::_Vb_reference<struct std::_Wrap_alloc<class std::allocator<unsigned int> > >
0
bool
class std::_Vb_reference<struct std::_Wrap_alloc<class std::allocator<unsigned int> > >
```

问题不在于auto怎么了而是在一些库的函数或者类实现的·时候，其索引\[]返回的不是常规的T&，而是代理类。

比如vector\<bool> 的索引值，返回的是std ::vector\<bool>::reference。这是由这个容器内部设计导致的，因为它的实现内部并不是一个一个的bool而是使用一个比特位便是bool值，这种方法在内存上比较紧凑，但是带来的坑就是如果你使用auto直接获得索引值，返回的不是目标的bool值，而是这个代理。

解决方法就是做一个强制类型转换，像上面b那样或则显示转换。auto b = static\_cast\<bool>(p);



只要有这种隐藏的代理类实现，直接使用auto就存在这个风险，只能强制转换。

# 8.创建对象进行初始化时()和{}区别

使用{}进行初始化的好处是可以禁止内建类型进行隐式窄化型别转换。

```cpp
double x=0;
double y = 0;

int sum{ x + y };  //编译不过
// Error C2397 conversion from 'double' to 'int' requires a narrowing conversion
```

使用{}进行初始化的坑点在于当在类构造函数的接口使用了一个列表初始化形参，当使用{}进行类构造时，这种接口会高优先级抢占其他接口，即使其它接口的类型跟输入更接近。

```cpp
class Widget3 {
public:
	Widget3(int i, bool b)
	{
		std::cout << "1 construct" << std::endl;
	}

	Widget3(int i, double d)
	{
		std::cout << "2 construct" << std::endl;
	}

	Widget3(std::initializer_list<std::string> il)
	{
		std::cout << "3 construct" << std::endl;
	}

private:
	int i;
	bool b;
	double d;
};

Widget3 w({ 1,2.0 });
```

上面代码，因为std::initializer\_list的参数是std::string，输入是数字实在类型匹配不上，所以会走第二个构造函数。打印2 construct



如果再加一个std::initializer\_list的参数是可以跟输入的数字进行隐式类型转换的，如下

```cpp
class Widget3 {
public:
	Widget3(int i, bool b)
	{
		std::cout << "1 construct" << std::endl;
	}

	Widget3(int i, double d)
	{
		std::cout << "2 construct" << std::endl;
	}

	Widget3(std::initializer_list<std::string> il)
	{
		std::cout << "3 construct" << std::endl;
	}
	Widget3(std::initializer_list<long double> il)
	{
		std::cout << "4 construct" << std::endl;
	}

private:
	int i;
	bool b;
	double d;
};
Widget3 w({ 1,2.0 });
```

因为1和2.0都可以转换为long double类型，所以打印4 construct



# 9.表示空指针优先使用nullptr，而非0或者NULL

NULL和0是旧版本兼容C的老写法，但是这种写法存在漏洞，因为它们并不是真正的空指针类型，所以这么写在大型软件中如果编程还不规范就容易出问题。如果在模板推导中



```cpp
void func2(void* inPtr) {
  if (inPtr == nullptr)
    std::cout << "nullptr" << std::endl;
  else
    std::cout << "others" << std::endl;	
}

template<typename FuncType,typename MuxType,typename PtrType>
decltype(auto) lockAndCall(FuncType func, MuxType& mutex, PtrType ptr)
{
  MuxGuard g(mutex);
  return func2(ptr);
}


std::mutex m;
lockAndCall(func2, m, nullptr);  // 正常使用
lockAndCall(func2, m, 0);        // 编译不过
lockAndCall(func2, m, NULL);     // 编译不过
func2(NULL);                     // 可以正常使用
```

通过模板推导，直接传入模板函数0或者NULL不能通过编译，但是如果绕过模板，直接用老语法进行类型转换调用func2可以正常使用。这就凸显出模板对于类型检查和转换会更加严格，这对于设计大型软件至关重要。



# 10.优先使用using而非typedef

大多数情况二者没有明显优劣。using可以直接用于模板中，但是typedef不行



```cpp
template<typename T>
using MyList = std::list<T>;

template<typename T>
struct MyList2 {
  typedef std::list<T> type;
};

MyList<Widget> w;    // using的使用方法
MyList2<Widget>::type w2;  // typedef的使用方法
```

using可以直接在模板中做别名，但是typedef不行，它得在模板类的里面做。所以using能稍微省点事。



# 11.优先使用限定作用域的枚举类型而非不限定作用域的枚举类型

这个避免命名空间污染的问题比较显而易见。当然因为不能做隐式类型转换，所以做显式类型转换的时候会写一大串，写法上会多一些。

```cpp
enum class Color {
  black,
  white,
  red
};

std::cout << static_cast<int> (Color::white) << std::endl;
```

不加作用域的枚举类型就是整型。限定作用域枚举类另外一个特征就是可以指定继承类型

```cpp
enum class Color: std::uint8_t
{
  good = 0,
  failing = 1
}
```

这样对于一些枚举空间不大的指定小的数据类型节约内存。



# 12.优先使用删除函数，而非private未定义函数

老的做法是在类的private定义方法，来避免外部调用。但是这样不能避免内部调用，而且只能在类内部实现禁止函数。而delete可以禁止任何函数，包括非类成员。

```cpp
void print(double x);
void print(int x);

void print(double x)
{
  std::cout << "double" << std::endl;
}

void print(int x)
{
  std::cout << "int" << std::endl;
}

void testDelete()
{
  print(10);
  print(10.0);
}
```

输出

```shellscript
int
double
```

而如果把函数禁止掉

```cpp
void print(double x) = delete;

void print(int x);

void print(double x)
{
  std::cout << "double" << std::endl;
}

void print(int x)
{
  std::cout << "int" << std::endl;
}

void testDelete()
{
  print(10);
  print(10.0);
}
```

则会直接编译不通过。



# 13.非模板函数左值引用右值引用跟模板函数左值引用右值引用区别

前面写过模板函数的函数形参带左值引用右值引用的类型推导，其中万能引用如下

```cpp
template<typename T>
void func(T&& param)
```

这时候如果传入左值引用就使用左值引用，如果传入右值引用就用右值引用，而如果想根据函数接口对左值引用右值引用不同做完美转发就用std::forward\<T>(arg)



但是对于非模板函数

```cpp
void func(int& param);  // 第一个函数
 
void func(int&& param); // 第二个函数

int i = 10;

func(i);  // 左值引用调用第一个函数
func(10); // 右值引用调用第二个函数
```

第一个函数是接受左值引用，而第二个函数接受右值引用，而不是万能引用。这点需要注意区分。

# 14.为意在改写的函数添加override声明

override主要是在派生类中重写虚函数，这样一个是看起来语义很清晰，一个是可以避免程序员预期之外的重写或未重写行为。

在重写过程中，编译器会检查函数的型别，修饰符，只有做到完全匹配，才回去重写该方法。



# 15.类返回成员左值引用和右值引用

```cpp
class Widget5 {
public:
  using DataType = std::vector<double>;
  DataType& data()& {
    std::cout << "left" << std::endl;
    std::cout << "this: " << this << std::endl;
    return values;
}

DataType data()&& {
  std::cout << "right" << std::endl;
  std::cout << "this: " << this << std::endl;
  return std::move(values);
}

void init()
{
  this->values = { 1.0,2.0,3.0 };
  std::cout << "init" << std::endl;
  std::cout << "this: " << this << std::endl;
  std::cout << "values " << &(this->values) << std::endl;

  for (const auto& d : values)
  {
    std::cout << d << " " << &d << std::endl;
  }
}

private:
	DataType values;
};

void testClassRef()
{
  Widget5 w;
  w.init();
	
  auto& r1 = w.data();
  for (const auto& d : r1)
  {
    std::cout << d << " " << &d << std::endl;
  }
  std::cout << &r1 << std::endl;

  r1[0] = 2;
  for (const auto& d : r1)
  {
    std::cout << d << " " << &d << std::endl;
  }

  auto r2 = std::move(w).data();
  std::cout << &r2 << std::endl;
  for (const auto& d : r2)
  {
    std::cout << d << " " << &d << std::endl;
  }
}
```

通过重载，获取左值和右值引用。这是左值引用DataType& data()&，这是右值引用DataType data()&&



# 16.优先使用const\_iterator而非iterator

```cpp
std::vector<int> v = { 1,2,3,4 };	
auto it = std::find(std::cbegin(v),std::cend(v),3);
if (it != std::cend(v))
{
  std::cout << *it << std::endl;
}
else
{
  std::cout << "no" << std::endl;
}
```

这个规则比较常规，只是c++11标准支持的不够好，c++14及以后用起来会方便点。



# 17.只要函数不会发射异常，就为其加上noexcept声明

noexcept声明会给函数性能带来优化，但是使用前一定要明确好是都能保证函数不发射异常。



# 18.只要有可能使用constexpr就使用它

constexpr修饰对象表达的是它的值是const的而且在编译期就是已知的。constexpr修饰函数，则表示如果函数的实参是编译期可以知道的，则函数的返回值也是编译期可以获得的。如果函数的实参编译期有一个不能获得，则函数只能在运行期获得该返回值。

能在编译期获得的值，就可以直接把值写进只读存储器，这样程序的性能会得到提升。

```cpp
int zs;

constexpr auto arraySize =zs;    // 不能通过编译因为zs在编译期不能获得它的值

constexpr int pow(int base, int exp) noexcept
{
  ...
}

constexpr auto num = 5;
std::array<int,pow(3,num)> res;  // array的大小需要编译期获得
```

上述代码pow函数被声明成不可抛出异常且函数值如果输入的实参可以编译期获得就可以在编译期输出函数返回值。而res变量构造的实参正好是可以编译期获得的，所以res的代码可以通过编译。



这样可以最大程度保证运行期性能，但是这样做的坏处是计算留在了编译期，会增加编译时间。



# 19.保证const成员函数的线程安全性

类成员函数在声明末尾加const，表示该函数行为不会改变类成员变量。但是如果类成员变量时mutable的，则这样声明的函数还是会改变这种类成员变量。

```cpp
class Polynomial {
public:
  Polynomial() {
    this->index = 10;
    this->value = 20;
  }

  void roots() const
  {
    index = index + 1;
    //value = value + 1;
  }

  void print() const
  {
    std::cout << "index: " << index << std::endl;
    std::cout << "value: " << value << std::endl;
  }

private:
  mutable int index;
  int value;
};

void testConst()
{
  Polynomial p;
  p.print();
  p.roots();
  p.print();
}
```

上述代码中，在声明const的成员函数中修改mutable数据成员index，输出如下

```cpp
index: 10
value: 20
index: 11
value: 20
```

也就是说可以更改。但是如果把 //value = value + 1;的注释解掉，则代码无法通过编译，原因是在const成员函数中更改非mutable数据成员。

所以一个类的成员函数声明了const并不代表其在多线程环境下，就没有数据竞争。如果成员在多线程环境下有数据竞争风险，就要加互斥元或者原子变量等同步机制保护。



# 20.使用std::unique\_ptr管理具备专属所有权的资源

unique\_ptr是专属的智能指针，也就是指针所指对象仅能为一个对象所独占。可以自己指定析构器。

```cpp

enum class InvestmentEnum {
	Stock,
	Bond,
	RealEstate
};

template <typename Ts>
class Investment {
public:
	Investment()
        {
		data = 1;
	}

	Investment(Ts&& params)
	{
		this->init(std::forward<Ts>(params));
	}

	void init(Ts& params) {
		std::cout << "left init" << std::endl;
		data = params;
	}

	void init(Ts&& params) {
		std::cout << "right init" << std::endl;
		data = std::move(params);
	}

	virtual ~Investment()
	{

	}
private:
	Ts data;
};

template <typename Ts>
class Stock :public Investment<Ts> {
public:
	Stock()
	{
		
	}

	Stock(Ts&& params)
	{
		this->init(std::forward<Ts>(params));
	}

	~Stock() {
		std::cout << "destory Stock" << std::endl;
	}
};

template <typename Ts>
class Bond :public Investment<Ts> {
public:
	Bond() 
	{

	}

	Bond(Ts&& params)
	{
		this->init(std::forward<Ts>(params));
	}

	~Bond() {
		std::cout << "destory Bond" << std::endl;
	}
};

template <typename Ts>
class RealEstate :public Investment<Ts> {
public:
	RealEstate() 
	{

	}

	RealEstate(Ts&& params)
	{
		this->init(std::forward<Ts>(params));
	}

	~RealEstate() {
		std::cout << "destory RealEstate" << std::endl;
	}
};

template <typename Ts>
auto makeInvestment(Ts&& params, InvestmentEnum className)
{
	auto delInvmt = [](Investment<Ts>* pInvestment)
		{
			std::cout << "deleta func!" << std::endl;
			delete pInvestment;
		};
	std::unique_ptr<Investment<Ts>, decltype(delInvmt)> pInv(nullptr, delInvmt);

	if (className == InvestmentEnum::Stock)
		pInv.reset(new Stock(std::forward<Ts>(params)));
	else if (className == InvestmentEnum::Bond)
		pInv.reset(new Bond(std::forward<Ts>(params)));
	else if (className == InvestmentEnum::RealEstate)
		pInv.reset(new RealEstate(std::forward<Ts>(params)));

	return pInv;
}

void testUniquePtr()
{
	auto i1 = makeInvestment(1, InvestmentEnum::Stock);
	auto i2 = makeInvestment(2, InvestmentEnum::Bond);
	auto i3 = makeInvestment(3, InvestmentEnum::RealEstate);
}
```

上述代码中Investment 是基类，派生类Stock、Bond 、RealEstate 。然后在工厂函数makeInvestment中创造对象，delInvmt是unique\_ptr指定的析构器，在工厂函数中现声明一个unique\_ptr指针，然后reset成指定的型号。



# 21.使用std::shared\_ptr管理具备共享所有权资源

shared\_ptr智能指针指向的对象可以被多个所有制所有，内部版含引用计数，只有当引用计数为0，所指向的对象自动完成析构。

使用shared\_ptr要避免一个坑就是不要用裸指针初始化多个shared\_ptr,否则智能指针会析构已经析构掉的内存，导致崩溃



```cpp
auto pw = new Widget;

std::shared_ptr<Widget> spw1(pw);
std::shared_ptr<Widget> spw2(pw);
```


上述代码，spw1会对pw完成一次析构，因为spw1的引用计数只有1，但是当spw2也进行析构的时候，它析构的是已经析构的内存，会导致崩溃或行为未定义。



解决方案就是老老实实用工厂函数make\_shared进行智能指针的第一次申请，然后用拷贝构造函数或者拷贝赋值操作符进行拷贝操作。



shared\_ptr有一个坑，就是在已经被shared\_ptr指向的对象内部使用指向该对象的shared\_ptr指针，不能直接使用，都则会行为未定义。正确的使用方法是让类继承自std::enable\_shared\_from\_this



```cpp
class MyClass : public std::enable_shared_from_this<MyClass> {
public:
	void show() {
		std::shared_ptr<MyClass> self = shared_from_this();
		std::cout << "Shared from this: " << self.use_count() << std::endl;
	}
};

std::shared_ptr<MyClass> ptr = std::make_shared<MyClass>();
ptr->show();
std::shared_ptr<MyClass> ptr2 = ptr;
ptr2->show();
```

执行输出

```shellscript
Shared from this: 2
Shared from this: 3
```

在外部ptr通过工厂函数初始化获得对象的指针，此时引用计数是1，在调用show函数的时候，为了获取指向对象本身的智能指针，使用shared\_from\_this函数，此时引用计数是2。


22.对于类似std::shared\_ptr但是有可能空悬的指针使用std::weak\_ptr
=================================================

```cpp
class MyClass2 {
public:
	MyClass2() { std::cout << "MyClass2 Constructor" << std::endl; }
	~MyClass2() { std::cout << "MyClass2 Destructor" << std::endl; }
};

void testWeakPtr()
{
	std::shared_ptr<MyClass2> sharedPtr = std::make_shared<MyClass2>();
	std::weak_ptr<MyClass2> weakPtr(sharedPtr);

	// 或者使用 reset 方法
	// std::weak_ptr<MyClass> weakPtr;
	// weakPtr = sharedPtr; // 直接通过赋值

	std::cout << "Use count of sharedPtr: " << sharedPtr.use_count() << std::endl; // 输出: 1

	if (auto tmpPtr = weakPtr.lock()) { 
		std::cout << "WeakPtr is valid." << std::endl;
		std::cout << "Use count of sharedPtr: " << sharedPtr.use_count() << std::endl; // 输出: 1
		std::cout << "tmpPtr: " << tmpPtr.use_count() << std::endl;
	}
	else {
		std::cout << "WeakPtr is expired." << std::endl;
	}

	sharedPtr.reset();
	std::cout << "SharedPtr reset." << std::endl;

	if (auto tmpPtr = weakPtr.lock()) {
		std::cout << "WeakPtr is valid." << std::endl;
	}
	else {
		std::cout << "WeakPtr is expired." << std::endl; 
	}
}
```

输出

```shellscript
MyClass2 Constructor
Use count of sharedPtr: 1
WeakPtr is valid.
Use count of sharedPtr: 2
tmpPtr: 2
MyClass2 Destructor
SharedPtr reset.
WeakPtr is expired.
```

weak\_ptr通过shared\_ptr初始化，但是调用weak\_ptr的lock方法，如果返回非空指针，tmpPtr 就会多占一个引用计数。把shared\_ptr重置后，弱引用指针因为并不占用引用计数，所以再调用lock方法，返回空指针，表示弱引用指针悬空。



weak\_ptr还可以避免shared\_ptr互相指向，导致内存泄漏。如果两个共享指针形成闭环指向，则他们的引用计数至少永远为1，则即使退出作用域，依然不会销毁内存。



# 23.优先使用std::make\_unique和std::make\_shared,而非直接使用new

直接使用new 初始化智能指针存在潜在的坑，就是在函数的实参这样做的时候，代码如下

```cpp
processWidget(std:shared_ptr<Widget>(new Widget),computePriority());
```

这行代码在编译的时候会分出三个步骤，一个是new对象，一个是把new对象赋值给智能指针，computePriority()函数调用。但是编译出来的优化代码三者顺序并不是唯一固定的。



如果先发生了new然后执行computePriority，这时候出现异常，则第一步创建出来的动态对象会造成内存泄漏。



合理的写法是

```cpp
std:shared_ptr<Widget> pw(new Widget);
processWidget(pw,computePriority());
```

更高效的做法是

```cpp
std:shared_ptr<Widget> pw(new Widget);
processWidget(std::move(pw),computePriority());
```

这样new和赋值给智能指针的中间不会有其他函数造成不必要的异常干扰。或者直接使用工厂函数，获取智能指针。

```cpp
processWidget(std::make_shared<Widget>(),computePriority());
```



这里只是说优先使用工厂函数，但是有些时候使用工厂函数会造成一定性能损失，这时候就要用new方法去构造只能指针。



比如一个比较大的对象

```cpp
auto pBigObj = std::make_shared<ReallyBigType>();

std::make_shared<ReallyBigType> pBigObj(new ReallyBigType);
```

当上述两个对象的shared指针的引用计数都为0，但是weak指针的计数不为零的时候，前者会等到weak指针引用计数为0才会去清理掉这块内存，而后者在shared指针引用计数为0的时候发生析构，就把内存回收了。对于这种体型较大的对象，应该是析构的时候就应该回收内存。所以使用new的写法对于内存的使用会更高效。



另外一种不适合使用make工厂函数的地方是自定义智能指针的析构器。

```cpp
void cusDel(){
  ...
}

std::shared_ptr<Widget> spw(new Widget,cusDel);
processWidget(spw,computePriority());
```

# 24.使用Pimpl习惯用法时，将特殊成员函数的定义放在实现文件中

就是隐藏实现，这样设计接口，调用者可以减少不必要的类内部使用的头文件包含，甚至可以在不做接口改变的情况下实现二进制兼容。



头文件

```cpp
class Widget6 {
public:
	Widget6();
	void print();

private:
	struct Impl;
	std::unique_ptr<Impl> pImpl;
};
```


源文件

```cpp
struct Widget6::Impl
{
	int x;
	std::vector<int> v;

	void print()
	{
		std::cout << "impl" << std::endl;
	}
};

Widget6::Widget6():pImpl(std::make_unique<Impl>()) {

}

void Widget6::print()
{
	std::cout << "Widget6" << std::endl;
	pImpl->print();
}
```

调用者

```cpp
Widget6 w;
w.print();
```

输出

```
Widget6
impl
```



# 25.理解std::move和std::forward

std::move执行的是无条件的强制右值转换。本身不会产生移动操作。而std::forward执行的是有条件地转换，只有实参是右值，才强制做向右值转换。

```cpp
class String
{
public:
	String()
	{
		index = 10;
	}

	String(const String& lhs)
	{
		std::cout << "left" << std::endl;
		this->index = lhs.index;
	}

	String(String&& rhs)noexcept :index(rhs.index)
	{
		std::cout << "right" << std::endl;
	}
private:
	int index;
};

void testMove()
{
	String s;

	String s2(s);
	String s3(std::move(s2));
}
```

s是左值，所以s2调用的是拷贝构造函数，s2本身也是临时变量，用move转成右值。则调用移动构造函数。把s2移动走后，原有的s2将变成未知。



# 26.针对右值实施std::move，针对万能引用实施std::forward

对于右值，本来就是临时变量，那么使用move加上右值引用就可以避免拷贝带来的性能损失。对于万能引用，直接使用完美转发就可以把参数配置到对应左值引用或者右值引用的接口上。也是尽可能避免性能损失。但是应该注意一点的是编译器的返回值优化。



```cpp
Widget makeWidget()
{
  Widget w;
  return w;
}
```

对于上述代码，由于RVO的存在，并不会真的复制对象w。RVO优化的条件是局部变量的型别跟局部变量型别一致且返回的就是局部变量本身。



所以在这种情况下，对于函数返回值，如果满足RVO优化，就不需要遵守本条规则去返回move或者完美转发了。

```cpp
Widget makeWidget()
{
  Widget w;
  return std::move(w);
}
```

上面这段代码就不合理。使用 `std::move` 将 `w` 转换为右值引用，意在启用移动语义。然而，在返回局部变量时，C++ 会自动进行返回值优化（Return Value Optimization, RVO），因此实际上不需要使用 `std::move`。编译器会优化这个返回，避免不必要的额外性能开销。



# 27.避免依万能引用型别进行重载

把万能引用作为重载候选型别的时候，会产生很多超越常规的函数，而且万能引用在进行函数匹配的时候会优先进行匹配，导致很多常规调用代码产生远超预期的行为。

使用完美转发构造函数也是会有很多问题。这些问题很麻烦，最好的办法就是不这么做。



为了应对这种问题的替代方案有舍去重载用不同的函数名跟对不同输入类型，或者按值传递。当然有更复杂的方法，但是坑太多就尽量别用了。



# 28.理解引用折叠

c++不支持引用的引用，所以当引用超过达到两个就要进行引用折叠。引用折叠的原则就是，如果原有的引用中存在左值引用，则变为左值引用，如果两个引用都是右值引用，则为右值引用。



# 29.移动语义的一些坑

常规理解，移动肯定是比拷贝性能更高，毕竟没有了拷贝操作。但是在一些情况下也不完全是这样。这个跟具体底层实现有关。比如如果有的容器内部的数据是通过一个指针进行索引的，那么移动只需要移动指针就行，那么这样移动语义的确是比拷贝高效的。但是比如std::array内部实现就是一块内存，即使是移动也是要一个元素一个元素移动，这样的底层实现是不符合对于移动的基本认识的。再比如std::string对于小字符串并不是申请堆上内存，而是内部自带的一个缓冲区，这种情况下移动并不会比复制性能明显提升。



# 30.完美转发失败的情况

通过完美转发把实参根据左值右值转入到对应函数，可以有效提升性能和接口设计的便利性。但是也存在一些场景，完美转发会失效或者编译不过。比如大括号初始化物(之前提到过的)，0和null空指针(不符合现代c++类型规范)，仅声明了的整型static const成员变量。同名但是不同实参的重载函数以及位域(就是类里面成员是按照位进行分配的)

```cpp
class Header {
public:
    // 定义位域
    std::uint32_t version : 4;      // 版本，占4位
    std::uint32_t IHL : 4;          // 首部长度，占4位
    std::uint32_t DSCP : 6;         // 区分服务代码点，占6位
    std::uint32_t ECN : 2;          // 显示拥塞通知，占2位
    std::uint32_t totalLength : 16; // 总长度，占16位

    // 构造函数初始化位域
    Header(std::uint32_t ver, std::uint32_t ihl, std::uint32_t dscp, std::uint32_t ecn, std::uint32_t length)
        : version(ver), IHL(ihl), DSCP(dscp), ECN(ecn), totalLength(length) {}
    
    // 显示头信息
    void display() const {
        std::cout << "Version: " << version << std::endl;
        std::cout << "IHL: " << IHL << std::endl;
        std::cout << "DSCP: " << DSCP << std::endl;
        std::cout << "ECN: " << ECN << std::endl;
        std::cout << "Total Length: " << totalLength << std::endl;
    }
};

void testBit()
{
	Header header(1, 5, 46, 2, 1500);

	// 显示位域内容
	header.display();

	// 查看 Header 的大小
	std::cout << "Size of Header: " << sizeof(Header) << " bytes" << std::endl;
}
```

上面的代码就是按位分布的变量。书里面的代码是不正确的，需要注意。



# 31.避免默认捕获模式

lambda表达式有两种默认捕获模式，一种是按照值，一种是按照引用。这个都比较常规，只是要注意一下，捕获变量的生命周期跟lambda表达式的生命周期是否匹配，如果lambda表达式还存在，但是捕获的变量生命周期结束，这样在lambda表达式中进行引用计算会出现行为未定义。



# 32.使用初始化捕获将对象移入闭包

```cpp
auto pw = std::make_unique<Widget>();

auto func = [p = std::move(pw)]() {
	p->printIndex();
	};

func();
```

pw对象是是unique指针，如果你想移动进入到lambda表达式，从c++14开始，可以在捕获的方括号内，通过表达式来进行捕获。

等号左侧的p是闭包的内部局部变量，等号右侧的表达式涉及变量时闭包外面的变量，这里通过move，获得智能指针右值引用。



# 33.对auto&&型别的形参使用decltype以std::forward之

C++14加入对lambda表达式的泛型支持。

```cpp
void norm(int& in)
{
	std::cout << "left" << std::endl;
}

void norm(int&& in)
{
	std::cout << "right" << std::endl;
}

void testLambdaGen()
{
	auto f = [](auto&& param) {
		norm(std::forward<decltype(param)>(param));
		};

	f(1);   // 传入右值，参数转发到右值引用接口的函数
	int i = 10;
	f(i);  // 传入左值，参数转发到左值引用的接口函数
}
```

# 34.优先使用lambda式而非std::bind

C++14之后，lambda表达式得到史诗级加强，而bind还得考虑占位符，需要用引用还得调用std::ref，用起来已经没有优势。一概改用lambda表达式完事。



# 35.优先选用基于任务而非基于线程的程序设计

基于线程的程序设计如下

```cpp
int dosSomething();

std::thread t(doSomething);
```

程序就会去开一个线程去执行，不过多线程还需要在后面补充同步之类的东西。



基于任务的程序设计如下

```cpp
int compute(int x) {
	// 模拟一个耗时的计算
	std::this_thread::sleep_for(std::chrono::seconds(2));
	return x * x; // 返回平方值
}

void testAsync()
{
	std::future<int> result = std::async(std::launch::async, compute, 5);

	std::cout << "Doing other work in main thread..." << std::endl;

	std::cout << "The result is: " << result.get() << std::endl; // 阻塞，直到结果准备好
}
```

设置这个参数std::launch::async的目的在于强制使用异步模式。如果使用std::launch::deffered，这表明设置的是使用推迟的方式。也就是说不会立刻去执行，而是等到后面调用相应结果的时候才去执行。



# 36.promise和future

```cpp
void calculateSquare(std::promise<int>&& p, int x) {
	std::this_thread::sleep_for(std::chrono::seconds(2));
	int result = x * x;
	p.set_value(result);
}

void testPromise()
{
	std::promise<int> prom;
	std::future<int> fut = prom.get_future();
	std::thread t(calculateSquare, std::move(prom), 5);
	std::cout << "Calculating square..." << std::endl;
	int result = fut.get(); // 阻塞，直到结果准备好
	std::cout << "The square is: " << result << std::endl;

	t.join(); // 等待线程完成
}
```

std::future\<int> fut用于存储线程函数计算的结果。当fut变量调用get方法，表示主线程是需要得到计算线程的解算结果才能进行下一步操作，此时主线程会阻塞等待计算线程返回结果。



# 37.对并发使用std::atomic,对特种内存使用volatile

atomic是原子变量，他们的常规操作是多线程安全的

```cpp
static std::atomic<int> counter(0);

void increment() {
	for (int i = 0; i < 1000; ++i) {
		++counter; // 原子递增
	}
}

void testAtomic()
{
	const int numThreads = 10; // 线程数量
	std::vector<std::thread> threads;

	// 创建多个线程
	for (int i = 0; i < numThreads; ++i) {
		threads.emplace_back(increment);
	}

	// 等待所有线程完成
	for (auto& th : threads) {
		th.join();
	}

	std::cout << "Final counter value: " << counter << std::endl; // 输出结果
}
```

上面代码多线程使用原子变量，不用加锁，因为他们在底层有硬件保证原子性。



volatile则是对特殊内存的一种声明，为了避免编译器对该内存做没有必要的优化。虽然`volatile`可以确保对变量的读取和写入不被优化，但它并不提供线程安全的保证。对于多线程同步，应使用其他同步机制，如互斥锁或原子变量。
















这本书是我 2021 年买的。那个时候我已经基本具备独立开发和承接软件项目的能力了，所以买了很多不同方向的书籍来拓展视野。

我刚工作时选择编程语言。如果当时选择 Java 做纯后端开发，我可以拿到比较丰厚的薪资待遇。但是我没有选择这个方向，而是把所有的时间和精力都投入到机器人 AI 方向。由于最开始没有经验，为了能在 2017 年左右赶上具身机器人的发展，我选择降低薪酬待遇，从最底层的数据标注和机器人应用做起，换来了大量真实项目落地的经验。机器人领域在那个年代主要使用 C++ 和 Python。而 C++ 是最难的编程语言，能与它较量的也就只有纯 C 和 Rust。我当时的想法是，先把最难的搞定，其他的自然就简单、容易上手了。

C++ 很难，除了需要程序员自己管理内存之外，还有一点就是它支持的编程范式极多，语言特性复杂。高水平的工程师和普通水平的工程师做出来的东西，一个天上、一个地下。我当时已经比较熟悉常规的 C++ 编程了，于是就买了这本讲函数式编程的书。因为很多高性能的 C++ 底层组件都或多或少使用了函数式编程范式，现代项目尤其如此。比如，STL 中很多算法都支持函数式范式；把函数当作数据的思想可以通过 lambda、函数指针、`std::function`、模板回调等方式实现，并且可以避免虚函数的运行时开销。在异步并发编程中，如果一个函数接近纯函数，只依赖输入、不修改全局变量，那么它更适合并行化，能够在提高计算效率的同时避免数据竞态问题。

现在写代码基本都是 AI Coding 了。在操纵 AI 完成任务的过程中，相当一部分经验已经没有太大价值了；仍然有价值的，主要是技术审美、风险把控、架构设计等比较高阶的能力。这本书的作者是 KDE 核心开发者，KDE 是一个非常庞大的 C++ GUI 项目。作者技术水平很高，只不过书里的一部分内容在新的语言版本中已经有了更优雅的实现。

我以前有一段时间非常痴迷计算机代数，这个方向的软件代表目前是 Mathematica。我以前学习天文学的天体物理课程时，就用这个软件做作业。多年后，我重新对底层计算机代数产生兴趣并拾起相关学习。这个软件的核心是进行数学符号计算，里面大量使用了函数式编程，因为这种编程范式与数学中的函数调用非常接近。

# 1.函数式编程
先写一个比较简单的函数式编程例子

```cpp
#include <iostream>
#include <vector>
#include <functional>
#include <ranges>
#include <numeric>

// 1. 高阶函数：接受函数作为参数（模板版本，编译期单态化，零开销）
template <typename Func>
auto apply_twice(Func f, int x) {
    return f(f(x));
}

// 2. 函数组合：把两个函数合成一个新函数
template <typename F, typename G>
auto compose(F f, G g) {
    return [f, g](auto x) { return f(g(x)); };
}

int main() {
    // --- lambda 作为一等公民 ---
    auto square = [](int x) { return x * x; };
    auto increment = [](int x) { return x + 1; };

    std::cout << "apply_twice(square, 3) = "
              << apply_twice(square, 3) << "\n"; // (3^2)^2 = 81

    // --- 函数组合：先 square 再 increment ---
    auto square_then_increment = compose(increment, square);
    std::cout << "compose(increment, square)(4) = "
              << square_then_increment(4) << "\n"; // 4^2 + 1 = 17

    // --- std::function：类型擦除，运行时多态，可放入容器/替换实现 ---
    std::function<int(int)> op = square;
    op = increment; // 运行时可切换，但有间接调用开销
    std::cout << "std::function after reassign: "
              << op(10) << "\n"; // 11

    // --- 惰性管道 (C++20 Ranges)：声明式、无副作用、组合式处理数据 ---
    std::vector<int> nums{1, 2, 3, 4, 5, 6};

    auto pipeline = nums
        | std::views::filter([](int n) { return n % 2 == 0; })
        | std::views::transform([](int n) { return n * n; });

    std::cout << "Pipeline (even numbers squared): ";
    for (int v : pipeline) std::cout << v << " ";
    std::cout << "\n";

    // --- 纯函数 + reduce：无副作用，天然适合并行化 ---
    auto sum = std::accumulate(nums.begin(), nums.end(), 0);
    std::cout << "Sum via accumulate (pure reduce): " << sum << "\n";

    return 0;
}
```
上面的例子使用了好几种方法。`apply_twice` 和 `compose` 直接使用模板，模板的输入类型就是函数。`std::function<int(int)> op` 使用的是 `std::function`，声明之后可以动态修改。`pipeline` 中使用 `|` 作为惰性求值管道。从这个例子可以看出来，函数式编程如果与泛型编程结合，可以把程序设计得非常泛化和灵活。数值求解器、高性能运动控制器以及大量算法的工程化实现，或多或少都有这种设计的影子。比如，人形机器人的全身运动控制系统是高度非线性的，需要设计高频控制器，频率一般在几百赫兹到 1000 Hz。数值求解过程中涉及二次规划、逆运动学以及一些非线性数值求解。在数值求解库的设计上也大量使用了函数式编程，比如通用的数值线性代数库 Eigen 使用惰性求值，CasADi 在进行自动微分时使用符号组合操作。

C++ 中还有一种模式是模板元编程：把可以在编译期计算出来的结果提前计算出来，运行期直接使用，从而提升性能。由于编译期计算所依赖的函数通常具有纯函数的特性，因此模板元编程在某种意义上也是函数式编程的一种。

上面的示例代码中
```cpp
auto pipeline = nums
        | std::views::filter([](int n) { return n % 2 == 0; })
        | std::views::transform([](int n) { return n * n; });
```
filter 接收一个数据集（range）和一个谓词，遍历数据集时对每个元素调用谓词进行筛选判断，谓词本身只处理单个元素。

书里展示了很多 `std::accumulate` 的用法，这个函数可以手动指定数据集和二元操作函数。C++ 标准库已经提供了不少标准算法，比如 `std::remove_if`、`std::copy_if`、`std::transform` 等。
```cpp
#include <iostream>
#include <vector>
#include <numeric>

int main()
{
    std::vector<int> nums{1, 2, 3, 4, 5};

    int sum = std::accumulate(
        nums.begin(),
        nums.end(),
        0,
        [](int acc, int x) {
            return acc + x;
        }
    );

    std::cout << sum << '\n';
}
```
除了使用标准库提供的标准算法之外，也可以根据自己的需求设计函数模板。
```cpp
#include <iostream>
#include <vector>
#include <string>

template<typename Iterator, typename Func>
std::string join_strings(
    Iterator first,
    Iterator last,
    std::string init,
    Func func)
{
    std::string result = init;

    for (; first != last; ++first)
    {
        result = func(result, *first);
    }

    return result;
}

int main()
{
    std::vector<std::string> names{
        "Alice",
        "Bob",
        "Charlie"
    };

    std::string result = join_strings(
        names.begin(),
        names.end(),
        std::string{},
        [](std::string acc, const std::string& name) {
            if (!acc.empty())
                acc += ", ";

            return acc + name;
        }
    );

    std::cout << result << '\n';
}
```
上面的例子通过自定义的模板函数传入 lambda 表达式来拼接字符串，也是一个左折叠的示例。

在函数式编程中，经常会使用递归来表达原本由循环完成的迭代过程。不过在 C++ 中，普通函数递归会产生函数调用开销，并且递归层数很深时可能消耗大量调用栈空间，甚至导致栈溢出。因此函数式的写法并不天然意味着更高的运行效率。部分编译器在满足条件并开启相应优化时，可以对尾递归进行尾调用优化（Tail Call Optimization），把递归调用优化成类似循环的形式，从而减少栈空间消耗，但 C++ 标准并不保证一定进行这种优化，所以工程代码中通常不能依赖它。对于普通运行期迭代，如果直接使用循环更加简单、清晰，一般没有必要为了函数式形式强行改写成递归。

递归在早期 C++ 的编译期计算中更加常见。经典模板元编程主要依靠模板实例化和递归来完成编译期迭代，例如通过模板递归计算阶乘或遍历类型列表。需要注意的是，这种“模板递归”发生在编译阶段，并不是普通函数的运行期递归，因此不存在运行时函数调用栈，也不存在通过尾调用优化减少运行时栈开销的问题；它主要受到模板实例化深度、编译时间和编译器资源消耗等限制。

随着 constexpr 能力不断增强，编译期计算已经不再必须依赖模板递归。特别是从 C++14 开始，constexpr 函数已经允许使用局部变量、修改局部状态以及 for、while 等循环，因此很多过去需要模板递归实现的编译期算法，可以直接写成更接近普通 C++ 的形式。C++17、C++20 以及之后的标准又进一步扩展了常量求值能力和可在编译期使用的标准库设施，使编译期编程越来越接近普通的命令式 C++。因此，现代 C++ 中通常可以根据问题选择模板元编程、constexpr 计算或者普通运行期算法，而不是为了函数式风格刻意使用递归。

上面的例子通过自定义模板函数传入 lambda 表达式实现字符串拼接，本质上也是一个左折叠（left fold）的示例。对于这种累积值本身具有一定复制成本的场景，还需要考虑移动语义。例如在每轮折叠时，如果后续不再需要旧的累积值，可以通过 `std::move` 将累积值传递给二元操作函数，避免不必要的深拷贝。C++ 标准库中的 `std::accumulate` 也经历过类似的演进，从 C++20 开始，其累积过程中会对累积值使用移动语义。因此具体使用标准库算法时，需要关注对应 C++ 标准版本的语义和标准库实现。

书里还讲了一些 API 封装与破坏性操作的问题。使用完美转发时，经过一层泛型 API 封装之后，应尽量保留调用者原始参数的值类别和引用语义，避免封装层无意中引入额外复制，或者把右值变成左值。

```cpp
#include <iostream>
#include <string>
#include <utility>

class older_than
{
public:
    explicit older_than(int age)
        : age_(age)
    {
    }

    template<typename Func, typename T>
    void check_and_print(int age, Func&& func, T&& value) const
    {
        if (age > age_)
        {
            std::forward<Func>(func)(
                std::forward<T>(value)
            );
        }
    }

private:
    int age_;
};

int main()
{
    older_than check{18};

    std::string name = "Alice";

    // 传左值
    check.check_and_print(
        20,
        [](const std::string& s) {
            std::cout << "lvalue: " << s << '\n';
        },
        name
    );

    // 传右值
    check.check_and_print(
        25,
        [](std::string&& s) {
            std::cout << "rvalue: " << s << '\n';
        },
        std::string{"Bob"}
    );
}
```
上面的示例代码把函数和参数全部完美转发了。

lambda 表达式在书里用了不少篇幅介绍。我已经比较熟悉了，所以就不写得太细了。lambda 表达式是一种语法糖，也就是说完全可以不用它来实现对应功能，只是有时候使用 lambda 表达式比较方便。在比较现代的 C++ 库中，它被广泛使用。比如在异步通信库 Asio 中，用 lambda 编写相应的回调非常方便。另一个例子是机器人中间件系统：要实现真正的机器人产品落地，仅仅使用一些开源或商业化的中间件还不够，还需要在其上进行封装。很多公司出于方便会直接使用 ROS 2，但也有一些技术较强的公司会自研中间件系统。比如，常规中间件对高速图传的性能优化往往不够好，因此一个成熟的机器人软件产品体系，通常会在基础中间件之上丰富很多功能。在一些场景下，例如做数据转发时，有时需要将时间、调试信息加入元数据，这时使用 lambda 表达式就非常方便。

比如下面这样
```cpp
register_forwarder(
    [this, topic_name](Message& msg) {
        msg.metadata.timestamp = clock_.now();
        msg.metadata.source = topic_name;
        forward(msg);
    }
);
```

标准库提供了 `std::bind` 用于参数绑定，不过如果不做专门的编译器优化，其性能可能低于 lambda 表达式，因此尽量使用 lambda 即可。二者在函数调用上具有相近的作用，但并不完全等价。

## 1.1 柯里化
柯里化（currying）指的是把一个接受多个参数的函数，转换成一系列只接受单个参数的函数链，每次调用只消费一个参数，返回一个新函数去接收下一个参数——比如 f(a, b, c) 变成 f(a)(b)(c)。这样做的目的是把一个通用的多参数函数逐步配置成更具体、更容易复用和组合的函数，形成一个 pipeline。

```cpp
#include <iostream>
#include <vector>
#include <ranges>

int main()
{
    // 柯里化：先接收 threshold，返回一个 predicate
    auto greater_than = [](int threshold) {
        return [threshold](int value) {
            return value > threshold;
        };
    };

    // 从通用函数生成两个专用函数
    auto greater_than_10 = greater_than(10);
    auto greater_than_100 = greater_than(100);

    std::cout << greater_than_10(20) << '\n';   // true
    std::cout << greater_than_100(20) << '\n';  // false

    std::vector<int> nums{3, 12, 7, 25, 100, 200};

    // 直接把柯里化生成的 predicate 交给 filter
    auto result =
        nums | std::views::filter(greater_than(10));

    for (int x : result)
        std::cout << x << ' ';
}
```

## 1.2 避免可变状态
可变性带来的问题，主要体现在并发访问共享变量时可能造成数据不一致。Rust 的一个设计初衷也是解决这类问题，因此设计了所有权机制，并且默认变量是不可变的。C++ 没有 Rust 那样在编译期施加的诸多语法限制，如果不针对这类问题进行专门设计，就非常容易出现多线程 bug。

## 1.3 惰性求值
惰性求值的思想在项目中有很多应用。比如，对于一些游戏 CG 项目，某些资源的启动很耗时，如果游戏一启动就全部加载，就需要等待很长时间。再比如加载并显示大数据量的数据时，页面实际只能显示很少的数据；如果一次性把所有数据加载到内存，软件可能会卡顿很久。用户想看哪些数据，就加载哪些数据，每次只加载几 MB，就可以让用户获得流畅的使用体验。C++ 中的惰性求值也与此类似：一个计算过程可能比较耗时，但结果并不是立即需要，这时可以等到真正需要时再计算。

## 1.4 range
Range 已经在 C++20 中进入标准库，也是 C++20 中新增的四大核心特性之一。它是对迭代器的更高层次抽象。

```cpp
#include <iostream>
#include <vector>
#include <ranges>

int main()
{
    std::vector<int> nums{1, 2, 3, 4, 5, 6};

    auto result = nums
        | std::views::filter([](int x) {
            return x % 2 == 0;
        })
        | std::views::transform([](int x) {
            return x * x;
        });

    for (int x : result)
        std::cout << x << ' ';
}
```
上述代码中的高阶函数是 `filter` 和 `transform`，它们接收 lambda；谓词是 `filter` 中决定元素是否保留的 lambda；映射是 `transform` 把 `x` 映射为 `x * x`；函数组合或 pipeline 是通过 `|` 将操作组合；惰性求值是指构造 `result` 时主要构造计算规则，遍历时才按需计算。

Range 可以分为有界 Range 和无界 Range。原理很简单：像 `vector` 这样的容器是有界的，使用 range-based for 进行遍历时处理的是有界 Range。对于数据处理管线或网络数据流，无法先生成全部数据再进行后续计算，实际上就是一种流式计算。这时可以先生成一个生成器，再由后续代码根据实际需求生成并处理相应的数据。

```cpp
#include <iostream>
#include <ranges>

int main()
{
    auto result =
        std::views::iota(1)
        | std::views::filter([](int x) {
            return x % 2 == 0;
        })
        | std::views::transform([](int x) {
            return x * x;
        })
        | std::views::take(5);

    for (int x : result)
        std::cout << x << ' ';
}
```
上面这个例子中，从自然数开始 → 找偶数 → 平方 → 只取前 5 个。无界 Range 描述一个潜在无限的数据流，通过惰性求值，只计算调用者真正需要的部分。Python 中早就有这些东西了。

书里讲了一种数据结构，称为位图向量树。比如一个 map，如果只修改一处却把整个数据结构复制一遍，就会造成大量资源浪费。对应的解决方案是只复制发生修改的部分，而原来大量未变动的公共数据则直接共享。

## 1.5 代数数据类型

| 类型 | 数学含义 | C++ |
|------|----------|-----|
| **Product Type** | A × B（乘法） | `struct`、`std::tuple` |
| **Sum Type** | A + B（加法） | `std::variant`、`enum`（部分） |

`std::variant` 的一个简单示例
```cpp
#include <iostream>
#include <string>
#include <variant>
#include <type_traits>

int main() {
    std::variant<int, double, std::string> v;

    v = 42;
    std::cout << std::get<int>(v) << '\n';

    v = 3.14;
    std::cout << std::get<double>(v) << '\n';

    v = std::string("hello");
    std::cout << std::get<std::string>(v) << '\n';
}
```
可以把 `std::variant` 理解为类型安全的 union。在上面的例子中，变量 `v` 被限定为只能是 `int`、`double`、`string` 三种类型之一。这样设计的好处是，例如在开发 JSON 库时，可以把待解析的对象限定为指定的类型集合；如果出现新的类型，就可以直接报告“不支持该类型”。

下面的例子会更贴近实战：用结构体定义事件，在事件处理的 handler 中使用 `std::visit` 访问具体的数据，并使用 `std::is_same_v` 做类型判断。这种现代 C++ 类型系统对于设计和实现大型软件非常有帮助，可以保证系统的准确性，并提前发现误用或错误。
```cpp
#include <iostream>
#include <string>
#include <variant>

struct Login {
    std::string username;
};

struct Logout {
    int userId;
};

struct Message {
    int userId;
    std::string text;
};

using Event = std::variant<Login, Logout, Message>;

void handle(const Event& event) {
    std::visit([](const auto& e) {
        using T = std::decay_t<decltype(e)>;

        if constexpr (std::is_same_v<T, Login>) {
            std::cout << "login: "
                      << e.username << '\n';
        }
        else if constexpr (std::is_same_v<T, Logout>) {
            std::cout << "logout: "
                      << e.userId << '\n';
        }
        else if constexpr (std::is_same_v<T, Message>) {
            std::cout << "message from "
                      << e.userId
                      << ": "
                      << e.text
                      << '\n';
        }

    }, event);
}

int main() {
    Event e1 = Login{"Alice"};
    Event e2 = Message{1001, "Hello"};
    Event e3 = Logout{1001};

    handle(e1);
    handle(e2);
    handle(e3);
}
```

`std::optional` 用来表示函数可能返回一个值，也可能返回空值。

```cpp
#include <iostream>
#include <optional>

std::optional<int> divide(int a, int b)
{
    if (b == 0)
        return std::nullopt;

    return a / b;
}

int main()
{
    auto r1 = divide(10, 2);
    auto r2 = divide(10, 0);

    if (r1)
        std::cout << "r1 = " << *r1 << '\n';

    if (r2)
        std::cout << "r2 = " << *r2 << '\n';
    else
        std::cout << "divide failed\n";
}
```

## 1.6 monad
monad 的概念来自范畴论，中文可以叫做单子，范畴论为函数式编程中的 Functor、Applicative、Monad 等抽象提供了一套统一的数学描述。工程上可以简单理解为：把一个值放进某种计算上下文中，并提供一套组合操作，使后续计算可以连续进行，同时由这个抽象统一处理上下文。

```
optional<T>   → 上下文是”可能没有值”
expected<T,E> → 上下文是”可能失败”
future<T>     → 上下文是”结果未来才产生”
```

```cpp
#include <iostream>
#include <optional>
#include <string>

std::optional<int> get_user_id()
{
    return 1001;
}

std::optional<std::string> get_user_name(int id)
{
    if (id == 1001)
        return "Alice";

    return std::nullopt;
}

std::optional<std::string> normalize_name(std::string name)
{
    if (name.empty())
        return std::nullopt;

    return name + "_normalized";
}

int main()
{
    auto result =
        get_user_id()
            .and_then(get_user_name)
            .and_then(normalize_name);

    std::cout << result.value_or("failed") << '\n';
}
```
上面的代码展示了一个非常常见的流程处理 pipeline。正常来说，每一步都有可能失败，因此每执行一个操作都要判断结果，并进行相应的错误处理。如果前面的步骤出错，后面的步骤就不再执行，并返回相应的错误，而不是继续处理。上面的代码可以很优雅地实现这一功能：将错误传播的控制流从业务逻辑的每一步中抽离出来，交给类型本身处理。类似的设计思路也可以用于连接数据库，或者实现异步操作的 future。

# 2.模板元编程
模板元编程是用来实现非常基础通用的组件库的一种编程范式，由于涉及编译期计算和代码行为抽象，一般的业务开发基本不需要自己写模板元编程。写这种代码的一般都是开发基础设施比如语言的标准库，boost 库这种或者英伟达 CUDA 的各种计算包。这块对 C++ 开发能力要求比较高。而且编译期计算的很多类和函数是在编译期就从模板展开了，模板元编程更像是编写源代码的源代码也就是元的意思。

```cpp
#include <iostream>
#include <string>
#include <tuple>
#include <utility>

struct Person
{
    std::string name;
    int age;
    double salary;
};

// ----------------------
// 一个字段更新操作
// ----------------------

template<typename Member, typename Value>
struct Set
{
    Member member;
    Value value;

    template<typename Object>
    void apply(Object& obj) const
    {
        obj.*member = value;
    }
};

// DSL 构造函数
template<typename Member, typename Value>
auto set(Member member, Value&& value)
{
    return Set<Member, std::decay_t<Value>>{
        member,
        std::forward<Value>(value)
    };
}

// ----------------------
// Transaction
// ----------------------

template<typename... Operations>
struct Transaction
{
    std::tuple<Operations...> operations;

    template<typename Object>
    void apply(Object& obj) const
    {
        std::apply(
            [&](const auto&... op)
            {
                (op.apply(obj), ...);
            },
            operations
        );
    }
};

// 创建 transaction
template<typename... Operations>
auto transaction(Operations... operations)
{
    return Transaction<Operations...>{
        std::tuple{operations...}
    };
}

int main()
{
    Person p{
        "Alice",
        20,
        5000.0
    };

    auto update =
        transaction(
            set(&Person::name, std::string{"Bob"}),
            set(&Person::age, 30),
            set(&Person::salary, 10000.0)
        );

    update.apply(p);

    std::cout
        << p.name << '\n'
        << p.age << '\n'
        << p.salary << '\n';
}
```
上面的例子体现了编译期多态和模板泛型编程。Person 是一个非常常规的数据类型，Set 类型定义基本的成员和值，通过 `set` 函数对外提供 API。类似地，Transaction 类型中存储操作元组，再通过工具函数 `transaction` 把多个操作转换到 Transaction 中。

DSL 在项目中非常常见，比如定义消息经常使用 protobuf，如果追求极致效率可以使用 flatbuffer，C++ 项目构建中使用 CMake 的 DSL，数据库查询的时候使用 SQL 的 DSL，机器人进行符号计算使用 CasADi 的 DSL 等等。

# 3.并发系统的函数式编程
在并发或分布式系统中，有一种模式叫作 Actor。每个 Actor 独立管理自己的状态，Actor 之间不直接共享状态，而是通过异步消息通信。其实，ROS 2 整体的设计和使用方式与此非常相似，其中的 Node 节点类似于 Actor，而且这种设计非常适合跨机器构建分布式系统。只不过，Actor 模式严格限制消息处理顺序；ROS 2 节点可以在单个节点内使用并发多线程。不过在实际机器人系统中，对于接收到的消息和数据通常也会按顺序处理，因为真机系统使用历史数据通常没有意义，使用的都是最新数据。只是在一些额外场景（例如数据采集）中，可能会存在数据延迟到达的情况，这时需要在逻辑侧进行时间对齐和数据落盘。书里介绍了作者设计的一些机制，我觉得 ROS 2 基本都提供了类似的机制。感兴趣的话可以看看 ROS 2，这里就不详细介绍了。

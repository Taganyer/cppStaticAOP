[English version](README.md)
# cppStaticAOP

**一个简单、易用的C++静态AOP组件**

* **增强**：cppStaticAOP 是一简单的 C++ 切面模板，你可以通过使用它来增强某一个函数或类的功能。
* **无侵入**：组件是无侵入式的，原有业务逻辑代码不需要作任何改动，就能够轻松使用，从根本上解耦合，避免横切逻辑代码重复。
* **静态编译**：切面模板将会在编译期将切面代码嵌入主业务代码中，运行时不会有太多的性能损失。
* **简单易用**：组件使用简单，使用方法多样，同时支持多种构造方式，不会对原有对象功能带来任何影响。

## 使用环境要求：
* C++17及以上
* gcc / clang

## 使用示例：

```c++
#include <iostream>
#include "AOP_src/AOP.hpp"

int main() {
#ifdef AOP_WILL_USE_SOURCE_LOCATION /// 用于获得调用函数名称，可以不使用它（删除位于 AOP.hpp 的该宏）
    using namespace std;
    using namespace Base;
    class A {
    public:
        A(int a) : a(a) {};

        static int static_fun() {
            AOP_FUN_MARK /// 这里是用于获得函数名称，你可以选择不使用。
            cout << CURRENT_FUN_LOCATION.function() << endl;
            return 0;
        }

        int fun(int &) {
            AOP_FUN_MARK
            cout << CURRENT_FUN_LOCATION.function() << endl;
            return 1;
        };

        int fun(int &) const {
            AOP_FUN_MARK
            cout << CURRENT_FUN_LOCATION.function() << endl;
            return 2;
        };

        void raise_error() const {
            throw runtime_error("A::raise_error()");
        };

    private:
        int a = 0;
    };

    /// 你可以有选择的定义以下函数。
    struct A1 {
        void before() {
            cout << "A1 before the: " << Base::AOPthreadLoc.function() << endl;
        };

        void after() {
            cout << "A1 after the: " << Base::AOPthreadLoc.function() << endl;
        };

        void before() const {
            cout << "A1 const before the: " << Base::AOPthreadLoc.function() << endl;
        };

        void after() const {
            cout << "A1 const after the: " << Base::AOPthreadLoc.function() << endl;
        };

        /// std::exception_ptr & 不起作用。
        void error(const std::exception_ptr &) {
            cerr << "A1 error in the: " << Base::AOPthreadLoc.function() << endl;
        };

        void error(const std::exception_ptr &) const {
            cerr << "const A1 error in the: " << Base::AOPthreadLoc.function() << endl;
        };
        
        void destroy() {
            cout << "destroy fun" << endl;
        };
    };

    AOP aop { A1(), A1() };
    static_assert(is_same_v<decltype(aop), AOP<A1, A1>>);
    auto fun = [] {
        AOP_FUN_MARK
        cout << "lambda fun" << endl;
        return 0;
    };
    int o_O = aop.invoke(fun);
    cout << endl;

    A a(1);
    AOP_Wrapper<A, A1, const A1> wrapper{a};
    const AOP_Wrapper<A, A1, const A1> &const_wrapper = wrapper;
    /// 你可以通过两种方式调用 A 的成员函数。
    /// WARN: C++17 无法自动推断同名函数的成员函数指针，需手动指定。
    int (A::*fun_non_const)(int &) = &A::fun;
    int (A::*fun_const)(int &) const = &A::fun;
    int t = 0;
    t = wrapper.invoke(fun_non_const, t);
    cout << endl;
    t = wrapper.invoke(fun_const, t);
    cout << endl;
    t = AOP_Wrapper_Agent(wrapper, fun, t);
    cout << endl;
    t = AOP_Wrapper_Agent(const_wrapper, fun, t);
    cout << endl;

    AOP<A1, const A1> aop1;
    AOP_Object object {aop1, a}; /// 注意第一个参数必须是 AOP 的左值或右值（与 AOP_Wrapper 不同），其余参数用于构造 A。
    static_assert(is_same_v<decltype(object), AOP_Object<A, A1, const A1>>);
    const AOP_Object<A, A1, const A1> &const_object = object;
    t = object.invoke(fun_non_const, t);
    cout << endl;
    t = object.invoke(fun_const, t);
    cout << endl;
    t = AOP_Object_Agent(object, fun, t);
    cout << endl;
    t = AOP_Object_Agent(const_object, fun, t);
    cout << endl;

    try {
        AOP_Object_Agent(object, raise_error);
    } catch (runtime_error &) {
        cerr << "error test" << endl;
    }
#endif
    return 0;
}
```
### 可能的运行结果
```text
A1 before the: unknown
A1 before the: unknown
lambda fun
A1 after the: main()::<lambda()>
A1 after the: main()::<lambda()>

A1 before the: unknown
A1 const before the: unknown
int main()::A::fun(int&)
A1 const after the: int main()::A::fun(int&)
A1 after the: int main()::A::fun(int&)

A1 before the: unknown
A1 const before the: unknown
int main()::A::fun(int&) const
A1 const after the: int main()::A::fun(int&) const
A1 after the: int main()::A::fun(int&) const

A1 before the: unknown
A1 const before the: unknown
int main()::A::fun(int&)
A1 const after the: int main()::A::fun(int&)
A1 after the: int main()::A::fun(int&)

A1 const before the: unknown
A1 const before the: unknown
int main()::A::fun(int&) const
A1 const after the: int main()::A::fun(int&) const
A1 const after the: int main()::A::fun(int&) const

A1 before the: unknown
A1 const before the: unknown
int main()::A::fun(int&)
A1 const after the: int main()::A::fun(int&)
A1 after the: int main()::A::fun(int&)

A1 before the: unknown
A1 const before the: unknown
int main()::A::fun(int&) const
A1 const after the: int main()::A::fun(int&) const
A1 after the: int main()::A::fun(int&) const

A1 before the: unknown
A1 const before the: unknown
int main()::A::fun(int&)
A1 const after the: int main()::A::fun(int&)
A1 after the: int main()::A::fun(int&)

A1 const before the: unknown
A1 const before the: unknown
int main()::A::fun(int&) const
A1 const after the: int main()::A::fun(int&) const
A1 const after the: int main()::A::fun(int&) const

A1 before the: unknown
A1 const before the: unknown
destroy fun
const A1 error in the: unknown
A1 error in the: unknown
error test
```
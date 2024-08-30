[中文版](README_CN.md)
# cppStaticAOP

**A simple, easy-to-use C++ static AOP component.**

* **Enhancement**: cppStaticAOP is a simple C++ aspect template that allows you to enhance the functionality of a particular function or class.
* **Non-intrusive**: The component is non-intrusive, meaning that the original business logic code does not need to be modified to use it easily, fundamentally decoupling it and avoiding repeated cross-cutting logic code.
* **Static compilation**: The aspect template will embed aspect code into the main business code at compile time, which will not incur significant performance loss at runtime.
* **Simple and easy to use**: The component is easy to use with multiple usage methods and supports various construction methods without affecting the original object's functionality.

## Environment Requirements:
* C++17 or above
* gcc / clang

## Usage Example:

```c++
#include <iostream>
#include "AOP_src/AOP.hpp"

int main() {
#ifdef AOP_WILL_USE_SOURCE_LOCATION /// Used to obtain the calling function name; you may choose not to use it (remove the macro from AOP.hpp)
    using namespace std;
    using namespace Base;
    class A {
    public:
        A(int a) : a(a) {};

        static int static_fun() {
            AOP_FUN_MARK /// Here is used to obtain the function name; you can choose not to use it.
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

    /// You can optionally define the following functions.
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

        /// std::exception_ptr & does not work.
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
    /// You can call member functions of A in two ways.
    /// WARN: C++17 cannot automatically deduce member function pointers for overloaded functions; manual specification is required.
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
    AOP_Object object {aop1, a}; /// Note that the first parameter must be an lvalue or rvalue of AOP (unlike AOP_Wrapper), and the remaining parameters are used to construct A.
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
### Possible Output

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
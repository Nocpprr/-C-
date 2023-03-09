C++ 11 新特性

1. std::move 
 - 引用折叠规则：
  间接的创建引用的引用，会触发引用的折叠，
  ```
  T& & -> T&
  T& && -> T&
  T&& & -> T&
  T&& && -> T&&
  ```
 - 右值引用的特殊推断
  当将一个左值传递给右值引用的函数，且此右值引用指向模板类型参数(T&&)时，编译器推断模板参数类型为实参的左值引用
  ```
  template<typename T>
  void f(T&&);
  
  int i = 10;
  f(i); -> f(int&) -> f(int& &&) -> f(int&)
  ```
  - std::move实现：
  ```
  template<typename T>
  constexpr typename std::remove_reference<T>::type&& move(T&& t){
      return static_cast<typename std::remove_reference<T>::type&&>(t);
  }
  ```
  - std::remove_reference
  ```
  template<typename T>
  struct remove_reference{
     typename T type;
  }

  template<typename T>
  struct remove_reference<T&>{
     typename T type;
  }

  template<typename T>
  struct remove_reference<T&&>{
     typename T type;
  }
  ```
  std::remove_reference的作用就是解引用，move函数的参数T&&是一个指向模板类型参数的右值引用，通过引用折叠，此参数可以和任何类型的实参匹配，因此move既可以传递一个左值，也可以传递一个右值。
  如果传递一个左值，那么根据右值引用模板参数推断和引用折叠，参数类型会转化为左值引用，再通过 static_cast转换成右值引用。如果传进来的本来就是一个右值，那么就什么都不做。
 
2. std::forward
  - forward实现了参数在传递过程中保持其值属性的功能，即若是左值，则传递之后仍然是左值，若是右值，则传递之后仍然是右值。
  ```
  template<typename T>
  constexpr T&& forward(typename std::remove_reference<T>::type& t)noexcept
  {
     return static_cast<T&&>(t);
  }

  template<typename T>
  constexpr T&& forward(typename std::remove_reference<T>::type&& t)noexcept
  {
     return static_cast<T&&>(t);
  }
  ```
 3. 关于std::move和std::forward的一些使用中的问题
  ```
  class A {
  private:  
  public:
    A() = default;
    A(const A& a) = default;
    A(A&& a) = default;
  }
  
  // 一个以右值引用为参数的模版定义的函数
  template<typename T>
  A* Create(T&& t)
  {
    return new A(t);
  }
  ```
  调用这个函数:
  ```
  A* a = Create(std::move(A()));
  ```
  看似会调用移动构造函数，实际上会调用拷贝构造函数，因为在Create函数中，t是一个变量，无论左值引用类型的变量还是右值引用类型的变量，都是左值，因为它们有名字。也就是说，在函数内部，右值引用被转换为了左值。因此传给构造函数的是一个左值。
  但在移动构造函数中，虽然参数a也是一个左值，但是已经成功调用了移动构造函数，不会有影响，但是在经过模板参数转发后，右值变成了左值，因此需要使用完美转发std::forward。

# -C-
C++做工程项目的一些心得


1.	避免使用成对的操作, 例如 malloc/free, new/delete 等, 因为可能因为一些行为使得成对操作的另一半无法执行，例如 跳出循环/中途退出程序等。我们应该让操作发生在对象的构造函数中，相反操作发生在对象的析构函数中，即”Resource acquisition in initialization”, 也叫 RALL。

2.  最好通过const引用传递函数参数(例如:const Address & Address)。

3.  除了那些需要改变的，尽量把变量设置为 const。

4.  成员函数都应被设置为 const，除非它需要修改对象的值。

5.  避免全局变量，使每个变量尽可能小的作用域。

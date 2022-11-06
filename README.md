# -C-
C++做工程项目的一些心得


1.	避免使用成对的操作, 例如 malloc/free, new/delete 等, 因为可能因为一些行为使得成对操作的另一半无法执行，例如 跳出循环/中途退出程序等。我们应该让操作发生在对象的构造函数中，相反操作发生在对象的析构函数中，即”Resource acquisition in initialization”, 也叫 RALL。

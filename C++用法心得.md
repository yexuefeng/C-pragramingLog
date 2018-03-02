#### Date: Tue 23 May 2017
1.对一个const对象调用non-const对象是不允许的，因为non-const成员函数可能会修改const对象，因此编译器做了一个安全假定，不允许const调用non-const成员函数:

>   例如：
>   
>    void Socket::sock_bind(const InetAddr & address)
>       {
>           struct sockaddr_in peeraddr;
>           memset(&peeraddr, 0, sizeof(peeraddr));
>           peeraddr = address.addr_get();
>           ......
>       }
>
>    如果addr_get()成员函数采用如下声明:
>    class InetAddr
>    {
>        ......
>        struct sockaddr_in addr_get(); // error usage
>        ......
>    }
>    则在编译时出现如下错误: error: passing 'const InetAddr' as 'this' argument
>    of 'sockaddr_in InetAddr::addr_get()' discards qualifiers
>    解决上述问题需要将addr_get()成员函数声明为常量成员，如下所示：
>    class InetAddr
>    {
>        ......
>        struct sockaddr_in addr_get() const; //right usage
>        ......
>    }

2.在使用accept函数时有两种用法：

>      struct sockaddr_in addr;
>      socklen_t addrlen = sizeof(addr);
>      int sockfd = accept(sfd, (struct sockaddr *)&addr, &addrlen);
>      或者
>      int sockfd = accept(sfd, NULL, NULL);
>      在采用第一种用法，一定要注意addrlen参数初始化，不可以按如下方式
>      使用：
>      struct sockaddr_in addr;
>      socklen_t addrlen;
>      int sockfd = accept(sfd, (struct sockaddr *)&addr, &addrlen);

#### Date: Wed 24 May 2017
今天效率较低没有写代码，没有什么收获


#### Date: Fri 09 Jun 2017
1.关于指针和引用的区别从编译器角度可以做如下理解:
  程序在编译的过程中分别将指针和引用添加到符号表上，符号表上记录的是变量名
  和变量对应的地址。指针变量在符号表上的地址为指针变量的地址，而引用在符号表
  上的地址对应的地址为引用对象的地址。符号表生成后不会发生改变，只能改变变量
  的值，因此指针可以改变其指向的对像，即指针变量的值，而引用对象则不能修改。

#### 2018年 02月 27日 星期二 11:45:03 CST
1.当算术表达式中既有无符号数又有int值时，int值会被转换为无符号数;
2.顶层const(top-level const)表示指针本身是一个常量，底层const(low-level const)表示指针所指的对象是一个常量; 在使用auto关键字，利用编译器推断类型时，一般会忽略顶层const, 但是会保留底层const。唯一的例外是引用类型，对应形如auto &i = ci, 由于引用类型天然具有顶层const，因而此时会保留顶层const。另外需要注意的是当使用引用用作初始值时，真正参与初始化的其实是引用对象的值，此时编译器以引用对象的类型作为auto的类型;

#### 2018年 03月 01日 星期四
1.关于getline函数，这个函数在C++和C中均有定义，但是这个函数在两者用法存在差异。
*   在C++中:
C++中有两个getline函数，一个是在string头文件中，定义的是全局的函数，另一个是istream的成员函数. 下面分别介绍:
> 1. 作为全局函数时  
>   函数声明是istream& getline(istream& is, string& str, char delim)与istream& getline(istream& is, string& str);
>   该函数从给定的输入流中读取数据，默认情况下直到遇到换行符为止(注意换行符也被读进来了), 然后把所读的内容存入到那个string对象中去(注意不存在换行符). getline只要一遇到换行符就结束读取操作并返回结果，哪怕输入一开始就是换行符。如果输入真的一开始是换行符, 那么所得的结果是个空string. 
> 2.作为成员函数时
>   函数声明是`istream& getline(char *s, streamsize n)`与`istream& getline(char *s, streamsize n, char delim)`;
>   该函数从给定的输入流中读取不超过n字节的数据，默认情况下直到遇到换行符为止, 然后把所读的内容存入到C风格字符串中;注意该函数会在存入的字符数组末尾自动添加上'\0'的结束字符(A null character('\0') is automatically appended to the written sequence if n is greater than zero, even if an empty string is extracted.)
*   在C语言中:
>   getline函数定义在头文件stdio.h中，该函数的声明如下所示:`ssize_t getline(char **lineptr, size_t *n, FILE *stream)`
>   getline() reads an entire line from stream, storing the address of the buffer containing the text into `*lineptr`.  The buffer is null-terminated and includes the newline character, if one was found. If `*lineptr` is NULL, then getline() will allocate a buffer for storing the line, which should be freed by the user program.  (In this case, the value in `*n` is ignored.) Alternatively,  before  calling  getline(),  `*lineptr` can contain a pointer to a malloc(3)-allocated buffer `*n` bytes in size.  If the buffer is not large enough to hold the line, getline() resizes it with realloc(3), updating `*lineptr` and `*n` as necessary. In either case, on a successful call,`*lineptr` and `*n` will be updated to reflect the buffer address and allocated size respectively

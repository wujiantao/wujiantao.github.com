---
title: c++中虚函数
layout: post
key: a84e4e07-3a48-42bb-b270-dead7d369951
tags:
  -c++ 
---


####作用####

虚函数的作用是实现动态联编，就是在程序运行阶段动态的选择合适的成员函数进行调用。如果在某个基类中定义了虚函数，那么可以在派生类中重新定义一个具有相同形参的函数(注意返回类型也要相同，如果不同程序会报错，这里和函数重载一样，程序不会根据返回类型来区别函数）。如果没有对虚函数进行重新定义，或者定义的函数形参类型或者个数不同那么，那么派生类都会继承基类的虚函数。如下所示。

	class Base
	{
		public:
			virtual void show()
			{
				printf("Base in show\n");
			}
			virtual void print()
			{
				printf("Base in print\n");
			}
	};

	class Sub : public Base
	{
		public:
			void show()
			{
				printf("Sub in show\n");
			}
			void print(int a)
			{
				printf("Sub in print\n");
			}
	};

	int main()
	{
		Base *p;
		Base  b;
		Sub   s;

		p = &b;
		p->show(); //Base in show
		p->print();//Base in print

		p = &s;
		p->show(); //Sub in show，根据指针指向的内容来决定调用哪一个函数。
		p->print();
		//Base in print，尽管指针指向的 s 类型为 Sub，但因为 Sub 中 print的形参与基类不一致，
		//会继承基类的 print函数，这里只是相当于实现了函数重载
	}

对于通常的非虚函数，程序在编译期间便根据调用对象的类型来决定调用哪一个成员函数。如将上面程序的virtual关键字去掉，那么下面的调用都是调用基类的成员函数。因为指针的类型为Base。

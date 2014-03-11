---
title: STL的迭代器(iterator_base)分析
layout: post
key: 60e9a431-f528-481e-aea6-e3732c557061
tags:
  -iterator
  -STL 
---
####说明####

stl\_iterator\_base.h 中定义了一些与迭代器相关的类，其使用几乎贯穿整个 STL 的实现，iterator\_base.h 中定义的类并不能直接为其他容器所使用，该文件中定义的所有类都没有数据成员，有的只是成员类型，这些类对迭代器进行了一个分类和规范，每个容器会遵照这种分类和规范设计和实现适合自身的迭代器。

<div class="cut"></div>

####分类####

程序开始处首先定义了五种迭代器标记，这五个结构体内部定义都为空，c++ 中没有任何成员的类，其实例占用一个字节的空间。五个迭代器标记用于区分不同的迭代器，迭代器约定俗成的设计规范，都会有一个成员类型 iterator\_category ，这个成员类型就是以下五种类型中的一种。用来表示当前迭代器的类型。

迭代器之间还有一些继承关系。random\_access\_iterator\_tag 继承自 bidirectional\_iterator\_tag，也即 random\_access\_iterator 类型的迭代器应该包含了 bidirectional\_iterator 所具有的性质。同样 bidirectional\_iterator\_tag 继承自 forward\_iterator\_tag。那么 bidirectional\_iterator 类型的迭代器也应该包含有 forward\_iterator 类型的迭代器的所有性质。

	struct input_iterator_tag {};
	struct output_iterator_tag {};
	struct forward_iterator_tag : public input_iterator_tag {};
	struct bidirectional_iterator_tag : public forward_iterator_tag {};
	struct random_access_iterator_tag : public bidirectional_iterator_tag {};

<div class="cut"></div>
####类模板 iterator####

紧接着定义了一个类模板 iterator 。该类模板有五个模板形参, \_Category 是上面定义的五种迭代器标记之一，用来标识迭代器的类型。\_Tp 表示迭代器指向的元素。  。两个迭代器之间可以计算其差值，\_Distance 表示用什么类型的数据存储其差值。\_Distance 的缺省实参为 ptrdiff\_t ，其中 ptrdifff\_t 是 signed long 的类型别名。迭代器可以通常都可以索引某个元素， \_Pointer 表示可以指向这种类型的元素的指针，\_Reference 表示对这种类型的元素的引用。

在 iterator 的内部定义了一些成为类型，其中 iterator\_category 为 \_Category 的类型别名，value\_type 为 \_Tp 的类型别名，difference\_type 为 \_Distance 的类型别名。 pointer 为 \_Pointer 类型的类型别名。\_Reference 为 reference 的类型别名。这五种成员类型 STL 中的每一种迭代器中都有定义。

	template <class _Category, class _Tp, class _Distance = ptrdiff_t,
		  class _Pointer = _Tp*, class _Reference = _Tp&>
	struct iterator {
	  typedef _Category  iterator_category;
	  typedef _Tp        value_type;
	  typedef _Distance  difference_type;
	  typedef _Pointer   pointer;
	  typedef _Reference reference;
	};

<div class="cut"></div>
####类模板 iterator\_traits####

iterator\_traits 可以看成是 iterator 的一个适配器，。其模板形参 \_Iterator 为一种迭代器。在内部根据 \_Iterator 重新定义了其自身的 iterator\_category, value\_type, difference\_type, pointer, reference 五种成员类型，因为前面已经说过 STL 中的所有迭代器都对这五种类型进行了定义，所以借助 iterator\_tarits 就可以获取到指定迭代器中的成员类型。

	template <class _Iterator>
	struct iterator_traits {
	  typedef typename _Iterator::iterator_category iterator_category;
	  typedef typename _Iterator::value_type        value_type;
	  typedef typename _Iterator::difference_type   difference_type;
	  typedef typename _Iterator::pointer           pointer;
	  typedef typename _Iterator::reference         reference;
	};

或许会将既然有了迭代器 \_Iterator 为什么不直接通过 \_Iterator 索引其成员类型，而多此一举的借助 iterator\_traits 来获取呢？我想第一个 iterator\_traits 可以提供一个统一的接口，第二个就是下面要介绍的，借助 iterator\_traits 可以将指针也看成是一种迭代器。因此实际的应用中，要获取一个迭代器的成员类型都是通过 iterator\_traits 来进行。

<div class="cut"></div>

iterator\_traits 的模板形参 \_Iterator 用指针进行实例化时，采用如下偏特化进行定义。在这种偏特化的定义中 iterator\_category 为 random\_access\_iterator\_tag 。value\_type 为 \_Tp，difference\_type 为 ptrdiff\_t，pointer 为 \_Tp\*，reference 为 \_Tp&。这也就是说在 STL 中将原始指针也看成是一种迭代器。

	template <class _Tp>
	struct iterator_traits<_Tp*> {
	  typedef random_access_iterator_tag iterator_category;
	  typedef _Tp                         value_type;
	  typedef ptrdiff_t                   difference_type;
	  typedef _Tp*                        pointer;
	  typedef _Tp&                        reference;
	};
<div class="cut"></div>

itertor\_traits 的模板形参 \_Iterator 用某种类型的常量指针进行实例化时，采用如下偏特化进行实例化。在这种偏特化的定义中 iterator\_category 为 random\_access\_iterator\_tag 。value\_type 为 \_Tp，difference\_type 为 ptrdiff\_t，pointer 为 const \_Tp\*，reference 为 const \_Tp&

	template <class _Tp>
	struct iterator_traits<const _Tp*> {
	  typedef random_access_iterator_tag iterator_category;
	  typedef _Tp                         value_type;
	  typedef ptrdiff_t                   difference_type;
	  typedef const _Tp*                  pointer;
	  typedef const _Tp&                  reference;
	};
<div class="cut"></div>
####相关函数####

函数模板 \_\_iterator\_category 用来返回某个迭代器实例的迭代器类型，函数的返回值为 iterator\_traits<\_Iter>::iterator\_category 的一个实例。\_Iter 为一种迭代器，通过 iterator\_traits 获取到 \_Iter 中定义的 iterator\_category 。\_Iter 中定义的 iterator\_category 为之前定义的五种迭代器标记中的一种。函数的形参为 \_Iter 类型的引用，它的作用就是用来获得到类型 \_Iter。

	template <class _Iter>
	inline typename iterator_traits<_Iter>::iterator_category
	__iterator_category(const _Iter&)
	{
	  typedef typename iterator_traits<_Iter>::iterator_category _Category;
	  return _Category();
	}

<div class="cut"></div>
函数模板 \_\_distance\_type 用来获取某种迭代器的 difference\_type 类型。也是借助 iterator\_traits 来获取 \_Iter 中定义的 difference\_type 类型。返回值为 iterator\_traits<\_Iter>::difference\_type 类型的空指针。函数的形参为 \_Iter 类型的一个引用。

	template <class _Iter>
	inline typename iterator_traits<_Iter>::difference_type*
	__distance_type(const _Iter&)
	{
	  return static_cast<typename iterator_traits<_Iter>::difference_type*>(0);
	}

<div class="cut"></div>
函数模板 \_\_value\_type 用来获取某种迭代器中的 value\_type 类型。函数模板的返回值为 iterator\_traits<\_Iter>::value\_type 类型的一个空指针 。

这里用 value\_type 类型的指针而不直接用 value\_type 类型的实例 。我想有两个好处，第一个返回值为指针，则不需声明一个 value\_type 的实例，当 value\_type 的实例占用空间较大时，返回指针就会省很大的空间，同时从效率上来说指针肯定比实例的声明也更快。第二个声明实例需要调用构造函数，如果不清楚 value\_type 中构造函数的定义，那么就会给声明实例带来困难，而返回指针，则不用考虑value\_type 中定义的任何细节，直接返回一个空指针即可。

	template <class _Iter>
	inline typename iterator_traits<_Iter>::value_type*
	__value_type(const _Iter&)
	{
	  return static_cast<typename iterator_traits<_Iter>::value_type*>(0);
	}

<div class="cut"></div>
函数 iterator\_type 用来获取某个迭代器的类型，函数模板 distance\_type 用来获取某个迭代器的 difference\_type 类型，函数模板 value\_type 用来获取某个迭代器的 value\_type 类型。这三个函数可以看成是对前面三个函数的一个封装。通过调用前面定义的函数来实现对应的功能。

	template <class _Iter>
	inline typename iterator_traits<_Iter>::iterator_category
	iterator_category(const _Iter& __i) { return __iterator_category(__i); }

	template <class _Iter>
	inline typename iterator_traits<_Iter>::difference_type*
	distance_type(const _Iter& __i) { return __distance_type(__i); }

	template <class _Iter>
	inline typename iterator_traits<_Iter>::value_type*
	value_type(const _Iter& __i) { return __value_type(__i); }

<div class="cut"></div>
函数模板 \_\_distance 用来计算两个迭代器之间的差值，模板形参 \_InputIterator 表示某种类型的迭代器，在该类型的迭代器中应该要对 operator++ 进行重载 (具体在定义迭代器时会对一些运算符函数进行重载，后续对容器进行介绍时，一并进行介绍)。

while 循环中对 \_\_first 迭代器应用后置自增操作，使得迭代器 \_\_first 朝着迭代器 \_\_last 移动。直到 \_\_first 和 \_\_last 相等。自增运算作用的次数就是两个迭代器的差值。
	
第三个函数形参限定了 \_InputIterator 为 input\_iterator 类型的迭代器 。即 \_InputIterator 中的 iterator\_category 为 input\_iterator\_tag (或者为 input\_iterator 的派生类)

	template <class _InputIterator>
	inline typename iterator_traits<_InputIterator>::difference_type
	__distance(_InputIterator __first, _InputIterator __last, input_iterator_tag)
	{
	  typename iterator_traits<_InputIterator>::difference_type __n = 0;
	  while (__first != __last) {
	    ++__first; ++__n;
	  }
	  return __n;
	}

<div class="cut"></div>
如果迭代器类型为 random\_access\_iterator 即其 iterator\_category 为 random\_access\_iterator\_tag。则直接调用 random\_access\_iterator 中定义的 operator- 函数。该函数用两个迭代器作为形参，计算两个迭代器之间的差值。

函数中的第一个语句为类型检测语句，它要求 \_RandomAccessIterator 为一个 random\_access\_iterator 类型的迭代器。同时第三个形参也用来限定 \_RandomAccessIterator 为 random\_access\_iterator 类型。

	template <class _RandomAccessIterator>
	inline typename iterator_traits<_RandomAccessIterator>::difference_type
	__distance(_RandomAccessIterator __first, _RandomAccessIterator __last,
		   random_access_iterator_tag) {
	  __STL_REQUIRES(_RandomAccessIterator, _RandomAccessIterator);
	  return __last - __first;
	}

<div class="cut"></div>
函数 distance 根据迭代器的类型来决定采用上面定义的哪种 \_\_distance 来计算两个迭代器的差值。

	template <class _InputIterator>
	inline typename iterator_traits<_InputIterator>::difference_type
	distance(_InputIterator __first, _InputIterator __last) {
	  typedef typename iterator_traits<_InputIterator>::iterator_category 
	    _Category;
	  __STL_REQUIRES(_InputIterator, _InputIterator);
	  return __distance(__first, __last, _Category());
	}

<div class="cut"></div>
函数模板 \_\_advance 用来将迭代器往后移动 \_\_n 个位置。第三个模板形参限定 _InputIter 为 input_iterator 类型的迭代器。 即\_InputIter 的 iterator\_category 要是 input\_iterator\_tag (或者 input\_iterator\_tag 的派生类)

	template <class _InputIter, class _Distance>
	inline void __advance(_InputIter& __i, _Distance __n, input_iterator_tag) {
	  while (__n--) ++__i;
	}

<div class="cut"></div>
如果迭代器类型为 bidirectional\_iterator (即其 iterator\_category 为 bidirectional\_iterator\_tag) 则可以前后移动迭代器的位置，这样根据\_\_n 的值的正负来判断是向前还是向后移动。这需要在 bidirectional\_iterator 类型的迭代器中对 operator++ 和 operator-- 都有定义。

	template <class _BidirectionalIterator, class _Distance>
	inline void __advance(_BidirectionalIterator& __i, _Distance __n, 
			      bidirectional_iterator_tag) {
	  __STL_REQUIRES(_BidirectionalIterator, _BidirectionalIterator);
	  if (__n >= 0)
	    while (__n--) ++__i;
	  else
	    while (__n++) --__i;
	}


<div class="cut"></div>
如果迭代器的类型为 random\_access\_iterator，则可以直接调用 operator+= 函数来移动迭代器的位置，同时 \_\_n 的值也是可正可负。

	template <class _RandomAccessIterator, class _Distance>
	inline void __advance(_RandomAccessIterator& __i, _Distance __n, 
			      random_access_iterator_tag) {
	  __STL_REQUIRES(_RandomAccessIterator, _RandomAccessIterator);
	  __i += __n;
	}

<div class="cut"></div>
函数模板 advance 用来将指定迭代器移动 \_\_n 个位置，函数内部会根据迭代器的 iterator\_category 来决定调用上面定义的哪个 \_\_advance 来移动迭代器的位置。

	template <class _InputIterator, class _Distance>
	inline void advance(_InputIterator& __i, _Distance __n) {
	  __STL_REQUIRES(_InputIterator, _InputIterator);
	  __advance(__i, __n, iterator_category(__i));
	}

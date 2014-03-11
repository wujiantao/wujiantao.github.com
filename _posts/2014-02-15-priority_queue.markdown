---
title: STL 的 priority_queue 分析
layout: post
key: 10bac18d-ce33-4cd8-bc97-f76774770b1f
tags:
  -priority_queue 
  -STL
---

priority\_queue 是一个类模板。有两个模板形参，分别为 \_Tp 和 \_Sequence 。其中 \_Tp 表示 priority\_queue 中的元素类型，\_Sequence 表示存储 priority\_queue 元素的底层容器的类型。\_Sequence 的缺省模板形参为 vector<\_Tp> 。\_Compare 是一个可以用来比较 priority\_queue 中不同元素的大小的一个函数对象 (\_Compare 中应重载 bool operator()(\_Tp&, \_Tp&) 函数)。\_Compare 的缺省形参为 less<typename \_Sequence::value\_type> 。less<\_Sequence::value\_type> 为一个函数对象，它的实例可以用来对两个 \_Sequence::value\_type 类型的元素进行比较。

	template <class _Tp, 
		  class _Sequence __STL_DEPENDENT_DEFAULT_TMPL(vector<_Tp>),
		  class _Compare
		  __STL_DEPENDENT_DEFAULT_TMPL(less<typename _Sequence::value_type>) >
	class priority_queue {

<div class="cut"></div>

priority\_queue 中定义了一些类型检测语句。第一句要求类型 \_Tp 是可赋值的。第二句要求类型 \_Sequence 是序列化的(这里瞎猜和乱说的) 。第三句要求 \_Sequence 类型是支持随机访问的容器。第四句定义了一个内部成员类型 \_Sequence\_value\_type ，他是 \_Sequence::value\_type 的类型别名。第五句要求 \_Tp 和 \_Sequence\_value\_type 是同一种类型。第六句要求 \_Compare 中定义了返回类型为 bool，第一个和第二个形参类型为 \_Tp 的函数 bool operator() (\_Tp, \_Tp) 。

	  __STL_CLASS_REQUIRES(_Tp, _Assignable);
	  __STL_CLASS_REQUIRES(_Sequence, _Sequence);
	  __STL_CLASS_REQUIRES(_Sequence, _RandomAccessContainer);
	  typedef typename _Sequence::value_type _Sequence_value_type;
	  __STL_CLASS_REQUIRES_SAME_TYPE(_Tp, _Sequence_value_type);
	  __STL_CLASS_BINARY_FUNCTION_CHECK(_Compare, bool, _Tp, _Tp);

<div class="cut"></div>

priority\_queue 中又定义了一些成员类型。

	  typedef typename _Sequence::value_type      value_type;
	  typedef typename _Sequence::size_type       size_type;
	  typedef          _Sequence                  container_type;

	  typedef typename _Sequence::reference       reference;
	  typedef typename _Sequence::const_reference const_reference;

<div class="cut"></div>

priority\_queue 中定义了两个保护类型的成员变量，其中 c 用来存储 priority\_queue 的元素，而 comp 用来对 priority\_queue 中的任意两个元素进行比较。

	  _Sequence c;
	  _Compare comp;

<div class="cut"></div>

priority\_queue 中定义了六个构造函数，第一个构造函数是空构造函数，在初始化列表中调用 c 的空构造函数对 成员变量 c 进行初始化。

	  priority_queue() : c() {}

<div class="cut"></div>

第二个构造函数有一个 \_Compare 类型的形参 \_\_x 。通过 \_\_x 来对 comp 进行初始化。并在初始化列表中对 c 进行初始化。

	  explicit priority_queue(const _Compare& __x) :  c(), comp(__x) {}
	  
<div class="cut"></div>

第三个构造函数用给定的 \_Compare 类型的变量 \_\_x 和 \_Sequence 类型的变量 \_\_s 分别对 c 和 comp 进行初始化，然后调用 make\_heap 对 c 中的元素构造一个的最大堆。

	  priority_queue(const _Compare& __x, const _Sequence& __s) 
	    : c(__s), comp(__x) 
	    { make_heap(c.begin(), c.end(), comp); }

<div class="cut"></div>

第四个构造函数用迭代器 \_\_first 到 \_\_last 之间的内容对 \_\_c 进行初始化。然后调用 make\_heap 为 c 中的元素构造最大堆。

	  template <class _InputIterator>
	  priority_queue(_InputIterator __first, _InputIterator __last) 
	    : c(__first, __last) { make_heap(c.begin(), c.end(), comp); }

<div class="cut"></div>

第五个构造函数用迭代器 \_\_first 和 \_\_last 之间的内容对 c 进行初始化，同时用 \_Compare 类型的变量 \_\_x 对 comp 进行初始化，再调用 make\_heap 为 c 中的内容构造最大堆。

	  template <class _InputIterator>
	  priority_queue(_InputIterator __first, 
			 _InputIterator __last, const _Compare& __x)
	    : c(__first, __last), comp(__x) 
	    { make_heap(c.begin(), c.end(), comp); }
	   
<div class="cut"></div>

第六个构造函数用 \_Sequence 类型的变量 \_\_s 和 \_Compare 类型的变量 \_\_x 分别对 c 和 comp 进行初始化。然后再将迭代器 \_\_first 到 \_\_last 之间的内容插入到 c 的尾部，再调用 make_heap 为 c 中的元素构造最大堆。 

	  template <class _InputIterator>
	  priority_queue(_InputIterator __first, _InputIterator __last,
			 const _Compare& __x, const _Sequence& __s)
	  : c(__s), comp(__x)
	  { 
	    c.insert(c.end(), __first, __last);
	    make_heap(c.begin(), c.end(), comp);
	  }

<div class="cut"></div>

empty() 函数判断当前 priority\_queue 是否为空。返回 c.empty() 。size() 函数返回 priority\_queue 中的元素个数。返回 c.size() 。top 函数返回 priority\_queue 中的最大元。最大元即为 c 的第一个元素，返回 c.front()。

	  bool empty() const { return c.empty(); }
	  size_type size() const { return c.size(); }
	  const_reference top() const { return c.front(); }

<div class="cut"></div>

push 函数用来向 priority\_queue 中插入一个新元素，并调整 priority\_queue 中的元素使得其符合最大堆的性质。函数先在 c 的尾部插入一个元素，然后调用 push\_heap 将最后一个元素加入到堆中。

	  void push(const value_type& __x) {
	    __STL_TRY {
	      c.push_back(__x); 
	      push_heap(c.begin(), c.end(), comp);
	    }
	    __STL_UNWIND(c.clear());
	  }

<div class="cut"></div>

pop() 函数用来将 priority\_queue 中的最大元弹出。函数首先调用 pop\_heap 将 c 的最大元(第一个元素) 放置到 c 的尾部。然后调用 c.pop\_back() 将这个被移到最尾部的元素弹出。

	  void pop() {
	    __STL_TRY {
	      pop_heap(c.begin(), c.end(), comp);
	      c.pop_back();
	    }
	    __STL_UNWIND(c.clear());
	  }

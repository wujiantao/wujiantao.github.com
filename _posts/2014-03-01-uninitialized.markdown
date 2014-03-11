---
title: STL 的 uninitialized 相关算法分析
layout: post
key: b6c4b201-5556-4b71-afc5-fb8497dcd26b
tags:
  -uninitialized
  -STL
---
stl\_uninitialized.h 中主要定义了 uninitialized\_copy uninitialized\_copy\_n, uninitialized\_fill 和 uninitialized\_fill\_n 函数。

<div class="cut"></div>

uninitialized\_copy 函数和 copy 函数的一个区别是，在 uninitialized\_copy 中根据需要复制的元素类型来决定是通过赋值运算还是通过按位 new 来构建新对象。但 copy 函数中一律使用赋值运算来进行元素的赋值，而有些对象 (比如未初始化的) 的复制需要用调用拷贝构造函数来实现(按位 new 会调用拷贝构造函数来实现)，而不是 operator= 函数来实现。

对于拷贝构造函数和 operator= 定义是一致的类型，copy 和 uninitialized\_copy 没有什么区别，但对于拷贝构造函数和 operator= 定义不同的类型，copy 和 uninitialized\_copy 则就存在不同了。fill 与 uninitialized\_fill 的关系也是如此。

<div class="cut"></div>
####函数  uninitialized\_copy ####

函数 \_\_uninitialized\_copy\_aux 用来将 \_\_first 到 \_\_last 之间的内容复制到 \_\_result 开始的位置上。对于那些拷贝构造函数和赋值函数(operator=)一致的类型，直接调用 copy 函数进行复制。第四个函数形参是 \_\_true\_type 用来限定 \_\_first 到 \_\_last 之间的元素都是 POD 类型的(POD 类型中的拷贝构造函数和operator= 的定义是一致的)。

	template <class _InputIter, class _ForwardIter>
	inline _ForwardIter 
	__uninitialized_copy_aux(_InputIter __first, _InputIter __last,
				 _ForwardIter __result,
				 __true_type)
	{
	  return copy(__first, __last, __result);
	}

<div class="cut"></div>

函数 uninitialized\_copy\_aux 用来将 \_\_first 到 \_\_last 之间的内容复制到 \_\_result 开始的位置上。对象的复制是通过调用 \_Construct 函数来实现，\_Construct 中调用按位 new 在指定的位置上调用拷贝构造函数来进行复制对象。因为第四个形参是 \_\_false\_type。表示 result 所在位置的元素类型不是 POD 类型(非 POD 类型其拷贝构造函数和赋值函数的定义可能会不同)，所以调用 \_Construct 函数来复制对象，而不是通过 operator= 函数来复制。

	template <class _InputIter, class _ForwardIter>
	_ForwardIter 
	__uninitialized_copy_aux(_InputIter __first, _InputIter __last,
				 _ForwardIter __result,
				 __false_type)
	{
	  _ForwardIter __cur = __result;
	  __STL_TRY {
	    for ( ; __first != __last; ++__first, ++__cur)
	      _Construct(&*__cur, *__first);
	    return __cur;
	  }
	  __STL_UNWIND(_Destroy(__result, __cur));
	}

<div class="cut"></div>

函数 \_\_uninitialized\_copy 函数将 \_\_first 到 \_\_last 之间的内容赋值到 \_\_result 开始的位置。其中第四个形参 Tp\* 用来传递类型 \_Tp。当 \_Tp 为 POD 类型时, \_Is\_POD 为 \_\_true\_type 的类型别名，否则它为 \_\_false\_type 的类型别名。根据 \_Is\_POD 类型的不同分别调用上面定义的不同的 \_\_uninitialized\_copy\_aux 函数来实现复制的功能。

	template <class _InputIter, class _ForwardIter, class _Tp>
	inline _ForwardIter
	__uninitialized_copy(_InputIter __first, _InputIter __last,
			     _ForwardIter __result, _Tp*)
	{
	  typedef typename __type_traits<_Tp>::is_POD_type _Is_POD;
	  return __uninitialized_copy_aux(__first, __last, __result, _Is_POD());
	}

<div class="cut"></div>

函数 uninitialized\_copy 将 \_\_first 到 \_\_last 之间的内容复制到 \_\_result 开始的位置。宏定义 \_\_VALUE\_TYPE 用来返回 \_\_result 所在位置的元素类型的空指针。

	template <class _InputIter, class _ForwardIter>
	inline _ForwardIter
	  uninitialized_copy(_InputIter __first, _InputIter __last,
			     _ForwardIter __result)
	{
	  return __uninitialized_copy(__first, __last, __result,
				      __VALUE_TYPE(__result));
	}

<div class="cut"></div>

当 \_\_first 和 \_\_last 是 const char\* (const wchar\_t\*) 类型，\_\_result 是 char\* (wchar\_t\*)类型时，使用 memmove 实现数据的复制。

	inline char* uninitialized_copy(const char* __first, const char* __last,
					char* __result) {
	  memmove(__result, __first, __last - __first);
	  return __result + (__last - __first);
	}

	inline wchar_t* 
	uninitialized_copy(const wchar_t* __first, const wchar_t* __last,
			   wchar_t* __result)
	{
	  memmove(__result, __first, sizeof(wchar_t) * (__last - __first));
	  return __result + (__last - __first);
	}

<div class="cut"></div>

函数 \_\_uninitialized\_copy\_n 用来将从 \_\_first 开始长度为 \_\_count 的元素序列复制到 \_\_result 开始的位置上。第四个形参限定 \_\_first 是 input\_iterator 类型的迭代器。for 循环中通过 \_Construct 函数来逐个进行元素的复制。

	template <class _InputIter, class _Size, class _ForwardIter>
	pair<_InputIter, _ForwardIter>
	__uninitialized_copy_n(_InputIter __first, _Size __count,
			       _ForwardIter __result,
			       input_iterator_tag)
	{
	  _ForwardIter __cur = __result;
	  __STL_TRY {
	    for ( ; __count > 0 ; --__count, ++__first, ++__cur) 
	      _Construct(&*__cur, *__first);
	    return pair<_InputIter, _ForwardIter>(__first, __cur);
	  }
	  __STL_UNWIND(_Destroy(__result, __cur));
	}

<div class="cut"></div>

####函数 uninitialized\_copy\_n####

函数 \_\_uninitialized\_copy\_n 将从 \_\_first 开始长度为 \_\_count 的元素序列复制到\_\_result 开始的位置上，第四个形参限定了 \_\_first 的迭代器类型为 random\_access\_iterator 类型。函数通过求得结束地址 \_\_last，然后调用 uninitialized\_copy 函数来进行复制。

	template <class _RandomAccessIter, class _Size, class _ForwardIter>
	inline pair<_RandomAccessIter, _ForwardIter>
	__uninitialized_copy_n(_RandomAccessIter __first, _Size __count,
			       _ForwardIter __result,
			       random_access_iterator_tag) {
	  _RandomAccessIter __last = __first + __count;
	  return pair<_RandomAccessIter, _ForwardIter>(
			 __last,
			 uninitialized_copy(__first, __last, __result));
	}

<div class="cut"></div>

函数 uninitialized\_copy\_n 将从 \_\_first 开始长度为 \_\_count 的元素序列复制到 \_\_result 开始的位置上。通过 \_\_first 的迭代器来分别调用上面定义的不同的 \_\_unintialized\_copy\_n 函数来实现复制的功能。

	template <class _InputIter, class _Size, class _ForwardIter>
	inline pair<_InputIter, _ForwardIter>
	uninitialized_copy_n(_InputIter __first, _Size __count,
			     _ForwardIter __result) {
	  return __uninitialized_copy_n(__first, __count, __result,
					__ITERATOR_CATEGORY(__first));
	}

<div class="cut"></div>

####函数 uninitialized\_fill####

函数 uninitialized\_fill 将从 \_\_first 开始到 \_\_last 结束的位置上的元素都置为 \_\_x 。函数直接调用 fill 函数来进行实现，其中第四个形参为 \_\_true\_type 类型表示 \_\_first 所在位置的元素类型为 POD 类型(POD 类型的拷贝构造函数和赋值函数定义是一致的)。可以直接调用赋值函数进行复制，而 fill 函数中正是使用赋值函数进行复制的。

	template <class _ForwardIter, class _Tp>
	inline void
	__uninitialized_fill_aux(_ForwardIter __first, _ForwardIter __last, 
				 const _Tp& __x, __true_type)
	{
	  fill(__first, __last, __x);
	}

<div class="cut"></div>

函数 uninitialized\_fill 将从 \_\_first 到 \_\_last 之间的位置上的元素值都置为 \_\_x 。其中第四个形参为 \_\_false\_type 类型，说明 \_\_first 所在位置的元素类型不是 POD 类型，则其拷贝构造函数的定义和赋值函数的定义可能不相同，因此使用 \_Construct 函数来调用按位 new 用拷贝构造函数来复制元素。

	template <class _ForwardIter, class _Tp>
	void
	__uninitialized_fill_aux(_ForwardIter __first, _ForwardIter __last, 
				 const _Tp& __x, __false_type)
	{
	  _ForwardIter __cur = __first;
	  __STL_TRY {
	    for ( ; __cur != __last; ++__cur)
	      _Construct(&*__cur, __x);
	  }
	  __STL_UNWIND(_Destroy(__first, __cur));
	}

<div class="cut"></div>

函数 \_\_uninitialized\_fill 将 \_\_first 到 \_\_last 之间的位置上的元素都置为 \_\_x。第四个形参 \_Tp1 表示 \_\_first 所在位置的元素类型，如果 \_Tp1 为 POD 类型则 \_Is\_POD 为 \_true\_type 的类型别名，否而其为 \_\_false\_type 的类型别名。然后根据 \_Is\_POD 的不同类型分别调用上面定义的不同的 \_\_uninitialized\_fill 函数。

	template <class _ForwardIter, class _Tp, class _Tp1>
	inline void __uninitialized_fill(_ForwardIter __first, 
					 _ForwardIter __last, const _Tp& __x, _Tp1*)
	{
	  typedef typename __type_traits<_Tp1>::is_POD_type _Is_POD;
	  __uninitialized_fill_aux(__first, __last, __x, _Is_POD());
			   
	}

<div class="cut"></div>

函数 unintialized\_fill 将从 \_\_first 开始到 \_\_last 结束的位置上的元素值都置为 \_\_x 。通过调用 \_\_uninitialized\_fill 函数来实现，其中 \_\_VALUE\_TYPE 返回 \_\_first 所在位置的元素类型的空指针。

	template <class _ForwardIter, class _Tp>
	inline void uninitialized_fill(_ForwardIter __first,
				       _ForwardIter __last, 
				       const _Tp& __x)
	{
	  __uninitialized_fill(__first, __last, __x, __VALUE_TYPE(__first));
	}

<div class="cut"></div>

####函数 uninitialized\_fill\_n####

函数 \_\_uninitialized\_fill\_n\_aux 用来将从 \_\_fisrt 开始长度为 \_\_n 的元素序列中的元素都置为 \_\_x 。第四个函数形参为 \_\_true\_type 限定了 \_\_first 所在位置的元素类型为 POD 类型。直接调用 fill\_n 函数来实现。

	template <class _ForwardIter, class _Size, class _Tp>
	inline _ForwardIter
	__uninitialized_fill_n_aux(_ForwardIter __first, _Size __n,
				   const _Tp& __x, __true_type)
	{
	  return fill_n(__first, __n, __x);
	}

<div class="cut"></div>

函数 \_\_uninitialized\_fill\_n\_aux 将从 \_\_first 开始长度为 \_\_n 的元素序列中的元素都置为 \_\_x 。第四个形参为 \_\_false\_type 限定了 \_\_first 所在位置的元素类型不是 POD 类型。则调用 \_Construct 函数使用拷贝构造函数来进行复制。

	template <class _ForwardIter, class _Size, class _Tp>
	_ForwardIter
	__uninitialized_fill_n_aux(_ForwardIter __first, _Size __n,
				   const _Tp& __x, __false_type)
	{
	  _ForwardIter __cur = __first;
	  __STL_TRY {
	    for ( ; __n > 0; --__n, ++__cur)
	      _Construct(&*__cur, __x);
	    return __cur;
	  }
	  __STL_UNWIND(_Destroy(__first, __cur));
	}

<div class="cut"></div>

函数 \_\_uninitialized\_fill\_n 根据 \_Tp1 类型是否为 POD 类型来分别调用上面定义的不同的 \_\_uninitialized\_fill\_n\_aux 函数来实现填充的功能。

	template <class _ForwardIter, class _Size, class _Tp, class _Tp1>
	inline _ForwardIter 
	__uninitialized_fill_n(_ForwardIter __first, _Size __n, const _Tp& __x, _Tp1*)
	{
	  typedef typename __type_traits<_Tp1>::is_POD_type _Is_POD;
	  return __uninitialized_fill_n_aux(__first, __n, __x, _Is_POD());
	}

<div class="cut"></div>

函数 uninitialized\_fill\_n 将从 \_\_first 开始往后 \_\_n 个位置上的元素都置为 \_\_x 。通过调用 \_\_uninitialized\_fill\_n 来实现。

	template <class _ForwardIter, class _Size, class _Tp>
	inline _ForwardIter 
	uninitialized_fill_n(_ForwardIter __first, _Size __n, const _Tp& __x)
	{
	  return __uninitialized_fill_n(__first, __n, __x, __VALUE_TYPE(__first));
	}

<div class="cut"></div>

函数 \_\_uninitialized\_copy\_copy 先将 \_\_first1 到 \_\_last1 之间的元素复制到 \_\_result 开始的位置，并获取到截止位置 \_\_mid。然后再将 \_\_first2 到 \_\_last2 之间的元素复制到 \_\_mid 开始的位置。通过调用两次 uninitialized\_copy 来实现。

	template <class _InputIter1, class _InputIter2, class _ForwardIter>
	inline _ForwardIter
	__uninitialized_copy_copy(_InputIter1 __first1, _InputIter1 __last1,
				  _InputIter2 __first2, _InputIter2 __last2,
				  _ForwardIter __result)
	{
	  _ForwardIter __mid = uninitialized_copy(__first1, __last1, __result);
	  __STL_TRY {
	    return uninitialized_copy(__first2, __last2, __mid);
	  }
	  __STL_UNWIND(_Destroy(__result, __mid));
	}

<div class="cut"></div>

函数 \_\_uninitialized\_fill\_copy 先将 \_\_first 到 \_\_mid 之间的元素都置为 \_\_x ，然后将 \_\_first 到 \_\_last 之间的元素赋值到 \_\_mid 开始的位置上。通过调用一次 uninitialized\_fill 和一次 uninitialized\_copy 函数来实现。

	template <class _ForwardIter, class _Tp, class _InputIter>
	inline _ForwardIter 
	__uninitialized_fill_copy(_ForwardIter __result, _ForwardIter __mid,
				  const _Tp& __x,
				  _InputIter __first, _InputIter __last)
	{
	  uninitialized_fill(__result, __mid, __x);
	  __STL_TRY {
	    return uninitialized_copy(__first, __last, __mid);
	  }
	  __STL_UNWIND(_Destroy(__result, __mid));
	}

<div class="cut"></div>

函数 \_\_uninitialized\_copy\_fill 先将 \_\_first1 到 \_\_last1 之间的内容复制到 \_\_first2 开始的位置上，并得到结束位置 \_\_mid2 。然后将 \_\_mid2 到 \_\_last2 之间的位置上的元素都置为 \_\_x。通过调用一次 uninitialized\_copy 和一次 uninitialized\_fill 来实现。

	template <class _InputIter, class _ForwardIter, class _Tp>
	inline void
	__uninitialized_copy_fill(_InputIter __first1, _InputIter __last1,
				  _ForwardIter __first2, _ForwardIter __last2,
				  const _Tp& __x)
	{
	  _ForwardIter __mid2 = uninitialized_copy(__first1, __last1, __first2);
	  __STL_TRY {
	    uninitialized_fill(__mid2, __last2, __x);
	  }
	  __STL_UNWIND(_Destroy(__first2, __mid2));
	}

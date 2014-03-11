---
title: STL 的 char_traits 分析
layout: post
key: 3d03587c-8a12-4382-827b-7ed922c08848
tags:
  -char_traits 
  -STL
---
类模板 \_\_char\_traits\_base 用来作为 char\_traits 的基类。

	template <class _CharT, class _IntT> class __char_traits_base {

<div class="cut"></div>

####类模板 \_\_char\_traits\_base####

\_\_char\_traits\_base 中定义了两个公有属性的成员类型。

	public:
	  typedef _CharT char_type;
	  typedef _IntT int_type;

<div class="cut"></div>

\_\_char\_traits\_base 定义了一系列的成员函数，char\_traits 中大量的成员函数都是继承自基类。

<div class="cut"></div>

函数 assign 用 char\_type 类型的给定变量 \_\_c2 对另一个 char\_type 类型的变量进行赋值。

	  static void assign(char_type& __c1, const char_type& __c2) { __c1 = __c2; }

<div class="cut"></div>

函数 eq 判断两个类型为 char\_type 的变量是否相等。

	  static bool eq(const _CharT& __c1, const _CharT& __c2) 
	    { return __c1 == __c2; }

<div class="cut"></div>

函数 lt 判断给定的 char\_type 类型的变量 \_\_c1 是否要小于另一个同类型的变量 \_\_c2 。

	  static bool lt(const _CharT& __c1, const _CharT& __c2) 
	    { return __c1 < __c2; }

<div class="cut"></div>

函数比较从 \_\_s1 开始长度为 \_\_n 的序列是否和从 \_\_s2 开始长度为 \_\_n 的序列大小，如果第一个序列小，则返回 -1，如果第一个序列大，则返回 1 。如果两个序列相等，则返回 0 。

	  static int compare(const _CharT* __s1, const _CharT* __s2, size_t __n) {
	    for (size_t __i = 0; __i < __n; ++__i)
	      if (!eq(__s1[__i], __s2[__i]))
		return __s1[__i] < __s2[__i] ? -1 : 1;
	    return 0;
	  }

<div class="cut"></div>

函数 length 计算从 \_\_s 开始的序列的长度，函数声明一个 CharT 类型的空实例 \_\_nullchar 。从 \_\_s 开始往后遍历，如果碰到一个为 \_\_nullchar 的元素，则停止遍历，并返回 \_\_nullchar 之前的元素个数。

	  static size_t length(const _CharT* __s) {
	    const _CharT __nullchar = _CharT();
	    size_t __i;
	    for (__i = 0; !eq(__s[__i], __nullchar); ++__i)
	      {}
	    return __i;
	  }

<div class="cut"></div>

在从 \_\_s 开始长度为 \_\_n 的序列中查找值为 \_\_c 的元素。

	  static const _CharT* find(const _CharT* __s, size_t __n, const _CharT& __c)
	  {
	    for ( ; __n > 0 ; ++__s, --__n)
	      if (eq(*__s, __c))
		return __s;
	    return 0;
	  }

<div class="cut"></div>

函数 move 调用 memmove 将从 \_\_s2 开始长度为 \_\_n 的序列移动到 \_\_s1 开始的位置。memmove 能保证当 \_\_s2 和 \_\_s1 之间有重叠时也能顺利移动。

	  static _CharT* move(_CharT* __s1, const _CharT* __s2, size_t __n) {
	    memmove(__s1, __s2, __n * sizeof(_CharT));
	    return __s1;
	  }

<div class="cut"></div>

函数 copy 调用 memcpy 将从 \_\_s2 开始长度为 \_\_n 的序列复制到 \_\_s1 开始的位置。当 \_\_s1 和 \_\_s2 之间有重叠时可能会产生覆盖的问题。

	  static _CharT* copy(_CharT* __s1, const _CharT* __s2, size_t __n) {
	    memcpy(__s1, __s2, __n * sizeof(_CharT));
	    return __s1;
	  } 

<div class="cut"></div>

函数 assign 将 \_\_s 开始长度为 \_\_n 的序列中的元素都赋值为 \_\_c 。

	  static _CharT* assign(_CharT* __s, size_t __n, _CharT __c) {
	    for (size_t __i = 0; __i < __n; ++__i)
	      __s[__i] = __c;
	    return __s;
	  }

<div class="cut"></div>

函数 not\_eof 判断 \_\_c 是否为 eof 。

	  static int_type not_eof(const int_type& __c) {
	    return !eq_int_type(__c, eof()) ? __c : 0;
	  }

<div class="cut"></div>

函数 to\_char\_type 将 int\_type 类型的变量 \_\_c 强制转换为 char\_type 类型的变量，并将强制转换得到的变量作为返回值返回。

	  static char_type to_char_type(const int_type& __c) {
	    return static_cast<char_type>(__c);
	  }

<div class="cut"></div>

函数 to\_int\_type 将 char\_type 类型的变量 \_\_c 强制转换为 int\_type 类型的变量，并将强制转换得到的变量作为返回值返回。

	  static int_type to_int_type(const char_type& __c) {
	    return static_cast<int_type>(__c);
	  }

<div class="cut"></div>

函数 eq\_int\_type 判断两个 int\_type 类型的变量 \_\_c1 和 \_\_c2 是否相等。

	  static bool eq_int_type(const int_type& __c1, const int_type& __c2) {
	    return __c1 == __c2;
	  }

<div class="cut"></div>

函数 eof 用来返回 eof(即 -1)。

	  static int_type eof() {
	    return static_cast<int_type>(-1);
	  }

<div class="cut"></div>
####类模板 char\_traits####

类模板 char\_traits 继承自 \_\_char\_traits\_base<\_CharT, \_CharT> 。

	template <class _CharT> class char_traits
	  : public __char_traits_base<_CharT, _CharT>
	{};

<div class="cut"></div>

当 char\_traits 的模板实参为 char 时， char\_traits 有如下的偏特化定义。该定义中重定义了 to\_char\_type, to\_int\_type, compare, length, assign 函数。

to\_char\_type 函数将 int 类型的变量强制转换为 char 类型，to\_int\_type 类型变量将 char 类型变量转换为 int 类型。 compare 调用 memcmp 比较两个字符串的大小。 length 函数调用 strlen 求给定字符串的长度。 assign 函数实现 const char 类型的变量对 char 类型变量的赋值，而第二个 assign 函数调用 memset 对从 \_\_s 开始长度为 \_\_n 的字符串中的字符赋同一个值。

	__STL_TEMPLATE_NULL class char_traits<char> 
	  : public __char_traits_base<char, int>
	{
	public:
	  static char_type to_char_type(const int_type& __c) {
	    return static_cast<char_type>(static_cast<unsigned char>(__c));
	  }

	  static int_type to_int_type(const char_type& __c) {
	    return static_cast<unsigned char>(__c);
	  }

	  static int compare(const char* __s1, const char* __s2, size_t __n) 
	    { return memcmp(__s1, __s2, __n); }
	  
	  static size_t length(const char* __s) { return strlen(__s); }

	  static void assign(char& __c1, const char& __c2) { __c1 = __c2; }

	  static char* assign(char* __s, size_t __n, char __c)
	    { memset(__s, __c, __n); return __s; }
	};

<div class="cut"></div>

当char\_traits 的模板实参为 wchar\_t 时使用如下偏特化定义。

	__STL_TEMPLATE_NULL class char_traits<wchar_t>
	  : public __char_traits_base<wchar_t, wint_t>
	{};

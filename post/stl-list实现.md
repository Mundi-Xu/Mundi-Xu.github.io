# stl_list 介绍

今天我们来总结一下stl_List, 与单链表比较而言，stl_list无非就是链表结构不一样，至于其中的增删改查的细节实现本质是一样的，都是处理指针偏移。**相比于vector，stl_List在插入和删除的时候可以达到O(1)的时间复杂度**。

stl_list是一个**双向循环链表**，相对单链表来说查找效率高，无论是插入时的前插和后插，还是从后往前查找某个元素等。既然查找效率高了，自然添加，删除和修改元素时效率也就更高。唯一一个可以称为不足的就是每个节点需要耗费4字节指针来保存前一个节点的地址，因此如果遇到对内存要求比较苛刻的场景，而且一些操作单链表即可满足，那么可以考虑使用标准库中的**forward_list**（单链表）。

# stl_list 源码分析

分析gnu c++标准库中的stl_list，我们只需把握住整体结构即可，实现总共由三部分组成，**链表节点**(struct _List_node : public __detail::_List_node_base) ，**迭代器**（struct _List_iterator），**链表数据结构**（class list : protected _List_base<_Tp, _Alloc>）。

gnu下最新版本的stl_list实现加了一些额外的继承关系，_list_base中保存了一个_List_impl _M_impl中间变量，由该类_M_impl来保存节点，并对节点做基本处理。

## 链表节点

父类维护两个指针，子类才加入具体的value。

```cpp
    struct _List_node_base
    {
        _List_node_base* _M_next;
        _List_node_base* _M_prev;

    };

        template<typename _Tp>
    struct _List_node : public __detail::_List_node_base
    {
        ///< User's data.
        _Tp _M_data;

    };
```

## 迭代器

主要是实现++和--等操作符重载，实现链表节点的前后移动。

```cpp
template<typename _Tp>
    struct _List_iterator
    {
        typedef _List_iterator<_Tp>                _Self;
        typedef _List_node<_Tp>                    _Node;

        typedef ptrdiff_t                          difference_type;
        typedef std::bidirectional_iterator_tag    iterator_category;
        typedef _Tp                                value_type;
        typedef _Tp*                               pointer;
        typedef _Tp&                               reference;

        _List_iterator() _GLIBCXX_NOEXCEPT
        : _M_node() { }

        explicit
        _List_iterator(__detail::_List_node_base* __x) _GLIBCXX_NOEXCEPT
        : _M_node(__x) { }

        _Self
        _M_const_cast() const _GLIBCXX_NOEXCEPT
        { return *this; }

        // Must downcast from _List_node_base to _List_node to get to _M_data.
        reference
        operator*() const _GLIBCXX_NOEXCEPT
        { return static_cast<_Node*>(_M_node)->_M_data; }

        pointer
        operator->() const _GLIBCXX_NOEXCEPT
        { return std::__addressof(static_cast<_Node*>(_M_node)->_M_data); }

        _Self&
        operator++() _GLIBCXX_NOEXCEPT
        {
            _M_node = _M_node->_M_next;    //本质是链表节点的next指针操作
            return *this;
        }

        _Self
        operator++(int) _GLIBCXX_NOEXCEPT
        {
            _Self __tmp = *this;
            _M_node = _M_node->_M_next;
            return __tmp;
        }

        _Self&
        operator--() _GLIBCXX_NOEXCEPT
        {
            _M_node = _M_node->_M_prev;  //本质是链表节点的prev指针操作
            return *this;
        }

        _Self
        operator--(int) _GLIBCXX_NOEXCEPT
        {
            _Self __tmp = *this;
            _M_node = _M_node->_M_prev;
            return __tmp;
        }

        bool
        operator==(const _Self& __x) const _GLIBCXX_NOEXCEPT
        { return _M_node == __x._M_node; }

        bool
        operator!=(const _Self& __x) const _GLIBCXX_NOEXCEPT
        { return _M_node != __x._M_node; }

        // The only member points to the %list element.
        __detail::_List_node_base* _M_node; //维护一个链表节点
    };
```

## 链表数据结构

实现类 _List_impl，主要用来维护链表节点，然后list类包含该类。

```cpp
struct _List_impl
      : public _Node_alloc_type
      {

    __detail::_List_node_base _M_node;  //其实就是维护节点，标准库中用了一个中间层来处理

    _List_impl()
    : _Node_alloc_type(), _M_node()
    { }

    _List_impl(const _Node_alloc_type& __a) _GLIBCXX_NOEXCEPT
    : _Node_alloc_type(__a), _M_node()
    { }

#if __cplusplus >= 201103L
    _List_impl(_Node_alloc_type&& __a) _GLIBCXX_NOEXCEPT
    : _Node_alloc_type(std::move(__a)), _M_node()
    { }
#endif
      };
```

_List_base类

```cpp
 template<typename _Tp, typename _Alloc>
    class _List_base
    {
    protected:

      typedef typename _Alloc::template rebind<_List_node<_Tp> >::other  _Node_alloc_type;

      typedef typename _Alloc::template rebind<_Tp>::other _Tp_alloc_type;

      static size_t
      _S_distance(const __detail::_List_node_base* __first,
          const __detail::_List_node_base* __last)
      {
    size_t __n = 0;
    while (__first != __last)
      {
        __first = __first->_M_next;
        ++__n;
      }
    return __n;
      }

      _List_impl _M_impl;    // 中间层类

      // count the number of nodes
      size_t _M_node_count() const
      {
    return _S_distance(_M_impl._M_node._M_next,
               std::__addressof(_M_impl._M_node));
      }


  public:
      typedef _Alloc allocator_type;

      void
      _M_clear() _GLIBCXX_NOEXCEPT;

      void
      _M_init() _GLIBCXX_NOEXCEPT
      {
        this->_M_impl._M_node._M_next = &this->_M_impl._M_node;
        this->_M_impl._M_node._M_prev = &this->_M_impl._M_node;
    _M_set_size(0);
      }
    };
```

list类

```cpp
template<typename _Tp, typename _Alloc = std::allocator<_Tp> >
    class list : protected _List_base<_Tp, _Alloc>
    {
      // concept requirements
      typedef typename _Alloc::value_type                _Alloc_value_type;
      __glibcxx_class_requires(_Tp, _SGIAssignableConcept)
      __glibcxx_class_requires2(_Tp, _Alloc_value_type, _SameTypeConcept)

      typedef _List_base<_Tp, _Alloc>                    _Base;
      typedef typename _Base::_Tp_alloc_type         _Tp_alloc_type;
      typedef typename _Base::_Node_alloc_type       _Node_alloc_type;

    public:
      typedef _Tp                                        value_type;
      typedef typename _Tp_alloc_type::pointer           pointer;
      typedef typename _Tp_alloc_type::const_pointer     const_pointer;
      typedef typename _Tp_alloc_type::reference         reference;
      typedef typename _Tp_alloc_type::const_reference   const_reference;
      typedef _List_iterator<_Tp>                        iterator;
      typedef _List_const_iterator<_Tp>                  const_iterator;
      typedef std::reverse_iterator<const_iterator>      const_reverse_iterator;
      typedef std::reverse_iterator<iterator>            reverse_iterator;
      typedef size_t                                     size_type;
      typedef ptrdiff_t                                  difference_type;
      typedef _Alloc                                     allocator_type;

    protected:
      // Note that pointers-to-_Node's can be ctor-converted to
      // iterator types.
      typedef _List_node<_Tp>                _Node;

      using _Base::_M_impl;
      using _Base::_M_put_node;
      using _Base::_M_get_node;
      using _Base::_M_get_Tp_allocator;
      using _Base::_M_get_Node_allocator;

       ..........................................................

}
```
大概截取了stl_list实现的一部分，主要为了体现stl_list的代码结构，具体接口实现可以查看源码。

# stl-list简单实现

## STL_List.h

```cpp
#ifndef STL_LIST
#define STL_LIST

#include "Def.h"

__MUNDI_BEGIN


template <typename T> 
class list
{
public:
  // The list node, the parent class maintains two pointers, and the subclass adds the specific value.
  struct list_node_base  
  {
    list_node_base* Next;
    list_node_base* Prev;

    list_node_base():Next(nullptr), Prev(nullptr){}
  };

  // dataEntry node
  struct list_node: public list_node_base
  {
     T dataEntry;
  };

  // iterator
  struct list_iterator
  {
    typedef list_iterator   _Self;
    typedef T               value_type;
    typedef T*              pointer;
    typedef T&              reference;

    list_iterator() _T_STD_NOEXCEPT
    {
      m_smartPtr = nullptr;
    }

    explicit list_iterator(list_node_base * pNode) _T_STD_NOEXCEPT
    {
      m_smartPtr = pNode;
    }

    reference operator*() _T_STD_NOEXCEPT
    {
      return  static_cast<list_node *>(m_smartPtr)->dataEntry;
    }

    list_node_base* operator->() _T_STD_NOEXCEPT
    {
      return m_smartPtr;
    }

    _Self operator++(int) _T_STD_NOEXCEPT // post increment
    {
      _Self __tmp = *this;
      m_smartPtr = m_smartPtr->Next;
      return __tmp;
    }

    _Self& operator++() _T_STD_NOEXCEPT // pre increment
    {
      m_smartPtr = m_smartPtr->Next;
      return *this;
    }

    _Self operator--(int) _T_STD_NOEXCEPT
    {
      _Self __tmp = *this;
      m_smartPtr = m_smartPtr->Prev;
      return __tmp;
    }

    _Self& operator--() _T_STD_NOEXCEPT
    {
      m_smartPtr = m_smartPtr->Prev;
      return *this;
    }

    bool operator==(const list_iterator & _Right) const _T_STD_NOEXCEPT
    {
      return m_smartPtr == _Right.m_smartPtr;
    }

    bool operator!=(const list_iterator & _Right) const _T_STD_NOEXCEPT
    {
       return m_smartPtr != _Right.m_smartPtr;
    }

    list_node_base * m_smartPtr; // Node pointer
  };

public:
  typedef list_iterator iterator;

public:
  list()  // Default constructor
  { 
    empty_init();
  }

  list(const list<T> & rhs) // Copy construction
  {
    if(this != &rhs)
    {
      empty_init(); // initialization

      iterator itrBegin = rhs.begin();
      iterator itrEnd = rhs.end();

      while(itrBegin != itrEnd)
      {
         list_node * tmp = static_cast<list_node *>(itrBegin.m_smartPtr);

         push_back(tmp->dataEntry);

         ++itrBegin;
      }
    }
  }

  list & operator = (const list<T> & rhs) // Assignment operator overloading
  {
    if(this != &rhs)
    {
      // If the original list has a value, it will be emptied first.
      if(begin() != end())
      {
        clear();
      }

      iterator itrBegin = rhs.begin();
      iterator itrEnd = rhs.end();

      while(itrBegin != itrEnd)
      {
         list_node * tmp = static_cast<list_node *>(itrBegin.m_smartPtr);

         push_back(tmp->dataEntry);

         ++itrBegin;
      }
    }
  }

  ~list()  // Destructor
  {
    clear();

    if(pHeadNode)
    {
      delete pHeadNode;
      pHeadNode = nullptr;
    }
  }

  iterator begin() _T_STD_NOEXCEPT
  {
    return iterator(pHeadNode->Next);
  }

  iterator end() _T_STD_NOEXCEPT
  {
    return iterator(pHeadNode);
  }

  void push_back(const T & value)
  {
    insert(end(), value);
  }

  void push_front(const T & value)
  {
    insert(begin(), value);
  }

  void pop_front() 
  {
     erase(begin()); 
  }

  void pop_back() 
  { 
    iterator tmp = end();
    erase(--tmp);
  }

  T & front()
  {
    return *begin();
  }

  T & back()
  {
    return *(--end());
  }

  unsigned int remove(const T & value)
  {
    unsigned int count = 0;

    iterator itrBegin = begin();
    while(itrBegin != end())
    {
      if(*itrBegin == value)
      {
        itrBegin = erase(itrBegin);
        ++count;
      }
      else
      {
        ++itrBegin;
      }
    }

    return count;
  }

  iterator erase(iterator position)
  {
    list_node_base* next_node = position.m_smartPtr->Next;
    list_node_base* prev_node = position.m_smartPtr->Prev;
    prev_node->Next = next_node;
    next_node->Prev = prev_node;

    delete position.m_smartPtr;
    position.m_smartPtr = nullptr;
    
    if(_size > 0)
    {
      _size--;
    }

    return iterator(next_node);
  }

  iterator insert(iterator position, const T& x) 
  {
    list_node* tmp = new list_node();
    tmp->dataEntry = x;
    tmp->Next = position.m_smartPtr;
    tmp->Prev = position.m_smartPtr->Prev;
    position.m_smartPtr->Prev->Next = tmp;
    position.m_smartPtr->Prev = tmp;

    ++_size;
    return iterator(tmp);
  }

  void clear()
  {
    iterator itrBegin = begin();
    while(itrBegin != end())
    {
      list_node* tmp =  static_cast<list_node *>(itrBegin.m_smartPtr);

      ++itrBegin;

      if(tmp)
      {
        delete tmp;
      }
    }

    pHeadNode->Next = pHeadNode;
    pHeadNode->Prev = pHeadNode;
    _size = 0;
  }

  int size() // return length
  {
    return _size;
  }

private:
  void empty_init() 
  { 
    pHeadNode = new list_node_base();
    pHeadNode->Next = pHeadNode;  // Initialize pointer to itself
    pHeadNode->Prev = pHeadNode;

    _size = 0;
  }

private:
  list_node_base* pHeadNode; // List head

  unsigned int _size; // the number of nodes, increase the efficiency of searching
};


__MUNDI_END

#endif
```

## Def.h

```cpp
#define __MUNDI_BEGIN namespace Mundi {
#define __MUNDI_END }


#ifndef _T_STD_NOEXCEPT
# if __cplusplus >= 201103L
#  define _T_STD_NOEXCEPT noexcept
#  define _T_STD_USE_NOEXCEPT noexcept
#  define _T_STD_THROW(_EXC)
# else
#  define _T_STD_NOEXCEPT
#  define _T_STD_USE_NOEXCEPT throw()
#  define _T_STD_THROW(_EXC) throw(_EXC)
# endif
#endif
```

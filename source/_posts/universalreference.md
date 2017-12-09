title: C++ 中的 universal reference
date: 2015-11-07 21:34:04
categories: Effective Modern C++
tags: [reference, C++, universal reference, Effective Modern C++, ]
---

之前看类型推导时，看到了 universal reference 这个词，之前完全不知道这是啥意思，看到后面才知道，按照 Scott Meyers 书上所说，这样称呼的原因如下

*In fact, “T&&” has two different meanings. One is rvalue reference, of course. Such references behave exactly the way you expect: they bind only to rvalues, and their primary raison d’être is to identify objects that may be moved from. 
The other meaning for “T&&” is either rvalue reference or lvalue reference. Such references look like rvalue references in the source code (i.e., “T&&”), but they can behave as if they were lvalue references (i.e., “T&”). Their dual nature permits them to bind to rvalues (like rvalue references) as well as lvalues (like lvalue references). Furthermore, they can bind to const or non-const objects, to volatile or non-volatile objects, even to objects that are both const and volatile. They can bind to virtually anything. Such unprecedentedly flexible references deserve a name of their own. I call them universal references.1*
*1 Item 25 explains that universal references should almost always have std::forward applied to them, and as this book goes to press, some members of the C++ community have started referring to universal references as forwarding references.*
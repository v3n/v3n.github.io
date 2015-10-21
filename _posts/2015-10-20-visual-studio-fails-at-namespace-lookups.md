---
layout: post
title: "Visual Studio Fails At Namespace Lookups"
tags: [c++, bug, msvc]
share: true
---

I recently ran into an interesting Visual Studio bug. Namely, if you have nested namespaces and are making us of a `using` statement, it seems to mess up namespace lookup. 

{% highlight c++ %}
namespace A {
    struct X  {};

    namespace B {
        struct Y {
            void fun (X &);
        };
    }
}

using namespace A::B;

void Y::fun(X &) {}

int main () {
    A::B::Y y;
}
{% endhighlight %}

This code compiles fine with clang and gcc, but for whatever reason, doesn't compile under Visual Studio. You'll see errors similiar to:

{% highlight c++ %}
missing type specifier - int assumed. Note: C++ does not support default-int
syntax error: missing ',' before '&'
'void A::B::Y::fun(X &)': overloaded member function not found in 'A::B::Y'
{% endhighlight %}

It seems to overwrite the root namespace scoping (in this case, `A`, making `A::X` not found). This is fixable by explicitly writing all namespaces like such:

{% highlight c++ %}
namespace A {
    struct X  {};

    namespace B {
        struct Y {
            void fun (X);
        };
    }
}

void A::B::Y::fun(X) {}

int main () {
    A::B::Y y;
}
{% endhighlight %}

However, this can be a huge pain if you have a lot of references and can really clutter your code. Fortunately, there's another easy fix, redeclare the root namespace:

{% highlight c++ %}
using namespace A;
{% endhighlight %}

And that should fix it. Thanks to those over at [StackOverflow](http://stackoverflow.com/questions/33176677/mcvs-compilation-error-missing-before-fine-on-clang) who helped me pinpoint this issue. I've also submitting a [bug for the Visual Studio folks.](https://connect.microsoft.com/VisualStudio/feedback/details/1910037/visual-c-namespace-resolution-failure)

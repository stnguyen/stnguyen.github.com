---
layout: post
title:  "Call Java functions from C++ (and vice-versa)"
date:   2014-02-04 01:36:05
categories: cocos2d-x
comments: true
---

Calling Java functions from C++ (and vice-versa) for a cross-platform cocos2d-x game is actually not that hard (and scary, as I thought). Learning from cocos2d's `Java_org_cocos2dx_lib_Cocos2dxHelper.cpp` file and a bit of searching on the internet should give us all the information we might need.

_The following note actually applies for all native Android applications, not just cocos2d-x games._

## Call Java functions from C++

### Simplest case: void method with no arguments

Suppose you have a `static void doMeAFavour()` java function declared in `MyAwesomeJavaClass`, calling it from C++ is as simple as:

{% highlight c++ linenos %}
JniMethodInfo t;

if (JniHelper::getStaticMethodInfo(t,
  "MyAwesomeJavaClass",
  "doMeAFavour",
  "()V")) {

    t.env->CallStaticVoidMethod(t.classID, t.methodID);
    t.env->DeleteLocalRef(t.classID);
}
{% endhighlight %}


That's it! Yes, that simple!

But what about passing parameters and/or retriving a return value? We will need to learn a bit about _Java method signature_.

### Java method signature

Do you see `"()V"` part when we call `JniHelper::getStaticMethodInfo()`? It's called a _Java method signature_, or in my words is **a  string showing argument types and return type of a Java method**.

Rules:

    +---------------------------+------------------------+
    |         Signature         |        Java Type       |
    +---------------------------+------------------------+
    | Z                         |  boolean               |
    | B                         |  byte                  |
    | C                         |  char                  |
    | S                         |  short                 |
    | I                         |  int                   |
    | J                         |  long                  |
    | F                         |  float                 |
    | D                         |  double                |
    | L fully-qualified-class ; |  fully-qualified-class |
    | [ type                    |  type[]                |
    | ( arg-types ) ret-type    |  method type           |
    +---------------------------+------------------------+

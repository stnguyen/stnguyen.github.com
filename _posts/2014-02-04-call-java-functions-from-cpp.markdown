---
layout: post
title:  "Call Java functions from C++ in cocos2d-x"
date:   2014-02-04 01:36:05
categories: cocos2d-x
tags: cocos2d-x java c++ jni game-development
comments: true
---

Calling Java functions from C++ (and vice-versa) for a cross-platform cocos2d-x game is actually not that hard (and scary, as I thought). Learning from cocos2d's `Java_org_cocos2dx_lib_Cocos2dxHelper.cpp` file and a bit of searching on the internet should give us all the information we might need.

_The following note actually applies for all native Android applications, not just cocos2d-x games._

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

#### Rules:

|         Signature         |        Java Type       |
|---------------------------|------------------------|
| V                         |  void                  |
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


#### Examples:

{% highlight java %}

static String doMeAFavour(int times)
// Signature: "(I;)Ljava/lang/String;"

static bool sayHello(String to, int times)
// Signature: "(Ljava/lang/String;I;)Z;"

{% endhighlight %}

### Passing parameters and receiving return value

Suppose we have a Java method which takes 2 arguments and return a bool value

{% highlight java %}
static bool sayHello(String to, int times)
{% endhighlight %}

To call it from C++, do:

{% highlight c++ linenos %}

JniMethodInfo t;

if (JniHelper::getStaticMethodInfo(t,
  "MyAwesomeJavaClass",
  "sayHello",
  "(Ljava/lang/String;I;)Z;")) {

    const char* myName = "Beautiful Name";
    int times = 3;

    jstring stringArg1 = t.env->NewStringUTF(myName);
    jboolean retV = t.env->CallStaticBooleanMethod(t.classID,
        t.methodID,
        stringArg1,
        times);

    t.env->DeleteLocalRef(t.classID);
    t.env->DeleteLocalRef(stringArg1);
}

{% endhighlight %}

Explain:

- Pass any number of parameters into `CallStaticBooleanMethod` call.
- You may need to wrap paramters into JNI types as in _line 11_. If you did so, remember to delete local refence as in _line 18_.
- There's a [whole family](http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/functions.html#wp4815) of `CallStatic[ReturnType]Method` depends on the
    returning value type you need. Each method return a corresponding JNI type (e.g. `jboolean`, `jfloat`, `jobject`...)

#### Receiving return value of type String

Recieving a string from Java is a bit trickier (yeah, just a bit). String is an object in Java, so we're using `CallStaticObjectMethod` and convert the result to `std::string`:

{% highlight c++ linenos %}

jstring s = (jstring) t.env->CallObjectMethod(t.classID,
        t.methodID);

// convert it to std::string
std::string str = JniHelper::jstring2string(s);

{% endhighlight %}

_You might need to retain the String being returned from Java method, or else C++ will receive a pointer to a released space. The easiest way would be making the returning value static_

---

In part 2 of this tutorial, I will write about calling C++ methods from Java ;) Stay tuned...

__UPDATE__: [part 2 is here](./call-cpp-functions-from-java.html)

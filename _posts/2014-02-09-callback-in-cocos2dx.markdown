---
layout: post
title:  "Callback in cocos2d-x"
date:   2014-02-09 11:36:05
categories: cocos2d-x
tags: cocos2d-x game-development callback cccallfunc
comments: true
---

Callback, the ability to declare a piece of code to be run at some time later, is essential in game development. cocos2d provides an easy-to-use mechanism to run callback via `CCNode.runAction()` through `CCCallFunc` family. We can extend its mechanism to run callback any time we want, regardless of `CCNode` or `CCCallFunc`.

Callback in `CCNode.runAction()`
---

cocos2d-x (and cocos2d-iphone) provides a bunch of `CCAction` subclasses which allow you to do callback in `CCNode.runAction()`. They are:

- `CCCallFunc`: calls a function with no arguments
- `CCCallFuncN`: calls a function with the node as the first argument. `N` stands for _node_
- `CCCallFuncND`: calls a function with the node as the first argument and the 2nd argument is params.
  `ND` means: Node and Data. Data is `void *`, so it could be anything.
- `CCCallFuncO`: calls a function with an `CCObject *` as the first argument. `O` stands for object.

Some __examples with detailed explanation__:

{% highlight c++ %}

class MyBeautifulLayer
{
    void demoCallbackWithNoArgument()
    {
        // Use CCCallFunc when you don't need to pass any parameter along
        myNode->runAction(CCSequence::create(
            CCFadeOut::create(2),
            CCCallFunc::create(
                this,
                callfunc_selector(MyBeautifulLayer::sayHello1)),
        NULL));

        // If you need to access the object being called callback on, use
        // CCCallFuncN, CCCallFuncND or CCCallFuncO.
        // In this case we don't need to pass any parameter along, so let's
        // use CCCallFuncN
        myNode->runAction(CCCallFuncN::create(
            this, 
            callfuncN_selector(MyBeautifulLayer::sayHelloN)));
    }

    void demoCallbackWithArguments()
    {
        // Use CCCallFuncND or CCCallFuncO when you need to
        // pass some parameters

        // Any object pointer (or CCObject* in case of CCCallFuncO)
        // will work.
        // Here we use CCArray to pass an arbitary number of CCObject*
        // as parameters.
        CCArray* params = CCArray::create(myObject1, myObject2, NULL);

        params->retain();
        // IMPORTANT: Remember to call params->release() on
        // callback function, or else you have a memory leak

        // Use CCCallFuncND
        myNode->runAction(CCCallFuncND::create(
            this, 
            callfuncND_selector(MyBeautifulLayer::sayHelloND),
            (void*) params));

        // Or use CCCallFuncO
        myNode->runAction(CCCallFuncO::create(
            this, 
            callfuncO_selector(
                MyBeautifulLayer::sayHelloO),
                params));
    }

    void sayHello(){}
    void sayHelloN(CCNode* sender){}
    void sayHelloND(CCNode* sender, void* data){}
    void sayHelloO(CCNode* sender, CCObject* data){}
}
{% endhighlight %}

Callback outside of `CCNode.runAction()`
---

There're times when you need to do callback outside of `runAction()`, for example: callback on network response, callback when a long calculation finishes...

In these cases, you need to store 2 things:

- Pointer to the object being called function on, or _target object_
- Selector to the function being called

{% highlight c++ %}
// Pointers we need
cocos2d::CCObject* mTarget;
cocos2d::SEL_CallFunc mFunctionSelector;
{% endhighlight %}

And when you need to run callback:

{% highlight c++ %}
(mTarget->*mFunctionSelector)();
{% endhighlight %}

You can change `SEL_CallFunc` to `SEL_CallFuncN`, `SEL_CallFuncND` or `SEL_CallFuncO` to pass some parameters along.

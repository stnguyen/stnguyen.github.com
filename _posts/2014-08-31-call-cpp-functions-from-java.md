---
layout: post
title:  "Call C++ functions from Java in cocos2d-x"
date:   2014-08-31 01:36:05
categories: cocos2d-x
tags: cocos2d-x java c++ jni game-development
comments: true

---

If you've read [Part 1. Calling Java functions from C++ in cocos2d-x](./call-java-functions-from-cpp.html), you'll agree with me that the task is not as hard as it seems. Fortunately, the other way around, i.e., calling C++ functions from Java, is also quite straight forward.

_The following note actually applies for all native Android applications, not just cocos2d-x games._

### Simplest case: void method with no arguments
There're 2 steps to make a C++ function callable in Java code:

1. Declare a `native` method in Java code 
2. Name the C++ function follows a rule

#### 1. Declare a `native` method in Java code 

Suppose there's a `awesomeCppFunction()` you wanna call from Java, first you'll need to declare it as `native` method in your __Java class__:

	public static native void awesomeCppFunction();


Calling this method in Java is as normal as calling any ordinary Java method:

	package com.myhouse;
	
	class HappyJavaClass {
	
		public static native void awesomeCppFunction();

		private void myOrdinaryJavaMethod() {
			String a = "Yes, it's pure Java!";
			
			awesomeCppFunction();
		}
		
	}
	
#### 2. Name the C++ function follows a rule

C++ function name must contain Java full-path classname which declared its signature in step (1).

In the above example, your C++ function must be named `Java_com_myhouse_HappyJavaClass_awesomeCppFunction`.

It must be declared in `.cpp` file like so:

	extern "C"
	{
	JNIEXPORT void JNICALL Java_com_myhouse_HappyJavaClass_awesomeCppFunction(JNIEnv* env, jobject thiz);
	};

	JNIEXPORT void JNICALL Java_com_myhouse_HappyJavaClass_awesomeCppFunction(JNIEnv* env, jobject thiz)
	{
		// your C++ code goes here
	}

The first 2 arguments `JNIEnv* env, jobject thiz` are required and are used to extract data sent from Java to C++.

### Pass parameters and return value

On __Java side__, you declare method params and return type as usual:

	public static native String awesomeCppFunction(int x, boolean y, String z);

On __C++ side__, you'll need to "convert" Java types to JNI (native) types.

|         JNI Types         |        Java Type          |
|---------------------------|---------------------------|
| void                      |  void                     |
| jboolean                  |  boolean                  |
| jbyte                 	 |  byte                     |
| jchar            	    	 |  char                     |
| jshort           		     |  short                    |
| jint                 		 |  int                      |
| jlong               		 |  long                     |
| jfloat               		 |  float                    |
| jdouble                	 |  double                   |
| jobject                   |  All Java objects         |
| jclass                    |  java.lang.Class objects  |
| jstring                   |  java.lang.String objects |
| jobjectArray              |  Array of objects         |
| jbooleanArray             |  Array of booleans        |
| jbyteArray                |  Array of bytes           |
| jshortArray               |  Array of shorts	         |
| jintArray                 |  Array of integers        |
| jlongArray                |  Array of longs           |
| jfloatArray               |  Array of floats          |
| jdoubleArray              |  Array of doubles         |

Add parameters after the 2 required params:

	extern "C"
	{
	JNIEXPORT jstring JNICALL Java_com_myhouse_HappyJavaClass_awesomeCppFunction(JNIEnv* env, jobject thiz, jint myIntParam, jstring myStringParam);
	};

	JNIEXPORT jstring JNICALL Java_com_myhouse_HappyJavaClass_awesomeCppFunction(JNIEnv* env, jobject thiz, jint myIntParam, jstring myStringParam)
	{
		// your C++ code goes here
	}
	
#### Use primitive type parameters

Primitive data (e.g., jboolean, jint, jchar, jfloat...) can be used as normal C++ equivalent data types:

	JNIEXPORT void JNICALL Java_com_myhouse_HappyJavaClass_awesomeCppFunction(JNIEnv* env, jobject thiz, jint myIntParam)
	{
		int myCppVariable = myIntParam;
	}
	
#### Use object parameters

Objects passed as parameters need to be converted to C++ equivalent data types:

Examples:

##### String

	JNIEXPORT void JNICALL Java_com_myhouse_HappyJavaClass_awesomeCppFunction(JNIEnv* env, jobject thiz, jstring myStringParam)
	{
  		  const char* str = env->GetStringUTFChars(myStringParam, NULL);
  		  
  		  // use the value
  		  
 		  env->ReleaseStringUTFChars(myStringParam, str);
	}

##### Array of integers:

_Example from [here](http://www.owsiak.org/?p=1514)_:

	JNIEXPORT void JNICALL Java_Simple_passArray(JNIEnv *env, jobject obj, jintArray ptr)
	{
	    int i;

	    jsize len = env->GetArrayLength(env, ptr);
	    jint *body = env->GetIntArrayElements(env, ptr, 0);
	  
	    for (i=0; i < len; i++)
	        printf("Hello from JNI - element: %d\n", body[i]);
	     
	    env->ReleaseIntArrayElements(env, ptr, body, 0);
	}
	
##### Array of Strings:

_Example from [here](http://www.owsiak.org/?p=1514)_:

	JNIEXPORT void JNICALL Java_Simple_passArray(JNIEnv *env, jobject obj, jstringArray ptr)
	{
	    int i;

	    jsize len = env->GetArrayLength(env, ptr);
	    jstring *body = env->GetObjectArrayElement(env, ptr, 0);
	  
	    for (i=0; i < len; i++)
	    {
            const char *rawString = env->GetStringUTFChars(body[i], false);
	        printf("Hello from JNI - element: %s\n", rawString);
	        
	        env->ReleaseStringUTFChars(body[i], rawString);
	    }
	    
	    env->ReleaseObjectArrayElements(env, ptr, body, 0);
	}

#### Return primitive data

Returning primitive data (e.g., jboolean, jint, jchar, jfloat...) is just as you normally do in C++:

	JNIEXPORT jint JNICALL Java_com_myhouse_HappyJavaClass_awesomeCppFunction(JNIEnv* env, jobject thiz)
	{
		int i = 3;
		
		return i;
	}
	
#### Return object

To return an object, you'll need to convert your C++ object to equivalent JNI object.

Examples:

##### String

	JNIEXPORT jstring JNICALL Java_com_myhouse_HappyJavaClass_awesomeCppFunction(JNIEnv* env, jobject thiz)
	{
		jstring result = env->NewStringUTF(FacebookManager::GetInstance()->GetWhitelistDevIds());
		env->DeleteLocalRef(result);
		return result;
	}
	
##### Array of integers

_Example extracted from [this stackoverflow answer](http://stackoverflow.com/questions/1610045/how-to-return-an-array-from-jni-to-java)_:

	JNIEXPORT jintArray JNICALL Java_com_myhouse_HappyJavaClass_awesomeCppFunction(JNIEnv* env, jobject thiz)
	{
		jintArray result;
		result = env->NewIntArray(env, size);
		if (result == NULL) {
		    return NULL; /* out of memory error thrown */
		}
		
		int i;
		// fill a temp structure to use to populate the java int array
		jint fill[256];
		for (i = 0; i < size; i++) {
		    fill[i] = 0; // put whatever logic you want to populate the values here.
		}
		// move from the temp structure to the java structure
		env->SetIntArrayRegion(env, result, 0, size, fill);
	    return result;
	}
	
##### Array of Strings

_Example extracted from [this stackoverflow answer](http://stackoverflow.com/questions/1610045/how-to-return-an-array-from-jni-to-java)_:

	JNIEXPORT jobjectArray JNICALL Java_com_myhouse_HappyJavaClass_awesomeCppFunction(JNIEnv* env, jobject thiz)
	{
		jobjectArray ret;  
	    int i;  
	
	    char *message[5]= {"first",   
	                       "second",   
	                       "third",   
	                       "fourth",   
	                       "fifth"};  
	
	    ret= (jobjectArray)env->NewObjectArray(5,  
	         env->FindClass("java/lang/String"),  
	         env->NewStringUTF(""));  
	
	    for(i=0;i<5;i++) {  
	        env->SetObjectArrayElement(  
	        ret,i,env->NewStringUTF(message[i]));  
	    }  
	    return(ret);  
	}

---

That's it! Happy coding :D
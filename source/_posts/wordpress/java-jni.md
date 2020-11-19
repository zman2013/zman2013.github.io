---
title: Java JNI
tags:
  - C调用Java
  - JNI
id: '119'
categories:
  - - Java
date: 2015-11-22 22:39:20
---

_介绍C++调用Java JNI例子和原理_ **简单的步骤**

1.  完成普通的Java类，没有限制，没有规则。如果已经有了现成的类，就直接拿来用吧。
2.  编译Java代码，没有限制，没有规则。
3.  开发C++代码: 初始化JVM，通过Java JNI Functions调用Java逻辑。
4.  编译、运行C++ 应用。

**例子（[源码](https://github.com/zman2013/java-jni-demo.git)）** **1.** 一个简单的Java类

package zman.demo.jni.cpp.invoke.java;

public class CppInvokeJava{
    //求和计算
    public int sum(int a, int b, int c) {
        return a + b + c;
    }

}

**2.** 编译一下

zman@zman:find . -name "\*.java"  xargs javac -d build

\-d build: 指定class输出路径。

**3.** 完成C++代码 **3.1** 声明一个类CppInvokeJava，文件名：cppinvokejava.h

#ifndef \_\_CPP\_INVOKE\_JAVA\_H\_\_
#define \_\_CPP\_INVOKE\_JAVA\_H\_\_

#include <jni.h>

class CppInvokeJava
{
public:
  //init nested jvm, and init java instance.
  CppInvokeJava();
  //destroy jvm
  ~CppInvokeJava();
  //normal method
  int Sum(int a, int b, int c);

private:
  JavaVM \*m\_jvm;
  JNIEnv \*m\_env;
  //the java class of CppInvokeJava
  jclass m\_cppinvokejava\_class;
  //the java instace of the class CppInvokeJava
  jobject m\_cppinvokejava;
  //the java method id of the java sum method which will be invoked by the above 'sum' method.
  jmethodID m\_sum\_method;
};

#endif

**3.2** 完成相应的实现逻辑，文件名：cppinvokejava.cpp

#include <jni.h>
#include <string.h>
#include "cppinvokejava.h"

CppInvokeJava::CppInvokeJava()
{
  //1. prepare jvm args
  JavaVMInitArgs vm\_args;
  vm\_args.version = JNI\_VERSION\_1\_8;

  JavaVMOption options\[1\];
  options\[0\].optionString = "-Djava.class.path=build";
  vm\_args.nOptions = 1;
  vm\_args.options = options;

  //2. create jvm
  long jvmStatus = JNI\_CreateJavaVM(&m\_jvm, (void \*\*)&m\_env, &vm\_args);
  if( jvmStatus == JNI\_ERR ){
    printf("Creating JVM failed.\\n");
  }

  printf("Created JVM successfully.\\n");

  //3. find the CppInvokeJava class
  m\_cppinvokejava\_class = m\_env->FindClass("zman/demo/jni/cpp/invoke/java/CppInvokeJava");
  if( m\_cppinvokejava\_class == 0 ){
    printf("Not found class CppInvokeJava.\\n");
  }
  //4. get the constructor method id
  jmethodID cppinvokejava\_constructor = m\_env->GetMethodID(m\_cppinvokejava\_class, "<init>", "()V");
  if( cppinvokejava\_constructor == 0 ){
    printf("Not found CppInvokeJava's constructor.\\n");
  }
  //5. create CppInvokeJava instance
  m\_cppinvokejava = m\_env->NewObject(m\_cppinvokejava\_class, cppinvokejava\_constructor);
  if( m\_cppinvokejava == 0 ){
    printf("Creating CppInvokeJava instance failed.\\n");
  }

  //6. find the sum method id
  m\_sum\_method = m\_env->GetMethodID(m\_cppinvokejava\_class, "sum", "(III)I");
  if( m\_sum\_method == 0 ){
    printf("Not found sum method.\\n");
  }

}

CppInvokeJava::~CppInvokeJava()
{
  m\_jvm->DestroyJavaVM();
}

int CppInvokeJava::Sum(int a, int b, int c)
{
  return m\_env->CallIntMethod(m\_cppinvokejava, m\_sum\_method, a, b, c);
}


int main()
{
  CppInvokeJava cppInvokeJava;
  int sum = cppInvokeJava.Sum(1, 2, 3);
  printf("The sum is: %d.\\n", sum);
}

1.  准备虚拟机参数
2.  创建虚拟机
3.  调用JNI FindClass方法找到CppInvokeJava class
4.  调用JNI GetMethodID方法找到CppInvokeJava的构造方法
5.  调用JNI NewObject方法创建CppInvokeJava对象
6.  调用JNI GetMethodID方法找到CppInvokeJava的sum方法

**4.** 编译、运行 **4.1** 编译

//Linux 版
zman@zman:g++ -o cppinvokejava src/main/java/zman/demo/jni/cpp/invoke/java/cppinvokejava.cpp -I ${JAVA\_HOME}/include/ -I ${JAVA\_HOME}/include/linux/ -L ${JAVA\_HOME}/jre/lib/amd64/server/ -ljvm

//Mac 版
$ g++ -o cppinvokejava src/main/java/zman/demo/jni/cpp/invoke/java/cppinvokejava.cpp -I ${JAVA\_HOME}/include/ -I ${JAVA\_HOME}/include/darwin/ -L ${JAVA\_HOME}/jre/lib/server/ -ljvm
//JDK>1.6时，Mac运行JNI会提示需要安装JDK6，可以通过一下命令解决。
$ sudo mkdir /System/Library/Java/JavaVirtualMachines/1.6.0.jdk/
$ sudo mkdir /System/Library/Java/Support/Deploy.bundle/

编译cppinvokejava.cpp，输出到cppinvokejava。

\-I：指定头文件的位置（jni.h等）

\-L：指定动态链接库的位置（libjvm.so）

\-l：指定动态链接库名称（jvm）

**4.2** 指定动态链接库环境变量

zman@zman:export LD\_LIBRARY\_PATH=${JAVA\_HOME}/jre/lib/amd64/server/

**4.3** 运行

zman@zman:java-jni-demo$ ./cppinvokejava 
Created JVM successfully.
The sum is: 6.
zman@zman:java-jni-demo$

**原理** 正如我们一直了解的，java的字节码是在JVM中加载执行的。C++调用java，事实上是启动一个JVM，java的字节码依然是运行在JVM中，C++通过Java JNI提供的方法与JVM进行通信，达到调用java逻辑的目的。 注意 **\*** 对于java对象，C++代码获得的都是引用，C++需要通过JNI Function调用对象提供的方法实现对象的修改。 **\*** C++不能直接获得java String字符串，需要通过GetStringUTFRegion将字符串内容复制到C++的空间中；也可以通过SetStringUTFRegion将C++中的字符串赋值给java String对象。 **\*** 数组操作与String操作基本相同。 **\*** C++中的局部变量如果是java对象的引用，C++方法运行结束时，局部变量自动释放，而相应java对象由JVM自动回收。（如果C++使用完java对象，后续有比较耗时的逻辑，可以调用JNI Function主动释放java对象的引用，不必等到方法结束） **调试** 1. 调试Java程序 设置JVM参数如下，然后在IDE中启动Remote Debug模式:

JavaVMOption options\[2\];
options\[0\].optionString = "-Djava.class.path=build";
//添加debug参数
options\[1\].optionString = "-agentlib:jdwp=transport=dt\_socket,address=8000,server=y,suspend=y";
vm\_args.nOptions = 2;
vm\_args.options = options;

2\. 调试C++程序

zman@zman: gdb ./cppinvokejava     //调试程序
(gdb) r        //启动
(gdb) c        //继续执行
(gdb> info s   //查看stack信息

**内存泄露** 1. 创建字符串引起内存泄露 实际开发中，在本地方法里，创建字符串并获得本地引用（LocalReference），此本地引用不会返回到上层函数，依然会引起引起内存泄露。 这一现象和JNI Guide and Specification第五章“Local and Global References”描述的机制不一样。 _Local and global references have different lifetimes. Local references are_ _automatically freed, whereas global and weak global references remain valid_ _until they are freed by the programmer._ _It is acceptable to leave up to 16 local references in use for the virtual machine_ _to delete after the native method returns._ 推荐：无论何时字符串对象不会再被引用，就手工释放引用。

void createString()
{
    jstring localRef = env->NewStringUTF("some content");
    ...
    env->DeleteLocalRef( localRef );
    ...
}

**[LINK](https://www.zmannotes.com/index.php/2015/11/22/java-jni/)** 1. [JNI Guide from Oracle](https://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/jniTOC.html) 2. [Java programming with JNI](http://www.ibm.com/developerworks/java/tutorials/j-jni/j-jni.html) 3. [JNI Guide and Specification](http://www.e-booksdirectory.com/details.php?ebook=325)
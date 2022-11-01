---



layout:     post
title:      Unity'IEnumerator and lambda perform in C++ 
subtitle:   unity协程迭代器和匿名方法在c++转译后的执行方式
date:       2022-11-01
author:     俊壳
header-img: img/tag-bg-o.jpg
catalog: true
tags:
    - unitiy
---

# 协程的作用

> 协程，即Coroutine，可以认为它是一个返回值是IEnumerator的函数，但是使用它需要配合StartCoroutine使用。
协程不是多线程，所有的任务还是在主线程上完成，是一种异步多任务处理的方式。
协程的好处在于，它可以在一个函数内实现像update那样按帧执行某个部分，或者按照过几秒后执行某个部分。

但是问题也来了，协程并没有办法提升整体的效率，所以把它放在一边用`while(条件)  return null;`是不明智的，这个时候用多线程或者unity提供的jobSystem更为恰当
## 问题剖析

协程在unity时如何实现：
为了能进一步思考这个问题，首先我们需要写出我们的代码，并把让c#的unity项目打包成输出成c++的模式
```c#
private void Start（）{

    startCoroutine(DoSomeThing())



}

IEnumerator DoSomething(){

    //意义不明的一行代码A(随便写些什么)
    //意义不明的一行代码B
    int i=0;
    yield return null;
    Debug.log("hero");
    while(ture){
        string a="";
        a+=i.ToString()

    }
}
```
>其实就是很简单的协程代码,写好这些就可以导出项目

![image]({{ "/assets/unity/1.png" | absolute_url }})

projectSetting-》Player-》configuration-》scriptBackend-》il2cpp 把mono的模式改成cpp
然后build，勾选上 create visual studio solution 和devolopment build 这样编译时就生成了对应c++的解决方案，一切就绪后，用visual studio打开生成的c++解决方案
![image]({{ "/assets/unity/2.png" | absolute_url }})

然后我们就需要搜索对应的协程方法名，找到具体实现，实现依赖一堆地址，所以我们多搜索几下

![image]({{ "/assets/unity/3.png" | absolute_url }})
![image]({{ "/assets/unity/4.png" | absolute_url }})
然后我们发现这里的迭代时用的比较lowwer的一种方法实现，goto,并且讲迭代器中用到变量都存在一个地址中，而非局部变量，所有实际上在协程/迭代器执行过程中，迭代器会在内存里常驻，而并没有消除，除非它使用完进行销毁，所以对一些大内存的协程等待我们需要考虑慎重


# 匿名方法的使用
同理，匿名方法也是一样的，我们写下如下代码
```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class newScript : MonoBehaviour
{


    public delegate void Delete2();
    public Delete2 my1;
    public Delete2 my2;
    // Start is called before the first frame update
    void Start()
    {
        testDelegateLambda();
        StartCoroutine(myProcess());
    }



    public IEnumerator myProcess() {

        yield return new WaitForSeconds(1);
        my2();


    }


    public void testDelegateLambda() {

        int outsideJ = 0;
        my2 = () =>
        {

            outsideJ++;
            print(outsideJ);
        };
    
    }

    // Update is called once per frame
    void Update()
    {
        
    }
}

```
我们在协程里调用了m2委托，而m2是一个匿名函数，并使用了`testDelegateLambda`里的外部`outsideJ`参数，按道理来说参数在testDelegateLambda函数外就消失了，m2本身应该无法在myProcess调用，但这块调用一切正常，但为什么可以正常调用而不会报错呢？我们也来看看c++的实现


![image]({{ "/assets/unity/4.png" | absolute_url }})
我们会发现，它会也把`outsideJ`这个参数声明成一个委托地址，然后调用时，将地址作为参数传入到新的委托，之后再调用新委托的地址，这样外部也同样可以使用这个参数了
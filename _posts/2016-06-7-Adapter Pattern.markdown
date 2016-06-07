---
layout: post
title: "适配器模式"
subtitle: "每一个模式描述了一个在外面周围不断重复发生的问题，以及该问题的解决方案的核心。这样，你就能一次又一次地使用该方案而不必做重复劳动。"
subtitleAuthor: "——克里斯托弗·亚历山大"
date: 2016-06-07
author: LuJiangBo
category: Software design pattern
tags: 设计模式
finished: true
---


## 意图

&emsp;&emsp;将一个类的接口转换成客户希望的另外一个接口。Adapter模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

## 分类

* 类适配器
* 对象适配器

## 适用性 
下列情况可以考虑使用适配器模式: 

* 你想使用一个已经存在的类，而它的接口不符合你的需求。  
* 你想创建一个可复用的类，该类可以与其他不相关的类或不可预见的类（即那些接口可能不一定兼容懂类)一起工作。
* (仅试用于对象Adapter)你想使用一些已经存在的子类，但是不可能对每一个都进行子类化以匹配它们的接口。对象适配器可以适配它的父类接口。

## 结构 
对象适配器的结构图下图所示。  
	![对象适配器]({{ post.url| prepend: site.url  }}/content/images/201606/2016-06-07-Adapter01.png) 

## 参与者
* Target: (目标接口)  
	－定义Client使用的接口。
* ClassA/ClassB: (实现类)   
	－一组对Target具体实现的类。  
* Adapter: (适配器)  
	－通过包装被适配类Adaptee，把原Adaptee接口转成目标接口。
* Adaptee: (需要适配的类)
    －需要被适配的类。

## 代码实现  
{% highlight c# %}
/// <summary>
/// 定义Client使用的接口类
/// </summary>
public interface Target{
    void Request();
}
/// <summary>
/// 对Target具体实现的类
/// </summary>
public class ClassA:Target
{
    public void Request(){
        //...logic
    }
}
/// <summary>
/// 对Target具体实现的类
/// </summary>
public class ClassB:Target
{
    public void Request(){
        //...logic
    }
}
/// <summary>
/// 需要被适配的类
/// </summary>
public class Adaptee
{
    public void SpecialRequest(){
        //...logic
    }
}

/// <summary>
/// 适配器类
/// </summary>
public class Adapter:Target
{
    private Adaptee _adaptee;
    public Adapter(){
        this._adaptee=new Adaptee();
    }
    public void Request(){
        _adaptee.SpecialRequest();
    }
} 
{% endhighlight %}

## 客户端调用  
{% highlight c# %}
class Program
    {
        static void Main(string[] args)
        {
            // 通过Adapter适配器得到统一的Target对象，同时调用了Adaptee中的方法
            Target target = new Adapter();
            target.Request();

        }
    }
{% endhighlight %}

## 类适配器
&emsp;&emsp;类适配器与对象适配器的区别在于类适配器使用多重继承对一个接口与另一个接口进行匹配。在c#、java等不支持类多继承的语言中，我们可以通过对接口的多继承在实现类适配器。这里给出UML图。
    ![类适配器]({{ post.url| prepend: site.url  }}/content/images/201606/2016-06-07-Adapter02.png)

## 模式优点  
* 复用现有类，解决了它的接口与当前环境不一致的问题。
* 将目标类和适配者类解耦，使目标类不需要依赖适配者类。
* 客户端实现了统一的接口调用，不需要为了某个适配者类单独做不同的对象创建。
* [类适配器]使得Adapter可以重定义Adaptee的部分行为，因为Adapter是Adaptee的子类。
* [类适配器]仅仅引入了一个对象，并不需要额外的指针以间接得到Adaptee。
* [对象适配器]允许一个Adapter与多个Adaptee(Adaptee本事和它的所有子类)同时工作.

## 模式缺点  
* [对象适配器]使得重定义Adaptee的行为比较困难，这需要创建Adaptee的子类，并且让Adapter引用这个子类而不是Adaptee本身。
* [类适配器]当我们想要匹配一个类以及所有它的子类时，类Adapter将不能胜任。








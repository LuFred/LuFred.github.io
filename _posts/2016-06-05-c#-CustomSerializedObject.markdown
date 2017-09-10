---
layout: post
title: "自定义可序列化对象"
subtitle: "自定义可序列化对象"
date: 2016-06-05
author: LuJiangBo
tags: DotNet
keyword: 
finished: true
---

## 定义  

 &emsp;&emsp;有些时候我们序列化一个自定义的类时并不想将它的所有成员都序列化，比如下面例子中A类中的count字段，它的值是由另外2个字段的值相加操作得出，既然是这样就没有必要序列化这个对象，只要在反序列化之后再加一次就行，这样就减少了序列化后对象的尺寸(同时降低在将序列化后的对象写如磁盘上时的存储需求和在网络上进行传送时对带宽的需求）

## 原理    
要做到这个效果只需对在类成员count使用[NonSerialized]特性，如下所示：
{% highlight c# %}

[NonSerialized]
public Decimal count;

{% endhighlight c# %}

&emsp;&emsp;若想让类自动初始化一个在序列化时被忽略的成员，使用IDeserializationCallback接口，运行时在每次反序列化完成后调用IDeserializationCallback.OnDeserialization方法。
 
## 实现  
知道原理了代码就很简单了.     
{% highlight Sql Server %}

using System;
using System.Collections.Generic;
using System.Linq;
using System.Runtime.Serialization.Formatters.Binary;
using System.Text;
using System.Threading.Tasks;
using System.IO;
using System.Runtime.Serialization;
namespace 序列化
{
    [Serializable]//必须声明该类型可以被序列化
    class A : IDeserializationCallback//添加IDeserializationCallback实现OnDeserialization方法为未序列化的对象操作初始化
    {
        public String name;
        public Decimal d1;
        public Decimal d2;
        [NonSerialized]
        public Decimal count;
        public A(String _name, Decimal _d1, Decimal _d2)
        {
            name = _name;
            d1 = _d1;
            d2 = _d2;
            count = d1 + d2;
        }
        public void OnDeserialization(object sender)
        {
            //反序列化时调用该方法为不能被序列化的count字段赋值
            count = d1 + d2;
        }
    }
    class Program
    {
        static void Main(string[] args)
        {
            //序列化：
            //创建需要被序列化的对象
            A a = new A("张三",100,200);
            BinaryFormatter bf = new BinaryFormatter();//创建BinaryFormatter
            FileStream fs = new FileStream("1.txt", FileMode.Create, FileAccess.Write);//创建用于保存序列化后的输出的流
            bf.Serialize(fs, a);//调用Serialize方法序列化对象，并将字节写入流
            fs.Close();//关闭流
            //反序列化：
            //创建用于接收反序列化后的数据的对象
            A b;
            FileStream fs1 = new FileStream("1.txt", FileMode.Open, FileAccess.Read);//创建用于读取序列化输出的流
            BinaryFormatter bf1 = new BinaryFormatter();//创建BinaryFormatter
            b = (A)bf1.Deserialize(fs1);//调用Deserialize方法反序列化并返回给对象
            fs1.Close();//关闭流对象
            Decimal d = b.count;
            
        }
    }
}

{% endhighlight %}








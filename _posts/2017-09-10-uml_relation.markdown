---
layout: post
title: "UML表达关系术语"
subtitle: "软件工程学习笔记之UML各类事物之间的关系"
date: 2017-09-10
author: LuJiangBo
tags: 软件工程
keyword: 软件工程,UML
finished: true
---


##  介绍
　在UML的世界中，有4种用于表达各类事物之间的相互依赖和作用的术语，它们是关联(association)、泛化(generalization)、细化(realization)、依赖(dependency)。通过它们可以表达出类之间各种具有特定语义的关系，构造一个结构良好的UML模型。

## 关联(Association)  
* 说明：关联是类之间的一种结构关系，是对一组具有相同结构、相同链(links)的描述。   
    * 链是对象之间具有特定语义关系的抽象，实现之后的链通常称为对象之间的连接(connection)
* 表示：用一条连接两个类目的线段表示，并可对其命名，如下图1-1。  
![uml]({{ post.url| prepend: site.url  }}/content/images/201709/uml_relation_1_1.png)  
工厂类和汽车类关联，工厂制造汽车。   
一个只连接2个类目的关联称为二元关联；如果说一个关联连接n个类目，则称为n元关联。下图演示了一个3元关联。
![uml]({{ post.url| prepend: site.url  }}/content/images/201709/uml_relation_1_2.png)  

### 聚合(Aggregation)
* 说明：聚合是关联的一种特殊形式，表达的是一种‘整体/部分’关系。  
* 表示：表示为带有空心菱形的线段，其中空心菱形在整体类的那一边，如图1-3所示。
![uml]({{ post.url| prepend: site.url  }}/content/images/201709/uml_relation_1_3.png)  

### 组合(Composition)
* 说明：组合又是聚合的一种特殊形式。若在一段时间内，整体类的实例中至少包含一个部分类的实例，且该整体类的实例负责创建和消除部分类的实例，特别是它们拥有相同的生命周期，那么这样的聚合称为组合。  
* 在一个组合中，组合末端的多重性显然不能超过1
* 在一个组合中，由一个链所连接的对象而构成的任何元组，必须同属于同一个整体类的对象。  

* 表示：组合有3中表示方法    
    * 将部分类的名字直接放到整体类的属性栏中  
    * 将部分类放到整体类的一个栏目中  
    * 使用带有实心菱形的线段     

这里我们给出第三种方式的事例图，用房子和门窗之间的组合关系。
![uml]({{ post.url| prepend: site.url  }}/content/images/201709/uml_relation_1_4.png) 

## 泛化(Generalization)
* 说明：泛化是一般性类目(称为超类或父类)与和它的较为特殊性类目(称为子类)之间的一种关系，有时也称为“is-a-kind-of”关系。如果两个类具有泛化关系，那么  
    * 子类可以继承父类的属性和操作，并可有更多的属性和操作
    * 子类可以替换父类的声明
    * 若子类的一个操作实现覆盖了父类同一个操作的实现，这种情况被称为操作多态性，但是两个操作必须具有相同的名字和参数
    * 可以在其他类目之间创建泛化，例如在节点之间、接口和类之间等  
* 表示：表示成从子类到父类的一条带空心三角形的线段，其中三角形在父类一端。  
下图1-5表示了一个多父类的uml图。
![uml]({{ post.url| prepend: site.url  }}/content/images/201709/uml_relation_1_5.png) 

## 细化(Realization)
* 说明：细化是类目之间的语义关系，其中一个类目规约保证另一个类目执行的契约。  
* 表示：表示为一个带空心三角形的虚线段
* 应用：  
    * 接口与实现它们的类和构建之间
    * 用况与实现它们的协作之间    

下图1-6是一个简单的细化事例。
![uml]({{ post.url| prepend: site.url  }}/content/images/201709/uml_relation_1_6.png) 

## 依赖(Dependency)
* 说明：依赖是一种使用关系，用于描述一个类目使用另一个类目的信息和服务。例如，一个类使用了另一个类的操作，在这种情况下，如果被使用的类发生了变化，那么另一个类的操作也会受到影响。  
* 表示：表示为一条有向虚线段，其中箭头那一端的类目称为目标，而把另一端的类目称为源。
下图1-7描绘了一个简单的依赖关系，做菜需要依赖厨具
![uml]({{ post.url| prepend: site.url  }}/content/images/201709/uml_relation_1_7.png) 
 


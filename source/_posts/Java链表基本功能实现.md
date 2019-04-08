---
title: Java链表基本功能实现
date: 2019-03-05 10:13:02
tags:
---

## 链表
链表（Linked list）是一种常见的基础数据结构，是一种线性表，但是并不会按线性的顺序存储数据，而是在每一个节点里存到下一个节点的指针(Pointer)。由于不必须按顺序存储，链表在插入的时候可以达到O(1)的复杂度，比另一种线性表顺序表快得多，但是查找一个节点或者访问特定编号的节点则需要O(n)的时间，而顺序表相应的时间复杂度分别是O(logn)和O(1)。

使用链表结构可以克服数组链表需要预先知道数据大小的缺点，链表结构可以充分利用计算机内存空间，实现灵活的内存动态管理。但是链表失去了数组随机读取的优点，同时链表由于增加了结点的指针域，空间开销比较大。

## 接口
一个基本的Java链表主要有一下的功能:
### add
首先,新建一个List类,以及一个Node类,通过List类来调用方法
```java
public class List{
    private Node root;//定义链表的根节点
    private int count = 0;//定义当前列表节点个数
}

public class Node{
    private String data;//这是一个数据类型是 String 的链表
    private Node next; //每个Node类需要一个指向当前Node的下一个Node,才行形成链关系
    
    pulic Node(String data){
        this.data = data;
    }
    
    public Node getNext() {
        return next;
    }
    
    public String getData() {
        return data;
    }
}
```
定义完基本数据类,我们来实现add方法
在List中定义`add`
```java
public void add(String data){
    Node newNode = new Node(data);
    if(this.root == null){
        this.root = newNode;
    } else{
        this.root.addNode(newNode,this); //我们需要传入当前list对象,用于新增节点后跟新count
    }
}
 public void addCount() {
    this.count++;
}
```
Node类中实现`addNode`,它接受一个Node类型参数
```java
public void addNode(Node newNode,List list){
    if(!this.equals(newNode)){
        if(this.next == null){
                this.next = newNode;
                list.addCount();
            } else {
                //如果当前下个节点不为空,则让next继续调用addNode方法
                this.next.addNode(newNode,list);
            }
    }
}
//为了做到不重复新增数据,我们需要在Node类中实现一个equals方法

public boolean equals(Node node){
    if(this.data == node.data){
        return true;
    }
    return false;
}

```
### get
get方法接受一个int类型的序号,用于找到当前类型对应位置的节点数据,在List类中定义`get`
```java
public String get(int index) {
    if (this.root == null) return null;
    return this.root.get(index, 0);
}
```
Node类中定义`getNode`方法,它除了接受`index`参数外,还接受一个步进参数`step`,用于匹配index是否等于当前`step`
```java
public String get(int index, int step) {
    if (index == step) return this.data;
    return this.next.get(index, ++step);
}
```
### set
`set`方法用于重新设置链表对应位置节点的数据,
```java
public void set(int index, String data) {
    if (this.root == null || index < 0 || index > this.count) return;
    this.root.setNode(index, data, 0);
}
```

### contains
`contains`方法返回一个`Boolean`,判断当前列表是否存在对应的数据
```java
//List
public boolean contains(String data) {
    if (this.root == null) return false;
    return this.root.containNode(data);
}
```
在Node类中定义`containNode`方法
```java
//Node
public boolean containNode(String data) {
    if (this.data.equals(data)) {
        return true;
    } else {
        if (this.next == null) return false;
        return this.next.containNode(data);
    }
}
```
### size
返回当前链表节点个数
```java
//List
public int size() {
    return this.count;
}
```

### isEmpty
判断当前链表是否为空
```java
public boolean isEmpty() {
    return this.count == 0;
}
```
### remove
删除节点数据
```java
//List
public void remove(String data) {
    if (this.root == null) return;
    if (this.root.equals(data)) { //根节点特殊处理
        this.root = this.root.getNext();
        this.count--;
    } else {
        this.root.removeNode(this.root, data,this);
    }
}

//同add操作,还需要另外定义一个reduceCount方法供Node实例调用
public void reduceCount() {
    this.count--;
}
```
Node类中定义removeNode
```java
//Node
public void removeNode(Node pre, String data, List list) {
    if (this.data.equals(data)) {
        pre.next = this.next;
        list.reduceCount();
    } else {
        if(this.next != null)
            this.next.removeNode(this, data,list);
        }
    }
```

### toArray
将当前链表输出为`array`数组对象
```java
//List 新增一个私有属性arrayList
private String[] arrayList;

publice String[] toArray(){
    this.arrayList = new String[this.count];
    this.root.toArrayNode(arrayList,0);
    return this.arrayList;
}
```
Node类中定义`toArrayNode`方法
```java
//Node
public void toArrayNode(String [] array,int step){
    array[step] = this.data;
    if(this.next != null){
        this.next.toArrayNode(array,++step);
    }
}
```
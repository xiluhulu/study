## 顺序表

是一个相似于数组的一种结构，但内涵长度。分静态动态两种。

- 静态分配顺序表：

  - 大小固定，不可重新分配内存。

  - ```Java
    //静态顺序表的实现
    class SqList<E> {
        private int Max;
        private int length;
        private E[] data;
    
        public SqList(int max) {
            Max = max;
            this.length = 0;
            this.data =  (E[])new Object[max];
        }
    //    添加操作
        public void add(){
            
        }
    //    获取指定下标数据
        public E get(int index){
            return data[index];
        }
    //    删除指定下标数据
        public void remove(int index){
            
        }
    //    获取长度
        public int length(){
            return length;
        }
    //    判断是否为空
        public boolean isEmpty(){
            return length==0;
        }
    }
    ```

- 动态分配顺序表：

  - 大小可自定义增加，可重新分配内存；但是也不算动态开辟内存，而是新开比原来大的一个数组，再将原数组数据复制进去删除原数组实现的大小增加。

  - ```Java
    //动态顺序表的实现
    class SqList2<E>{
        private int initSize;
        private int size;
        private E[] data;
        public SqList2(int initSize) {
            this.initSize = initSize;
            this.size = 0;
            this.data = (E[]) new Object[this.initSize];
        }
        public void addSize(int size){
            E[] d = (E[])new Object[initSize+size];
            for(int i=0;i<size;i++){
                d[i]=this.data[i];
            }
            this.data=d;
        }
        //    添加操作
        public void add(){
    
        }
        //    获取指定下标数据
        public E get(int index){
            return data[index];
        }
        //    删除指定下标数据
        public void remove(int index){
    
        }
        //    获取长度
        public int size(){
            return size;
        }
        //    判断是否为空
        public boolean isEmpty(){
            return size==0;
        }
    }
    ```

## 链式表



- 单项链表

  - 定义：

    ```java
    //单向链表
    class LinkNode<E>{
        E data;
    //    下一个节点
        LinkNode<E> next;
        public LinkNode(E data){
            this.data=data;
            this.next=null;
        }
        public  LinkNode(E data,LinkNode<E> next){
            this.data=data;
            this.next=next;
        }
    }
    ```

    

  - 基本操作：

    ```java
    
    ```

- 双向链表

  - 定义：

    ```java
    //双向链表
    class DoubleLinkNode<E>{
        E data;
        DoubleLinkNode<E> prior;//上一个节点
        DoubleLinkNode<E> next;//下一个节点
        public DoubleLinkNode(E data){
            this.data=data;
            this.prior=null;
            this.next=null;
        }
        public DoubleLinkNode(E data,DoubleLinkNode<E> next){
            this.data=data;
            this.prior=null;
            this.next=next;
        }
    
    }
    ```

  - 基本操作：

    ```Java
    
    ```

- 循环链表

  - 循环单链表：

    - 定义：

      ```java
      //单向循环链表
      class LoopLinkNode<E>{
          E data;
          LoopLinkNode next;
          public LoopLinkNode(E data){
              this.data=data;
              this.next=this;
          }
      }
      ```

      

    - 基本操作：

      ```Java
      
      ```

      

  - 循环双链表：

    - 定义：

      ```Java
      //双向循环链表
      class DoubleLoopLinkNode<E>{
          E data;
          DoubleLoopLinkNode prior;
          DoubleLoopLinkNode next;
          public DoubleLoopLinkNode(E data){
              this.data=data;
              this.prior=this;
              this.next=this;
          }
      }
      ```

    - 基本操作：

      ```Java
      
      ```

      

- 静态链表

  - 定义：
  - 基本操作：

## 栈

后进先出，压子弹一样，射出去的总是最后压下去的。

- 栈顺序存储：【使用数组实现】

  - 定义与基本操作：

    ```java
    class SqStack<E>{
        E[] data;
        int top;
        int size;
        public SqStack(int size){
            this.size=size;
            this.data=(E[]) new Object[this.size];
            this.top=-1;
            Stack<Object> objects = new Stack<>();
        }
        public SqStack(E data,int size){
            this.size=size;
            this.data=(E[]) new Object[this.size];
            this.data[0]=data;
            this.top=0;
        }
        //    进栈
        public boolean push(E data){
            if(this.top+1==this.size){
                return false;
            }
            this.top=this.top+1;
            this.data[this.top]=data;
            return true;
        }
        //    出栈
        public E pop(){
            if(this.top==-1){
                return null;
            }
            E datum = this.data[this.top];
            this.data[this.top]=null;
            this.top=this.top-1;
            return datum;
        }
        //    获取栈顶元素
        public E getTop(){
            if(this.top==-1){
                return null;
            }
            return this.data[this.top];
        }
        public boolean isEmpty(){
            if(top==-1){
                return true;
            }
            return false;
        }
    }
    ```

- 栈链式存储：【使用链表实现】

  - 定义：

    ```Java
    //链栈
    class LinkStack<E>{
        E data;
        LinkStack<E> next=null;
        public LinkStack(E data){
            this.data=data;
            this.next=null;
        }
    }
    ```

  - 基本操作：

    ```java
    
    ```

## 队列

先进先出的线性表
术语：队头、队尾、空队列。

- 顺序队列
- 链式队列
- 双端队列

## 串

## 树

## 图


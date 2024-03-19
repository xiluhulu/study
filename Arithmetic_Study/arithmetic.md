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
    //    下一个结点
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
        DoubleLinkNode<E> prior;//上一个结点
        DoubleLinkNode<E> next;//下一个结点
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

![image-20240319145518825](E:\GitHubWorkArea\study\Arithmetic_Study\arithmetic.assets\image-20240319145518825.png)

非空树的特性：

- 树是n个结点的有限集合，n=0；
- 有且仅有一个特点结点的称为根的结点。

结点之间的描述：

- 祖先结点：父结点以上的都称为祖先结点；
- 子孙结点：子结点一下的都是子孙结点；
- 双亲结点（父结点）：字面意思；
- 孩子结点（子结点）：字面意思；
- 兄弟结点：同意父结点下的同一层；
- 堂兄弟结点：不同父结点但同祖先结点的同一层结点；

树属性的描述：

- 结点的层次：从头结点往最下的数找到最大的便是（树的高度）层数，该节点在第几层便是第几层；
- 结点的高度：从最下的结点层数往上数，数到头节点便是节点的高度；
- 树的高度：从头结点往最下的数找到最大的便是（树的高度）层数；
- 结点的度：该结点有多少个子节点，不算孙结点；
- 树的度：在数中取结点的度最大的作为树的度；

### 完全二叉树

- 顺序存储

  - 定及基本操作：

    ```java
    //完全二叉树的顺序存储
    class CompleteBinaryTree<E>{
        private E[] data;
        private int size;
    
        public  CompleteBinaryTree(int size)   {
            if((size-1)%2!=0){
                throw  new IllegalArgumentException("大小不合理");
            }
            data = (E[]) new Object[size];
        }
        public  CompleteBinaryTree(Integer tier) {
            int max_size=0;
            if(tier==1){
                max_size=tier;
            }
            if(tier>1){
                for(int i=0;i<tier;i++){
                    int x=1;
                    for(int j=0;j<i;j++){
                        x*=2;
                    }
                    max_size+=x;
                }
            }
            data = (E[]) new Object[max_size];
        }
    //    按层次顺序插入
        public void insert(E data) throws Exception {
            if(this.size<this.data.length){
                this.data[this.size]=data;
                this.size++;
            }else {
                throw new Exception("已满");
            }
        }
    
    }
    ```

    

- 链式存储



​	

## 图

## 算法例题

### 1【蓝桥杯题库 2414.滑行】![image-20240317202054798](E:\GitHubWorkArea\study\Arithmetic_Study\arithmetic.assets\image-20240317202054798.png)

![image-20240317202212441](E:\GitHubWorkArea\study\Arithmetic_Study\arithmetic.assets\image-20240317202212441.png)

```Java

import java.util.Scanner;
import java.util.HashSet;
// 1:无需package
// 2: 类名必须Main, 不可修改

public class Main {
    static int sum=-1;
    static int[][] bj;
    static int[][] f;

    public static void main(String[] args) {
        
        Scanner scan = new Scanner(System.in);
        //在此输入您的代码...
        int n = scan.nextInt();
        int m = scan.nextInt();
        int[][] arrh= new int[n][m];
        f= new int[n][m];
        bj= new int[n][m];

        for(int h=0;h<n;h++){
            for(int w=0;w<m;w++){
                arrh[h][w]=scan.nextInt();
            }
        }
        for(int i=0;i<n;i++){
            for(int j=0;j<m;j++){

                sum = Math.max(sum,dfs(arrh,j,i,n,m,1));
            }
        }
        scan.close();
        System.out.println(sum);
        
    }
    //dfa深度搜索+记忆搜索
    public static int dfs(int[][] arrh,int x,int y,int n,int m ,int step){
            if(bj[y][x]==1){
                return f[y][x];
            }
            bj[y][x]=1;
            f[y][x]=1;
            //右
            if(x+1>=0 && x+1<m && arrh[y][x]>arrh[y][x+1] ){
                f[y][x]=Math.max(f[y][x],dfs(arrh,x+1,y,n,m,step+1)+1);
            }
            //下
            if(y+1>=0 && y+1<n && arrh[y][x]>arrh[y+1][x] ){
                f[y][x]=Math.max(f[y][x],dfs(arrh,x,y+1,n,m,step+1)+1);
            }
            //左
            if(x-1>=0  && x-1<m && arrh[y][x]>arrh[y][x-1] ){
                f[y][x]=Math.max(f[y][x],dfs(arrh,x-1,y,n,m,step+1)+1);
            }
            //上
            if(y-1>=0 && y-1<n && arrh[y][x]>arrh[y-1][x] ){
                f[y][x]=Math.max(f[y][x],dfs(arrh,x,y-1,n,m,step+1)+1);
            }


        return f[y][x];

    }
}
```



### 2【力扣 [200. 岛屿数量](https://leetcode.cn/problems/number-of-islands/)】

![](E:\GitHubWorkArea\study\Arithmetic_Study\arithmetic.assets\image-20240317233053279.png)![image-20240317233116403](E:\GitHubWorkArea\study\Arithmetic_Study\arithmetic.assets\image-20240317233116403.png)

![image-20240317233139864](E:\GitHubWorkArea\study\Arithmetic_Study\arithmetic.assets\image-20240317233139864.png)

```Java
class Solution {
    static boolean[][] v;
    public int numIslands(char[][] grid) {
        int r=grid.length,c=grid[0].length,sum=0;
        v=new boolean[r][c];
        for(int i=0;i<r;i++){
            for(int j=0;j<c;j++){
                boolean dfs = dfs(grid, i, j, r, c);
                if(dfs){
                    sum++;
                }
            }
        }
        return sum;

    }
    public boolean dfs(char[][] grid,int y,int x,int r,int c){
        //判断当前坐标是否被访问
        if(!(grid[y][x]=='1') || v[y][x]){
            return false;
        }
        //为被访问则将状态设置成以访问
        v[y][x]=true;
        //判断能否往右访问
        if(x+1<c && !v[y][x+1] && grid[y][x+1]=='1'){
                dfs(grid, y, x + 1, r, c);
        }
        //判断能否往下访问
        if(y+1<r && !v[y+1][x] && grid[y+1][x]=='1'){
            dfs(grid, y + 1, x, r, c);
        }
        //判断能否往左访问
        if(x-1>=0 && !v[y][x-1] && grid[y][x-1]=='1'){
            dfs(grid, y, x - 1, r, c);

        }
        //判断能否往上访问
        if(y-1>=0 && !v[y-1][x] && grid[y-1][x]=='1'){
            dfs(grid, y - 1, x, r, c);

        }
        return v[y][x];
    }
}
```


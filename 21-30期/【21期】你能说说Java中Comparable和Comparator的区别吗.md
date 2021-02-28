#Java中Comparable和Comparator的区别
##Comparable
>  比较者大于被比较者，返回正整数
    比较者等于被比较者，返回0
    比较者小于被比较者，返回负整数

```

public class Domain implements Comparable<Domain>
{
   private String str;

   public Domain(String str)
   {
       this.str = str;
   }

   public int compareTo(Domain domain)
   {
       if (this.str.compareTo(domain.str) > 0)
           return 1;
       else if (this.str.compareTo(domain.str) == 0)
           return 0;
       else 
           return -1;
   }

   public String getStr()
   {
       return str;
   }
}
```
```
public static void main(String[] args)
   {
       Domain d1 = new Domain("c");
       Domain d2 = new Domain("c");
       Domain d3 = new Domain("b");
       Domain d4 = new Domain("d");
       System.out.println(d1.compareTo(d2));
       System.out.println(d1.compareTo(d3));
       System.out.println(d1.compareTo(d4));
   }
```
运行结果为：
```
0
1
-1
```

##Comparator
>  o1大于o2，返回正整数
    o1等于o2，返回0
    o1小于o3，返回负整数

```
public class DomainComparator implements Comparator<Domain>
{
   public int compare(Domain domain1, Domain domain2)
   {
       if (domain1.getStr().compareTo(domain2.getStr()) > 0)
           return 1;
       else if (domain1.getStr().compareTo(domain2.getStr()) == 0)
           return 0;
       else 
           return -1;
   }
}
```

```
public static void main(String[] args)
{
   Domain d1 = new Domain("c");
   Domain d2 = new Domain("c");
   Domain d3 = new Domain("b");
   Domain d4 = new Domain("d");
   DomainComparator dc = new DomainComparator();
   System.out.println(dc.compare(d1, d2));
   System.out.println(dc.compare(d1, d3));
   System.out.println(dc.compare(d1, d4));
}
```
看一下运行结果：
```
0
1
-1
```
因为泛型指定死了，所以实现Comparator接口的实现类只能是两个相同的对象（不能一个Domain、一个String）进行比较了，实现Comparator接口的实现类一般都会以"待比较的实体类+Comparator"来命名

##总结
如果实现类没有实现Comparable接口，又想对两个类进行比较（或者实现类实现了Comparable接口，但是对compareTo方法内的比较算法不满意），那么可以实现Comparator接口，自定义一个比较器，写比较算法。

实现Comparable接口的方式比实现Comparator接口的耦合性要强一些，如果要修改比较算法，要修改Comparable接口的实现类，而实现Comparator的类是在外部进行比较的，不需要对实现类有任何修改。
- 对于一些普通的数据类型（比如 String, Integer, Double…），它们默认实现了Comparable 接口，实现了 compareTo 方法，我们可以直接使用。 
- 而对于一些自定义类，它们可能在不同情况下需要实现不同的比较策略，我们可以新创建 Comparator 接口，然后使用特定的 Comparator 实现进行比较。
---
title: "阿里巴巴Java规约学习-foreach"
date: 2017-02-14 00:21:13
tags: 
        java
---

> 
    7. 【强制】不要在 foreach 循环里进行元素的 remove / add 操作。 remove 元素请使用 Iterator
    方式，如果并发操作，需要对 Iterator 对象加锁。
    反例： 
    List<String> a = new ArrayList<String>();
        a.add("1");
        a.add("2");
        for (String temp : a) {
        if("1".equals(temp)){
        a.remove(temp);
        }
    } 
    说明：以上代码的执行结果肯定会出乎大家的意料，那么试一下把“1”换成“2”，会是同样的
    结果吗？
    正例： 
    Iterator<String> it = a.iterator();
        while(it.hasNext()){
            String temp = it.next();
            if(删除元素的条件){
            it.remove();
        }
    } 

先看看输出结果: 

```Java 

public class Test {
    public static void main(String[] args) {
        List<String> a = new ArrayList<String>();
        a.add("1");
        a.add("2");
//        for (String temp : a) {
//            if ("1".equals(temp)) {
//                a.remove(temp);//正常
//            }
//        } 
//        for (String temp : a) {// java.util.ConcurrentModificationException at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:859) at java.util.ArrayList$Itr.next(ArrayList.java:831)
//            if ("2".equals(temp)) {
//                a.remove(temp);
//            }
//        }
        Iterator<String> it = a.iterator();
        while (it.hasNext()) {
            String temp = it.next();
            if ("1".equals(temp)) {//不论是1,2都正常
                it.remove();
            }
        }
        System.out.println(a.toString());
    }

}
```

<!--more-->

---------- 

### 为什么用foreach的时候，1没问题，2出错了呢？
先从报错看异常是怎么出来的。
由报错可知，是next方法中调用checkForComodification出错。

![next](http://img.blog.csdn.net/20170213150134707?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGhld2luZGtlZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![checkForComodification](http://img.blog.csdn.net/20170213150157426?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGhld2luZGtlZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**所以是modeCount !=expectedModCount导致的。**


[foreach是通过Iterable接口在序列中进行移动。](http://www.cnblogs.com/vinozly/p/5465454.html)
    这里在foreach处打断点，进入可以知道：
    **赋值给temp的时候先new Itr()(*仅第一次*)，Itr对象再hasNext()判断，接着再调用next()赋值。**

expectedModCount是初始化(此时生成一个Iterator对象)的时候被赋值的。而modCount又是AbstractList的变量。

![expectedModCount](http://img.blog.csdn.net/20170213150358622?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGhld2luZGtlZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**每次调用list的add/remove的时候，modCount++，而这时它并没有传递给expectedModCount。**

![modCount会变化](http://img.blog.csdn.net/20170213153607317?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGhld2luZGtlZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

当第一次remove掉"2"后，modCount改变 ，for(String temp:a)的时候，会调用ArrayList$Itr.next()方法，next方法会检查modCount是否等于expectedModCount，所以用foreach的时候remove 2出错了。

---------- 

### foreach的时候remove 1没出错呢？
稍微改下代码:
```Java
 List<String> a = new ArrayList<String>();
        a.add("1");
        a.add("2");
        int n=0;
        for (String temp : a) {//正常,但是n为1，说明只有一次
            n++;// 
            System.out.println(n);//只输出1次:1
            if ("1".equals(temp)) {
                a.remove(temp);
            }
        } 
```
输出结果:
```
1
[2]
```

    可见,foreach 只执行了一次循环。

---------- 

### 是什么导致本该两次的循环只进入了一次？

之前说到foreach赋值的时候，会先先用hasNext判断。cursor！=size才进入循环体。
这里的**cursor指的是当前元素的index**。循环体进入一次后，remove掉"1"，此时size由2变成了1，cursor也由于next后变为1。再次 for (String temp : a)的时候先调用hasNext因为其相等，所以不再进入循环体。

**remove后size--:**

![remove后size--](http://img.blog.csdn.net/20170213153607317?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGhld2luZGtlZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**next后,cursor+1:**

![next](http://img.blog.csdn.net/20170213150134707?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGhld2luZGtlZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**hasNext比较**

![hasNext](http://img.blog.csdn.net/20170213155819209?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGhld2luZGtlZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

---------- 

### 为什么直接用Iterator的remove是正常的？

a.iterator()直接return new Itr()，Itr 实现了Iterator。
我们看看Itr的remove方法：

![Itr的remove方法](http://img.blog.csdn.net/20170213161356327?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGhld2luZGtlZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
    
这里明显看出当它remove后， 马上把modCount的值赋值到了expectedModCount上，所以不会出现list.remove那样报ConcurrentModificationException 。

** 当然如果并发操作，需要对 Iterator 对象加锁。**


感谢阿里巴巴的Java规约~



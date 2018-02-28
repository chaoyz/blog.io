[TOC]

#OOM

##HeapOutOfMemory

jvm堆区爆炸~

> import java.util.LinkedList;
> 
> 
> 
> import java.util.List;
> 
> 
> 
>  
> 
> 
> 
> public class HeapOOM {
> 
> 
> 
>  
> 
> 
> 
>     public static void main(String[] args) {
> 
> 
> 
>         List<String> list = new LinkedList<String>();
> 
> 
> 
>         int i = 0;
> 
> 
> 
>         while(true) {
> 
> 
> 
>             list.add(new String("" + i++));
> 
> 
> 
>         }
> 
> 
> 
>  
> 
> 
> 
>     }
> 
> 
> 
>  
> 
> 
> 
> }
> 
> 
> 
> 
> 
> 错误信息
> 
> 
> Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
> 
>     at java.util.LinkedList.linkLast(Unknown Source)
> 
>     at java.util.LinkedList.add(Unknown Source)
> 
>     at zTest.HeapOOM.main(HeapOOM.java:12)

##Young OutOfMemory

##MethodArea OutOfMemory

##ConstantPool OutOfMemory

##DirectMemory OutOfMemory

##Stack OutOfMemory

#Stack OverFlow
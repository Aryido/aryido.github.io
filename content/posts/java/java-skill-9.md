---
title: Java賦值語句的返回值

author: Aryido

date: 2022-11-13T23:32:56+08:00

thumbnailImage: "/images/java/java-bean-logo1.jpg"

categories:
- java

comment: false

reward: false
---
<!--BODY-->
> Java賦值語句，是有返回值的，而且還並不是想像中的 bool 類型 ！ 想想其實一直都有看到一些類似的用法，但因自己平時開發並沒有特別使用過，也沒有很深入去探討了解。今天在這邊就舉例一些出來，來說明 Java 賦值語句的返回值。

<!--more-->

---
這邊直接拿出一些 code 舉例，也方便直接展現 Java 賦值語句有返回值的便捷性。

# ArrayList 的 iterator 的 next 方法
```java
 public E next() {
    checkForComodification();
    int i = cursor;
    if (i >= size)
        throw new NoSuchElementException();
    Object[] elementData = ArrayList.this.elementData;
    if (i >= elementData.length)
        throw new ConcurrentModificationException();
    cursor = i + 1;
    return (E) elementData[lastRet = i];
}

```
注意最後一行 ```return (E) elementData[lastRet = i]``` ，明顯可以知道 *lastRest=i* 是有返回值的。

# HashMap put 方法
```java
if ((tab = table) == null || (n = tab.length) == 0)
    n = (tab = resize()).length;
```
能夠看到例如 ```(tab = table) == null```，在判斷語句中使用了賦值語句的結果來和 null 和 0 比較。

# 一些讀寫文件的 code
```java
while ((line = reader.readLine()) != null) {
    out.append(line);
}
```
把 reader.readLine() 指定給 line 來和 null 比較。

{{< alert info >}}
以上範例，都可以說明**賦值語句是有返回值得**，且**賦值語句返回的是右側的結果**。
{{< /alert >}}


---
可以寫個 code 測試
```java
public void test(){
	int index = 0;
	System.out.println(index = 2); //2
	System.out.println(index); //2

}
```
---
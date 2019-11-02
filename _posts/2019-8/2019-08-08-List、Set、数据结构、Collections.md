---
title: List、Set、数据结构、Collections
categories: 集合
tags: 集合 数据结构
date: 2019-08-08 10:24:01
background_music: '<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=52 
src="//music.163.com/outchain/player?type=2&id=2740360&auto=0&height=32"></iframe>'
---

# 删除集合中的元素

 - 第一种方式

    ArrayList<String> list = new ArrayList<>();

        list.add("Java");
        list.add("Java");
        list.add("PHP");
        list.add("Python");
        list.add("C");
    
        for (int i = 0; i < list.size(); i++) {
            // 如果i索引的元素是Java
            if ("Java".equals(list.get(i))) {
                // 删除
                list.remove(i);
                // 如果删除了元素, i--
                i--;
            }
        }
    
        System.out.println(list);  

 - 第二种方式

     ArrayList<String> list = new ArrayList<>();

        list.add("PHP");
        list.add("Python");
        list.add("Java");
        list.add("Java");
        list.add("C");

        // 倒着遍历
        for (int i = list.size() - 1 ; i >= 0; i--) {
            // 如果i索引的元素是Java
            if ("Java".equals(list.get(i))) {
                // 删除
                list.remove(i);
            }
        }

        System.out.println(list);

 - 第三种方式

     ArrayList<String> list = new ArrayList<>();

        list.add("PHP");
        list.add("Python");
        list.add("Java");
        list.add("Java");
        list.add("C");

        Iterator<String> it = list.iterator();

        while (it.hasNext()) {
            if ("Java".equals(it.next())) {
                it.remove();
            }
        }

        System.out.println(list);

   **注意** 增强for不能删除

    CopyOnWriteArrayList<E>: 迭代器保证不会抛出 ConcurrentModificationException

# 一. List 

## 1.List集合的特点
   有序(存取有序),有索引,可以重复

## 2.特有功能
增加:add(索引,元素)
       List<String> list = new ArrayList<>();

        list.add("贾巴尔");
        list.add("邓肯");
        list.add("张伯伦");
    
        list.add(1, "乔丹"); // [贾巴尔, 乔丹, 邓肯, 张伯伦]

删除:remove(索引),remove(元素)

// 如果remove方法传入的是基本数据类型int就是按照索引删除
// 如果想按照元素删除, 必须传入包装类类型的Integer

        List<Integer> list = new ArrayList<>();
        list.add(2);
        list.add(3);
        list.add(4);
        list.add(100);  // >> Integer
    
        Integer i = 2;
        list.remove(i); // 按照元素删除
    
        list.remove(2);// 按照索引删除
    
        System.out.println(list);
修改 : set(索引, 元素)

        List<String> list = new ArrayList<>();
        list.add("贾巴尔");
        list.add("邓肯");
        list.add("张伯伦");
    
        list.set(1, "邓呆呆"); // [贾巴尔, 邓呆呆, 张伯伦]

查询 : get(索引)

       List<String> list = new ArrayList<>();
        list.add("贾巴尔");
        list.add("邓肯");
        list.add("张伯伦");
    
        System.out.println(list.get(1)); // 邓肯


## 3.List集合的子类

    >> n : 除以2的n次方
    
    << n : 乘以2的n次方 

### (1)ArrayList

-ArrayList底层是使用数组结构
     查询快, 增删慢

- 数组结构图解

![2019-08-08-09-21-11.png](https://ae01.alicdn.com/kf/Hfef8dd706a9240f3b5b1b01b6ee0fc299.png)

### (2) LinkedList

- LinkedList底层使用链表结构

     查询慢, 增删快

- LinkedList的特有方法

// addFirst(元素): 添加到头
// addLast(元素): 添加到尾
// removeFirst() : 删除头
// removeLast(): 删除尾
// getFirst(): 获取头
// getLast(): 获取尾

// push(元素): 将元素推导容器中, addFirst()
// pop(): 将元素弹出去, removeFirst()

- 链表结构图解

![2019-08-08-09-24-16.png](https://ae01.alicdn.com/kf/H7bc0239998264a30b7868dd586f65bdbS.png)

### (3) Vector

- Vector底层使用数组结构

       Vector<Integer> v = new Vector<>();
    
        v.add(10);
        v.add(20);
        v.add(30);
        v.add(40);
    
        Enumeration<Integer> e = v.elements();
    
        while (e.hasMoreElements()) {
            Integer i = e.nextElement();
            System.out.println(i);
        }

# 二.Set

## 1. Set集合的特点

- 无序(存和取的顺序), 无索引, 不可以重复
- 不可以重复:  保证元素的唯一

## 2. HashSet集合如何保证元素唯一

- HashSet保证元素唯一是通过equals()和hashCode()实现
- HashSet集合中要存储自定义对象, 并需要保证元素唯一的话, 需要让自定义对象所在的类, 重写equals()和hashCode()方法

## 3. TreeSet集合如何保证元素唯一

- 可以给集合中的元素进行升序的排序
- TreeSet保证元素唯一和排序都是通过底层的二叉树来实现

## 4. LinkedHashSet

- 特点: 去重(保证元素唯一),  有序(Set集合中惟一一个可以保证怎么存就怎么取的集合)

# 5. 为什么不使用Set集合

排序

    Collections.sort(list);

保证元素唯一

    /**
    * 用来去除List集合中重复的元素
    */
    public static void getSingle(ArrayList<Integer> list{
        LinkedHashSet<Integer> set = new LinkedHashSet();
        // 将list集合中的元素, 添加到set集合中
        set.addAll(list);
       // 清空list集合
        list.clear();
        // 将set集合中的元素, 添加到list集合中
        list.addAll(set);
   }

# 三. Collections

- 是一个用来操作集合的工具类

1. sort() : 排序

    Collections.sort(list); // 将集合使用升序排序

2. shuffle() : 随机置换

   Collections.shuffle(list)

3. addAll() :  添加全部元素

    public static <T> boolean addAll(Collection<T> c,  T... elements)

可变参数 

- 作用: 可以传入任意个该类型的参数
- 可变参数其实就是数组

     如果参数列表中有多个参数, 一定要将可变参数放到最后

4. 比较器排序

    public static <T> void sort(List<T> list,  Comparator<T> c)
       // 方法的参数列表是Comparator接口, 那么实际传入的就是它的实现类对象
       // 在Collections的sort方法中, 需要传入比较器对象(Comparator的子类对象 ---  匿名内部类)
       Collections.sort(list, new Comparator<Integer>() {
      // 如果要升序排序:  o1 - o2
      // 如果要降序排序:  o2 - o1
       @Override
       public int compare(Integer o1, Integer o2) {
          return o2 - o1;
       }
    });

    // 字符串的排序
   Collections.sort(list, new Comparator<String>() {
       @Override
       public int compare(String o1, String o2) {
        return o2.compareTo(o1);
      }
   });


总结

list: 

- ArrayList
- LinkedList

set: 

- HashSet
- TreeSet
- LinkedHashSet

详细集合分类图

![2019-08-08-10-21-28.jpg](https://ae01.alicdn.com/kf/H91c29e3ab3b14d0d9a64b53aa8e74dc7r.jpg)

家族图

![2019-08-08-10-22-22.jpg](https://ae01.alicdn.com/kf/H1c295c88009c47afb106c8e96293a662T.jpg)


---
title: Java排序之Comparable与Comparator
date: 2019-05-18 22:49:54
tags:
---

平时我们在对集合或数组排序时，会碰到Comparable和Comparator这两个很类似的接口，它们以不同的方式实现了元素的排序功能，今天我们就来谈谈这两个接口的区别，并介绍下Java排序一些常见方法的使用。

首先介绍一下Comparable与Comparator的区别。

>Comparable是排序接口，若一个类实现了Comparable接口，该类的对象就支持排序；<br>
而Comparator是比较器接口，我们可以实现一个Comparator接口（一般采用匿名内部类的写法），定义一种对A类的排序规则，而不需要对A类做任何改变。<br>
所以，Comparable相当于内部比较器，而Comparator相当于外部比较器。

下面列出集合排序时一些常用的方法：
- list.sort(comparator)
- Collections.sort(list)
- Collections.sort(list, comparator)
- list.stream().sorted();
- list.stream().sorted(comparator)

可以发现有的排序方法要求提供Comparator接口作为参数，有些则没有要求；<br>
若提供了Comparator接口，则集合的元素无需实现Comparable接口，若元素没有实现Comparable接口，则需要提供Comparator接口。

### Comparable接口使用示例

```java
public class ComparableTest {
    public static void main(String[] args) {
        List<Student> list = Arrays.asList(
                new Student("张三",80),
                new Student("李四",67),
                new Student("王五",93)
        );
        
        // 对学生按成绩排序
        Collections.sort(list);
        System.out.println(list);
        // [Student{name='李四', score=67}, Student{name='张三', score=80}, Student{name='王五', score=93}]
    }
}

class Student implements Comparable<Student> {
    private String name;
    private int score;

    @Override
    public int compareTo(Student o) {
        if (this.score > o.score) {
            return 1;
        } else if (this.score < o.score) {
            return -1;
        } else {
            return 0;
        }
    }

    public Student(String name, int score) {
        this.name = name;
        this.score = score;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", score=" + score +
                '}';
    }
};
```

### Comparator接口使用示例

```java
public class ComparatorTest {
    public static void main(String[] args) {
        List<Teacher> list = Arrays.asList(
                new Teacher("老王", 30),
                new Teacher("老张", 27),
                new Teacher("老李", 45)
        );

        // 对老师按年龄排序
        list.sort(new Comparator<Teacher>() {
            @Override
            public int compare(Teacher o1, Teacher o2) {
                int age1 = o1.getAge();
                int age2 = o2.getAge();
                if (age1 > age2) {
                    return 1;
                } else if (age1 < age2) {
                    return -1;
                } else {
                    return 0;
                }
            }
        });
        System.out.println(list);
        //[Teacher{name='老张', age=27}, Teacher{name='老王', age=30}, Teacher{name='老李', age=45}]
    }
}

class Teacher {
    private String name;
    private int age;

    public Teacher(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Teacher{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
````

可以发现，上面两个例子排序的结果都是从小到大，这种排序顺序在Java里叫做自然排序，采取上面的写法就可以了，如果你想要排序的结果是从大到小，只需要交换大于时和小于时的返回值即可。<br>

这时候，你可能就明白了String、Integer等对象为什么能够排序了，打开Integer类的源码，果然Integer类也实现了Comparable接口。<br>
Java中，String、Integer、Double等类，都实现了Comparable接口，而且排序时候的顺序都是自然排序，即从小到大。

### 按字母排序字符串列表的示例

```java
List<String> cities = Arrays.asList("c", "a", "B");
cities.sort(Comparator.naturalOrder());//自然排序（从小到大）
System.out.println(cities);//[B, a, c]

cities.sort(String.CASE_INSENSITIVE_ORDER);// 忽略大小写，自然排序（从小到大）
System.out.println(cities);//[a, B, c]

cities.sort(Comparator.reverseOrder());//反转（从大到小）
System.out.println(cities);//[c, a, B]
```

### Comparator的一些其他技巧

```java
List<Teacher> list = Arrays.asList(
        new Teacher("a_t", 30),
        new Teacher("b_t", 27),
        new Teacher("c_t", 45),
        new Teacher("d_t", 30)
);

//使用age排序
list.sort(Comparator.comparing(Teacher::getAge));
System.out.println(list);
//[Teacher{name='b_t', age=27}, Teacher{name='a_t', age=30}, Teacher{name='d_t', age=30}, Teacher{name='c_t', age=45}]


//使用age排序，并反转
list.sort(Comparator.comparing(Teacher::getAge).reversed());
System.out.println(list);
//[Teacher{name='c_t', age=45}, Teacher{name='a_t', age=30}, Teacher{name='d_t', age=30}, Teacher{name='b_t', age=27}]


//使用age排序，并反转，当age相同时，使用name排序，并反转
list.sort(Comparator.comparing(Teacher::getAge).reversed().thenComparing(Teacher::getName).reversed());
System.out.println(list);
//[Teacher{name='b_t', age=27}, Teacher{name='d_t', age=30}, Teacher{name='a_t', age=30}, Teacher{name='c_t', age=45}]
```
注意，Teacher类并没有实现Comparable接口。<br>
上面的一些写法用到了lambda表达式，这样可以让我们的代码更加简洁，还没使用的小伙伴快加入吧！

(完)
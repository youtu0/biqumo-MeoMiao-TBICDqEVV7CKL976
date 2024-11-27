
在Java语言中，数据类型分为基本数据类型和引用数据类型。


基本数据类型（如`int`、`double`、`char`等）的值直接保存在栈上。这些类型的变量在栈内存中有固定的大小，并且值是直接存储在这些变量中的，数据的传递为值传递，这个好理解。以下以引用数据类型来讲解。


## 引用和实例化对象


比如new一个对象的代码：等号左边的`person`为引用；等号的右边`new Person()`为实例化对象。



```
Person person = new Person();
引用 = 实例化对象

```

`引用`和`实例化对象`在内存中，分别保存在`栈`和`堆`中。引用会保存着实例化对象的地址，从而可以通过引用来获取到具体的实例化对象保存在哪里。


关系如图：


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1256e7b09e6f45658d8ef188bb67d311.jpeg#pic_center)


## 值传递和引用传递


顾名思义，值传递是把“值”传递到方法中，而引用传递是把“引用”传递到方法中。


在Java 的参数传递方式中，**只有值传递**。对于引用对象也是值传递，而这个`值`是`引用的值（即堆地址或句柄）`被拷贝传递到方法中。


文字太抽象了，看图和代码：


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b2dc77c73b694a51a7c0822c55c31a7f.jpeg#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5f8b302547624847a888323ac7268a41.jpeg#pic_center)


### 对比两者


值传递



```
源引用地址：0x0a --> 对象地址：0x10
传递时：新引用地址：0x0b --> 对象地址：0x10

```

引用传递



```
源引用地址：0x0a --> 对象地址：0x10
传递时：方法中仍是引用地址：0x0a --> 对象地址：0x10

```

**值传递**：是复制一份“引用”传给方法的形参，`person` 和 `person2` 是两个不同的栈内存地址（如 `0x0a`和`0x0b`）


**引用传递**：是将实参的引用直接传给方法的形参，`person` 和 `person2` 实际共享了相同的栈内存地址（如 `0x0a`）


两者有着本质的区别！对比两者的行为和带来的影响。


### 对比示意图




| **行为** | **值传递（Java 的实际行为）** | **引用传递（假设机制）** |
| --- | --- | --- |
| 方法参数接收到的值 | 引用地址的副本（如 `0x0b`） | 原始引用地址本身（如 `0x0a`） |
| 引用的改变影响范围 | 改变方法内的引用指向，不影响原始引用 | 改变引用指向会影响原始引用 |
| 对象属性的修改 | 通过引用修改对象属性，会影响原始对象 | 通过引用修改对象属性，会影响原始对象 |


### 代码验证


测试代码：测试引用的改变影响范围和对象属性的修改



```
public class ValuePassDemo {

    public static void main(String[] args) {
        Person person = new Person();
        person.setAge(18);
        person.setName("Denny");
        // 初始值
        System.out.println("地址："+ Integer.toHexString(person.hashCode()) + ">>>" + person);
        modifyReference(person);
        // 是否会被上一个方法修改值
        System.out.println("地址："+ Integer.toHexString(person.hashCode()) + ">>>" + person);
        modifyReference2(person);
        // 是否会被上一个方法修改值
        System.out.println("地址："+ Integer.toHexString(person.hashCode()) + ">>>" + person);
    }

    /**
     * 形参和实参引用指向的实例化对象是同一个
     * 实例化对象的值被任意一边修改时，都会改变
     */
    public static void modifyReference(Person person2) {
        person2.setAge(28);
        person2.setName("Jack");
        System.out.println("地址："+ Integer.toHexString(person2.hashCode()) + ">>>" + person2);
    }

    /**
     * 如果是引用传递，
     *   那实参引用 person 等于形参引用 person2，
     *   那么引用 person2 的指向被改变的话，形参引用 person也会指向新的实例化对象
     * 如果不成立，那就是值传递，引用person2 只是引用person的拷贝，而非本身给了它
     */
    public static void modifyReference2(Person person2) {
        person2 = new Person();
        person2.setAge(20);
        person2.setName("apple");
        System.out.println("地址："+ Integer.toHexString(person2.hashCode()) + ">>>" + person2);
    }
}

```

测试结果：


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b2a5bddffe784cec925608a2e132b7cf.jpeg#pic_center)


### 方法内部修改引用的指向


调用`modifyReference2`方法，会给形参变量赋一个新的实例化对象的情况，


如果是值传递，形参和实参分别指向不同的实例化对象，如图：


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/08f101c3c2f848ccb0f6b72ce78f3d02.jpeg#pic_center)


如果是引用传递，形参和实参都指向相同的实例化对象，而原来的实例化对象就没有引用指向。


如图：


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/536f6c00253644d1af82387b68de076c.jpeg#pic_center)


原来的实例化对象没有引用指向，会导致内存泄漏，用C\+\+的语言描述：按引用传递时，并且在方法内修改引用指向新new的对象时，需要手动释放内存。


* **`void myFunction(Person* obj)`** 是**按值传递**。
* **`void myFunction(Person*& obj)`** 是**引用传递**。



```
void myFunctionWithReference(Person*& obj) {
    delete obj;  // 先释放原对象的内存
    obj = new Person(20);  // 重新分配新的对象，并让 obj 指向它
}

```

## 为什么Java只有值传递


个人觉得Java 选择只有**值传递**的参数传递机制（pass\-by\-value）目的应该包含：内存安全性、简化内存管理、保持语言行为一致性和语言简单易用。


这也是Java语言的优点，弱化对内存操作的概念，让这门语言更加简洁易用；同时这也是Java语言的缺点，降低了灵活性，无法直接通过方法修改引用变量的指向。


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/98b3825eac5f4fe381aeb6510c086c41.png)


[超实用的SpringAOP实战之日志记录](https://github.com)


[软考中级\-\-软件设计师毫无保留的备考分享](https://github.com)


[单例模式及其思想](https://github.com)


[2023年下半年软考考试重磅消息](https://github.com)


[通过软考后却领取不到实体证书？](https://github.com):[westworld加速器](https://xbsj9.com)


[计算机算法设计与分析（第5版）](https://github.com)


[Java全栈学习路线、学习资源和面试题一条龙](https://github.com)


[软考证书\=职称证书？](https://github.com)


[什么是设计模式？](https://github.com)



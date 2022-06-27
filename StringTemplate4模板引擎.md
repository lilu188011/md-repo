## StringTemplate4模板引擎

### 一、POM导入

```
<dependency>
  <groupId>org.antlr</groupId>
  <artifactId>ST4</artifactId>
  <version>4.3</version>
  <scope>compile</scope>
</dependency>
```

### 二、入门

#### 1 模板(ST)

本质上，ST 的语法很简单，分为两个部分：`文本和属性表达式 (attribute expressions)。`

- 文本部分会被原样输出。
- 属性表达式会被求值后输出。

```java
    @Test
   public void test01(){
        ST hello = new ST("Hello <name>");
        hello.add("name","World");
        System.out.println(hello.render()); // Hello World
    }

```

**`<name>`就被替换为了`World`，所以合起来的输出是`Hello World`。**

**分隔符**
默认属性表达式使用`尖括号`包围，当然这个是可以自定义的，比如如果你要生成 HTML 代码，用尖括号就非常麻烦了，可以改用%。比如上面的例子中的模板：

```java
    @Test
    public void test01_delimiter(){
        ST hello = new ST("Hello %name%",'%','%');
        hello.add("name","World");
        System.out.println(hello.render()); // Hello World
    }

```

#### 2 模板组（Groups of templates）

代码生成是具有复杂逻辑的，一般不会在一个模板中搞定，而是`分解为多个小的模板，然后拼装起来`。
ST 的模板更加强大，可以有输入参数，写起来和编程语言很类似。一个模板组里可以定义多个模板，**只有在同一个模板组里的模板才可以互相调用**。

```java
    @Test
    public void test_group1(){
        STGroup stGroup = new STGroup();
        //defineTemplate 需要三个参数，分别是模板名称，模板参数，模板内容
        stGroup.defineTemplate("template1","input1","【 <input1> 】");
        //调用 template1 内容
        stGroup.defineTemplate("template2","input2","Hello 		<template1(input2)>");

        ST st = stGroup.getInstanceOf("template2");
        st.add("input2","World");
        System.out.println(st.render()); // Hello 【 World 】
    }

```

##### **文件中读取模板组**

```text
/**
 * <type> : 使用入参
 * <init(value)>: 调用方法
 */
decl(type, name, value) ::= "<type> <name><init(value)>;"
init(v) ::= "<if(v)> = <v><endif>"

```

```java
@Test
public void test03_template_file(){
    STGroup group = new STGroupFile("tmp\\test.stg");
    ST st = group.getInstanceOf("decl");
    st.add("type", "int");
    st.add("name", "x");
    st.add("value", 0);
    System.out.println(st.render()); // int x = 0;
}
```
##### 字符串读取模板组

```java
@Test
public void test04_template_String(){
    String s = "say(name) ::= \"hello, <name>.\"";
    STGroup group = new STGroupString(s);
    ST st = group.getInstanceOf("say");
    st.add("name", "zhangsan");

    System.out.println(st.render()); //hello, zhangsan.
}
```
##### 向模板赋值

###### 传递数组

如果多次 add 同一个参数的值，是**不会覆盖**的，而是**追加**，也就是`每个参数其实是一个数组`:

```java
@Test
public void test04_template_multi_String(){
    String s = "say(name) ::= \"hello, <name>.\"";
    STGroup group = new STGroupString(s);
    ST st = group.getInstanceOf("say");
    st.add("name", "zhangsan");
    st.add("name", "lisi");

    System.out.println(st.render()); //hello, zhangsanlisi.
}
```
###### **控制多个值的输出格式**

```java
    @Test
    public void test04_template_array1_String(){
    	// say(name) ::= "hello, <name;separator=",">"    //设置分隔符
        String s = "say(name) ::= \"hello, <name;separator=\\\",\\\">.\"";
        STGroup group = new STGroupString(s);
        ST st = group.getInstanceOf("say");
        st.add("name", "zhangsan");
        st.add("name", "lisi");

        System.out.println(st.render()); //hello, zhangsan,lisi.
    }

```

###### 使用模板来处理每一个元素

```
say(name) ::= "hello, <name:bracket();separator=",">"
bracket(x) ::= "(<x>)"

```

###### 匿名模板

```
say(name) ::= "hello, <name:{x|[<x>]};separator=",">"

```

```java
st.add("name", "mushan");
st.add("name","willing");

// 输出： hello, [mushan],[willing]

```

##### 传递对象

```java
public static class User {
    public int id; // template can directly access via u.id
    private String name; // template can't access this -- 私有属性无法访问
    public User(int id, String name) { this.id = id; this.name = name; }
    public boolean isManager() { return true; } // u.manager
    public boolean hasParkingSpot() { return true; } // u.parkingSpot
    public String getName() { return name; } // u.name
    public String toString() { return id+":"+name; } // u
}

```

> 属性的范围是比较简单的，仅可以访问`public`属性
> 方法的访问有一套映射关系，比如对于`o.p`，st 会在类上尝试访问`getP(), isP(), hasP()`，如果都不存在，则会报错。

```java
ST st = new ST("<b>$u.id$</b>: $u.name$", '$', '$');
st.add("u", new User(999, "parrt"));
String result = st.render(); // "<b>999</b>: parrt"

```

可以看到在 ST 模板中使用 Java 类是非常简单的。但是和 js 中直接使用 json 相比，Java 中定义一个类，只是为了传递数据就定义一个类是非常繁琐的，
还好，ST 为我们提供了一种简便的方法来传递对象性质的信息：

```java
//模板
//introduction(person) ::= "I'm <person.name>, I'm <person.age> years old."

st = stGroupFile.getInstanceOf("introduction");
st.addAggr("person.{name,age}","mushan",18); // addAggr
System.out.println(st.render());

// 输出： I'm mushan, I'm 18 years old.

```

## StringTemplate 模板语

ST 模板的语法很丰富，可以`实现复杂逻辑`，以处理复杂的代码生成需求。

上面我提到的语法是最常用的语法，全面的模板语法可以参考[stringtemplate4/cheatsheet.md](https://github.com/antlr/stringtemplate4/blob/master/doc/cheatsheet.md)
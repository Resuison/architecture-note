## 1. 引言

本文尝试给出一些不同类型的空值检查或使用NPE(NullPointerException)避免技术的示例。

## 2. java.util.Optional

一个容器对象，可能包含也可能不包含非空值。如果存在值，`isPresent()`将返回`true`并将`get()`返回该值。

在处理的项目中使用 Optional 的最常见的地方之一是从数据库中检索数据时。

假设你要做三件事：

- 一个模型类称为 `Student.java`
- 服务接口及其实现：`StudentService.java`和`StudentServiceImpl.java`
- DAO 接口及其实现：`StudentDao.java`和`StudentDaoImpl.java`

并且您想检索具有相关 ID 的学生。

### 2.1 Optional.ofNullable()

返回一个描述给定值的 Optional，如果非 null，否则返回一个空 Optional。

```java
package dao;

import com.mongodb.client.MongoClient;
import lombok.RequiredArgsConstructor;
import model.Student;
import org.springframework.stereotype.Repository;

import java.util.Optional;

import static com.mongodb.client.model.Filters.eq;

@Repository
@RequiredArgsConstructor
public class StudentDaoImpl implements StudentDao {
    private final MongoClient mongoClient;

    @Override
    public Optional<Student> findStudentsById(Long id) {
        return Optional.ofNullable(mongoClient.getDatabase("dbName")
                .getCollection("collectionName", Student.class)
                .find(eq("id", id))
                .first());
    }
}
```

此方法的返回值永远不会为空。如果为 null，则返回值为`Optional.empty()`. 这样，如果这种方法的结果在其他地方使用，就没有机会获得 NPE。

### 2.2 Optional.orElseThrow

如果存在值，则返回该值，否则抛出 NoSuchElementException。

在 null 的情况下，如果你想抛出一个异常，你可以使用`orElseThrow()`

```
package service;

import dao.StudentDao;
import exception.CustomException;
import lombok.RequiredArgsConstructor;
import model.Student;
import org.springframework.stereotype.Service;

import java.util.Optional;

import static exception.ExceptionConstants.STUDENT_NOT_FOUND;

@Service
@RequiredArgsConstructor
public class StudentServiceImpl implements StudentService{
    private final StudentDao studentDao;

    @Override
    public Student getStudentById(Long id) {
        return studentDao.findStudentsById(id).orElseThrow(()-> new CustomException(STUDENT_NOT_FOUND));
    }
}
```

### 2.3 Optional.orElse

如果存在值，则返回该值，否则返回其他值。在 null 的情况下，如果您不想抛出异常但想返回示例学生实例，`orElse()`则可以使用。

```java
@Override 
public Student getMockStudent(Long id) { 
    final var student = Student. builder () 
            .id(1L) 
            .name("ege") .build 
            (); 

    return studentDao.findStudentsById(id).orElse(student); 
}
```

### 2.4 Optional.get()

如果存在值，则返回该值，否则抛出 NoSuchElementException。

```java
@Override 
public Student getStudentDirectly(Long id) { 
    return studentDao.findStudentsById(id).get(); 
}
```

在这种情况下，如果您使用 IntelliJ，它会立即发出警告：![截屏2022-01-27 上午8.53.25](/Users/sghl/Library/Application Support/typora-user-images/截屏2022-01-27 上午8.53.25.png)

基本上它是说“首先检查您的学生是否为空，然后继续”。

```java
// 正确的做法
@Override 
public Student getStudentDirectly(Long id) { 
    final var studentOptional = studentDao.findStudentsById(id); 
    if(studentOptional.isPresent()){ 
        // 在这里做你的事情
    } 
}
```

`get()`当我确定我调用的方法返回了数据时，我主要在单元测试中使用。但是我不会`isPresent()`在没有实际代码的情况下使用它，即使我确信不会有空值。

另一个例子; 下面的代码片段尝试获取具有相关 ID 的学生，如果没有这样的学生，则返回默认名称（“Hayley”）。`rElse()`当有学生但没有名字时，传递给的值也会返回。

```java
@Override 
public String getStudentName(Long id) { 
    return studentDao.findStudentsById(id) 
            .map(Student::getName) 
            .orElse("Hayley"); 
}
```

## 3. apache.commons 的实用类

- 对于 Collection 实例：`CollectionUtils.isEmpty()`或`CollectionUtils.isEmpty()`
- 对于地图实例：`MapUtils.isEmpty()`或`MapUtils.isNotEmpty()`
- 对于字符串：`StringUtils.isEmpty()`或`StringUtils.isNotEmpty()`

对于列表、地图等，`isEmpty()` 检查集合/地图是否为空或大小为 0。同样，`String`它检查是否`String`为空或长度为 0。

```java
// Check for list
final var list = List.of();
// final List<Integer> list = null;

if (CollectionUtils.isEmpty(list)) {
    System.out.print("list is empty or null");
}
// Does the same thing as previous if statement
if (list == null || list.size() == 0) {
    System.out.print("list is empty or null");
}

// Check for string
final var str = "";
//final String str = null; 

if (StringUtils.isEmpty(str)) {
    System.out.println("string is empty or null");
}
// Does the same thing as previous if statement
if (str == null || str.equals("")) {
    System.out.println("string is empty or null");
}
```

为了使用`CollectionUtils`and `MapUtils`，您需要将以下依赖项添加到`build.gradle`文件中：

```
implementation org.apache.commons:commons-collections4:4.4
```

要想使用`StringUtils`你需要：

```
implementation org.apache.commons:commons-lang3:3.0
```

## 4. Streams中的Objects::nonNull

如果提供的引用为非 null 则返回`true`，否则返回`false`.

假设您有一个数据流，您将在此流上执行一些链式操作，但在此之前您希望过滤掉任何空值。

```java
final var list = Arrays.asList(1, 2, null, 3, null, 4);

list.stream().filter(Objects::nonNull).forEach(System.out::print);
```

结果：`1234`

## 5. java.util.Objects的requireNonNull方法

### 5.1 requireNonNull()

检查指定的对象引用是否不是，如果是，则`null`抛出自定义。`*NullPointerException*`此方法主要用于在具有多个参数的方法和构造函数中进行参数验证。

### 5.2 requireNonNullElse()

如果第一个参数为非，则返回第一个参数`null`，否则返回非`null`第二个参数

### 5.3 requireNonNullElseGet()

如果它是非，则返回第一个参数`null`，否则返回.³的非`null`值`supplier.get()`

最常见的三种使用构造函数的地方：

```java
package com.example.demodemo.model;

import lombok.*;

import java.util.*;

@ToString
@Getter
@Setter
@Builder
@NoArgsConstructor
public class Student {

    private Long id;
    private String name;
    private List<SchoolClass> classes;
    private Map<Integer, Teacher> teacherMap;

    public Student(Long id, String name, List<SchoolClass> classes, Map<Integer, Teacher> teacherMap) {
        this.id = Objects.requireNonNull(id, "id is required");
        this.name = Objects.requireNonNullElse(name, "hayley");
        this.classes = Objects.requireNonNullElseGet(classes, ArrayList::new);
        this.teacherMap = Objects.requireNonNullElseGet(teacherMap, HashMap::new);
    }
}
```

我们来分析一下上面的例子。

`requireNonNull on id`：我们说“这个字段是必需的，所以如果它是空的；则抛出带有“需要 id”消息的 NPE。

`requireNonNullElse on name`：我们说“这个字段是必需的，所以如果它是空的；不要抛出异常，而是为其设置一个默认值。” 在我们的例子中，默认值为“hayley”。

`requireNonNullElseGet on classes`：我们说“这个字段是必需的，所以如果它是空的；不要抛出异常，而是为其设置一个默认值。”。

不同之处`requireNonNullElse`在于，此方法需要`Supplier`作为第二个参数。

因此，如果您想将它们初始化为空列表、映射、集合等，我们可以使用方法引用与`requireNonNullElseGet.`它在处理列表、映射、集合时特别有用。

![截屏2022-01-27 上午9.03.35](/Users/sghl/Library/Application Support/typora-user-images/截屏2022-01-27 上午9.03.35.png)

![截屏2022-01-27 上午9.03.56](/Users/sghl/Library/Application Support/typora-user-images/截屏2022-01-27 上午9.03.56.png)

![截屏2022-01-27 上午9.04.12](/Users/sghl/Library/Application Support/typora-user-images/截屏2022-01-27 上午9.04.12.png)

我们来看看实际的操作：

```java
public void test() {
    // throws NPE with message "id is required"
    /* final var student = new Student(null, null, null, null);
          System.out.println(student.getName());*/

    // throws NPE with message "id is required"
    /* final var student = Student.builder().build();
          System.out.println("Student: " + student);*/

    // sets a default name as "hayley"
    final var student1 = new Student(1L, null, null, null);
    System.out.println("Student 1 name: " + student1.getName());

    // initializes classes as empty list and teacherMap as empty map
    final var student2 = new Student(1L, "ege", null, null);
    System.out.println("Student 2: " + student2);

    // using noArgsConstructor, so does not set a default value
    final var student3 = new Student();
    System.out.println("Student 3: " + student3);

    final var student4 = Student.builder().id(1L).name("ege").build();
    System.out.println("Student 4: " + student4);

    final var schoolClasses = List.of(new SchoolClass());
    final var student5 = Student.builder().id(1L).name("ege").classes(schoolClasses).build();
    System.out.println("Student 5: " + student5);
}
```

结果如下：

```
Student 1 name: hayley
Student 2: Student(id=1, name=ege, classes=[], teacherMap={})
Student 3: Student(id=null, name=null, classes=null, teacherMap=null)
Student 4: Student(id=1, name=ege, classes=[], teacherMap={})
Student 5: Student(id=1, name=ege, classes=[SchoolClass(id=null, name=null, type=null, teacher=null, students=null, extras=null, startDate=null)], teacherMap={})
```

## 6. Lombok’s Builder.Default

如果你不熟悉Lombok，我强烈建议你去看看。我个人很喜欢 Lombok，它让开发人员的生活更轻松 :)

假设您拥有`Student.java`诸如 id、name 和 classes 等字段。您可以在相关字段之前使用 put`@Builder.Default`并为其指定默认值。

```java
package com.example.demodemo.model;

import lombok.*;

import java.util.ArrayList;
import java.util.List;

@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Student {

    private Long id;
    @Builder.Default
    private String name = "hayley";
    @Builder.Default
    private List<SchoolClass> classes = new ArrayList<>();
}
```

当创建此类的实例时`Student`，它将具有“类”作为空列表，而不是 null。

```java
final var student1 = Student.builder().build();
final var student2 = new Student();
final var student3 = Student.builder().id(1L).name("ege").build();

System.out.println(student1.getClasses());
System.out.println(student2.getClasses());
System.out.println(student3.getClasses());

System.out.println(student1.getName());
System.out.println(student2.getName());
System.out.println(student3.getName());
```

打印结果如下：

```java
[]
[]
[]
hayley
hayley
ege
```

如果您只是`Student.java`这样声明字段：

```java
private Long id;
private String name;
private List<SchoolClass> classes;
```

结果如下：

```
null
null
null
null
null
ege
```

在处理List、Map等时，它对我特别有用。因为对于我正在处理的当前项目，List或Map比其他字段更可能为空。此外，如果在这些列表/映射上执行更多操作，则它们很可能在它们为空时获得 NPE。

假设您想要获取学生注册的课程的名称，并且您没有`Builder.Default`在“课程”列表中使用。

```
final var student1 = Student.builder().build();
System.out.println(student1.getClasses().stream().map(SchoolClass::getName));
```

抛出 NPE。

## 7. NotNull, NotEmpty, NotBlank Annotations

- `@NotNull` : 带注释的元素不能为空。接受任何类型

- `@NotEmpty` : 注解的元素不能为空或空。支持的类型是 CharSequence、Collection、Map、Array

- `@NotBlank`: 带注释的元素不能为空，并且必须至少包含一个非空白字符。接受字符序列

假设您有一个控制器和一个`saveStudent()`方法。当您希望 id、name 和 classes 字段不为 null 或为空时，您可以将这些注释如下所示放在 Student 类中：

```java
@NotNull
private Long id;
@NotEmpty
private String name;
@NotEmpty
private List<SchoolClass> classes;
private Map<Integer, Teacher> teacherMap;
```

如果您使用的是 Spring Boot，您可以将这些注释与`@Validated`API 请求体的注释结合起来，如下所示。

```java
@PostMapping() 
public void saveStudent(@Validated @RequestBody Student student){ 
    studentService.saveStudent(student); 
}
```

例如，您有这样的请求：

![截屏2022-01-27 上午9.09.49](/Users/sghl/Library/Application Support/typora-user-images/截屏2022-01-27 上午9.09.49.png)

如您所见，您将收到“400”错误，并且在应用程序的控制台中您将看到：

```java
DefaultHandlerExceptionResolver : Resolved [org.springframework.web.bind.MethodArgumentNotValidException: Validation failed for argument [0] in public void com.example.demodemo.controller.StudentController.saveStudent(com.example.demodemo.model.Student) with 2 errors: [Field error in object 'student' on field 'classes': rejected value [null]; 
codes [NotEmpty.student.classes,NotEmpty.classes,NotEmpty.java.util.List,NotEmpty]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [student.classes,classes]; arguments []; default message [classes]]; default message [must not be empty]] [Field error in object 'student' on field 'id': rejected value [null]; 
codes [NotNull.student.id,NotNull.id,NotNull.java.lang.Long,NotNull]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [student.id,id]; arguments []; default message [id]]; default message [must not be null]] ]

```

如果你愿意的话，可以捕获它`MethodArgumentNotValidException`并返回自定义错误。当然，你需要添加依赖：

```
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

这就是本章节我们要学的知识。当我了解更多进行空值检查的方法时，我将尝试更新此列表。有时我仍然需要使用旧的“if else”块进行空检查，但我会尽可能地尝试应用这些方法。

### 8. 参考文档

1. https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html
2. https://docs.oracle.com/javase/8/docs/api/java/util/Objects.html
3. https://docs.oracle.com/javase/9/docs/api/java/util/Objects.html
4. https://docs.oracle.com/javaee/7/api/javax/validation/constraints/NotNull.html
5. https://javaee.github.io/javaee-spec/javadocs/javax/validation/constraints/NotEmpty.html
6. https://javaee.github.io/javaee-spec/javadocs/javax/validation/constraints/NotBlank.html
7. https://www.baeldung.com/java-avoid-null-check
8. https://projectlombok.org/

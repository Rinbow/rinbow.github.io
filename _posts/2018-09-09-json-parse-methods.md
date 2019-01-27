---
title: JSON 解析的四种常用方式
tags: [java]
---

### JSON 解析的四种方式

用于数据转换的实体类：

```java
import java.util.List;

public class School {
    private String name;
    private List<Student> students;

    public School(String name, List<Student> students) {
        this.name = name;
        this.students = students;
    }

    public School() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<Student> getStudents() {
        return students;
    }

    public void setStudents(List<Student> students) {
        this.students = students;
    }

    @Override
    public String toString() {
        return "School{" +
                "name='" + name + '\'' +
                ", students=" + students +
                '}';
    }
}
```

```java
public class Student {
    private String number;
    private String name;

    public Student(String number, String name) {
        this.number = number;
        this.name = name;
    }

    public Student() {
    }

    public String getNumber() {
        return number;
    }

    public void setNumber(String number) {
        this.number = number;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Student{" +
                "number='" + number + '\'' +
                ", name='" + name + '\'' +
                '}';
    }
}
```

#### 一、传统方式

Maven 依赖：

```xml
<!-- https://mvnrepository.com/artifact/org.json/json -->
<dependency>
    <groupId>org.json</groupId>
    <artifactId>json</artifactId>
    <version>20160810</version>
</dependency>
```

示例代码：

```java
import org.json.JSONArray;
import org.json.JSONObject;

import java.util.ArrayList;
import java.util.List;

public class TestJSON {
    public static void main(String[] args) {
        parseToJson();
        parseToObject();
    }

    private static void parseToJson() {
        JSONObject jo1 = new JSONObject();
        jo1.put("number", "01");
        jo1.put("name", "ming");
        JSONObject jo2 = new JSONObject();
        jo2.put("number", "02");
        jo2.put("name", "bai");
        JSONArray array = new JSONArray();
        array.put(jo1);
        array.put(jo2);
        JSONObject jo3 = new JSONObject();
        jo3.put("name", "bjut");
        jo3.put("students", array);
        String json = jo3.toString();
        System.out.println(json);
    }

    private static void parseToObject() {
        String json = "{\"students\":[{\"name\":\"ming\",\"number\":\"01\"},{\"name\":\"bai\",\"number\":\"02\"}],\"name\":\"bjut\"}";
        JSONObject object = new JSONObject(json);
        School school = new School();
        school.setName(object.get("name").toString());
        JSONArray jsonArray = object.getJSONArray("students");
        List<Student> students = new ArrayList<Student>();
        for (int i = 0; i < jsonArray.length(); i++) {
            JSONObject o = jsonArray.getJSONObject(i);
            Student student = new Student(o.getString("number"), o.getString("name"));
            students.add(student);
        }
        school.setStudents(students);
        System.out.println(school);
    }
}
```

#### 二、Gson

Maven 依赖：

```xml
<!-- https://mvnrepository.com/artifact/com.google.code.gson/gson -->
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.5</version>
</dependency>
```

示例代码：

```java
import com.google.gson.Gson;

import java.util.ArrayList;
import java.util.List;

/**
 * @author siyunfei
 * 2018/9/8 下午11:09
 */
public class TestGson {
    public static void main(String[] args) {
        parseToJson();
        parseToObject();
    }

    private static void parseToJson() {
        School school = new School();
        Student s1 = new Student("01", "ming");
        Student s2 = new Student("02", "bai");
        List<Student> students = new ArrayList<>();
        students.add(s1);
        students.add(s2);
        school.setName("bjut");
        school.setStudents(students);
        String json = new Gson().toJson(school);
        System.out.println(json);
    }

    private static void parseToObject() {
        String json = "{\"name\":\"bjut\",\"students\":[{\"number\":\"01\",\"name\":\"ming\"},{\"number\":\"02\",\"name\":\"bai\"}]}";
        School school = new Gson().fromJson(json, School.class);
        System.out.println(school);
    }
}
```

#### 三、FastJson

Maven 依赖：

```xml
<!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.47</version>
</dependency>
```

示例代码：

```java
import com.alibaba.fastjson.JSON;

import java.util.ArrayList;
import java.util.List;

/**
 * @author siyunfei
 * 2018/9/8 下午11:31
 */
public class TestFastJson {
    public static void main(String[] args) {
        parseToJson();
        parseToObject();
    }

    private static void parseToJson() {
        School school = new School();
        Student s1 = new Student("01", "ming");
        Student s2 = new Student("02", "bai");
        List<Student> students = new ArrayList<>();
        students.add(s1);
        students.add(s2);
        school.setName("bjut");
        school.setStudents(students);
        String json = JSON.toJSONString(school);
        System.out.println(json);
    }

    private static void parseToObject() {
        String json = "{\"name\":\"bjut\",\"students\":[{\"name\":\"ming\",\"number\":\"01\"},{\"name\":\"bai\",\"number\":\"02\"}]}";
        School school = JSON.parseObject(json, School.class);
        System.out.println(school);
    }
}
```

#### 四、Jackson

Maven 依赖：

```xml
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.6</version>
</dependency>
```

导入 `jackson-databind` 会自动导入 `jackson-core` 和 `jackson-annotations` 两个依赖库。

示例代码：

```java
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.omg.CORBA.OMGVMCID;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * @author siyunfei
 * 2018/9/9 上午10:52
 */
public class TestJackson {
    public static void main(String[] args) throws IOException {
        parseToJson();
        parseToObject();
    }

    private static void parseToJson() throws JsonProcessingException {
        School school = new School();
        Student s1 = new Student("01", "ming");
        Student s2 = new Student("02", "bai");
        List<Student> students = new ArrayList<>();
        students.add(s1);
        students.add(s2);
        school.setName("bjut");
        school.setStudents(students);
        ObjectMapper mapper = new ObjectMapper();
        String json = mapper.writeValueAsString(school);
        System.out.println(json);
    }

    private static void parseToObject() throws IOException {
        String json = "{\"name\":\"bjut\",\"students\":[{\"number\":\"01\",\"name\":\"ming\"},{\"number\":\"02\",\"name\":\"bai\"}]}";
        ObjectMapper mapper = new ObjectMapper();
        School school = mapper.readValue(json, School.class);
        System.out.println(school);
    }
}
```

**注意：FastJson 和 Jackson 解析需要对应的实体类有完整的 `get`、`set` 方法。**


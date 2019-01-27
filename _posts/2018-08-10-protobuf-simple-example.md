---
title: Protocol Buffers 入门实例
tags: [java, protobuf, distributed]
---

### Protocol Buffers 简介

Protocol Buffers（简称 protobuf）是谷歌的一项技术，用于将结构化的数据序列化、反序列化，经常用于网络传输。

### 为什么需要 Protocol Buffers

The example we're going to use is a very simple "address book" application that can read and write people's contact details to and from a file. Each person in the address book has a name, an ID, an email address, and a contact phone number.

How do you serialize and retrieve structured data like this? There are a few ways to solve this problem:

- Use Java Serialization. This is the default approach since it's built into the language, but it has a host of well-known problems (see Effective Java, by Josh Bloch pp. 213), and also doesn't work very well if you need to share data with applications written in C++ or Python.
- You can invent an ad-hoc way to encode the data items into a single string – such as encoding 4 ints as "12:3:-23:67". This is a simple and flexible approach, although it does require writing one-off encoding and parsing code, and the parsing imposes a small run-time cost. This works best for encoding very simple data.
- Serialize the data to XML. This approach can be very attractive since XML is (sort of) human readable and there are binding libraries for lots of languages. This can be a good choice if you want to share data with other applications/projects. However, XML is notoriously space intensive, and encoding/decoding it can impose a huge performance penalty on applications. Also, navigating an XML DOM tree is considerably more complicated than navigating simple fields in a class normally would be.

Protocol buffers are the flexible, efficient, automated solution to solve exactly this problem. With protocol buffers, you write a `.proto` description of the data structure you wish to store. From that, the protocol buffer compiler creates a class that implements automatic encoding and parsing of the protocol buffer data with an efficient binary format. The generated class provides getters and setters for the fields that make up a protocol buffer and takes care of the details of reading and writing the protocol buffer as a unit. Importantly, the protocol buffer format supports the idea of extending the format over time in such a way that the code can still read data encoded with the old format.

### 简要使用

- 定义数据结构文件（`addressbook.proto`）

  ```protobuf
  syntax = "proto2";
  
  option java_package = "com.example.tutorial";
  option java_outer_classname = "AddressBookProtos";
  
  message Person {
      required string name = 1;
      required int32 id = 2;
      optional string email = 3;
  
      enum PhoneType {
          MOBILE = 0;
          HOME = 1;
          WORK = 2;
      }
  
      message PhoneNumber {
          required string number = 1;
          optional PhoneType type = 2 [default = HOME];
      }
  
      repeated PhoneNumber phones = 4;
  
  }
  
  message AddressBook {
      repeated Person people = 1;
  }
  ```

- 使用 protoc 编译

  如果不想自己编译的话，直接去 Github 上下载二进制文件包 [protoc-3.6.1-win32.zip](https://github.com/google/protobuf/releases/download/v3.6.1/protoc-3.6.1-win32.zip) 。

  解压后包里的 protoc.exe 就是用来编译 `.proto` 的程序，进入到 protoc.exe 所在目录下，命令行执行：

  ```powershell
  protoc --java_out=$output_dir $src_dir/addressbook.proto
  ```

  `ouput_dir` 是要输出的目录，`scr_dir` 是 `addressbook.proto` 所在的目录。

- 使用 Java protocol buffer API  读写 message

  - 创建 Maven 项目，添加依赖

    ```xml
    <dependency>
        <groupId>com.google.protobuf</groupId>
        <artifactId>protobuf-java</artifactId>
        <version>2.5.0</version>
    </dependency>
    ```

  - 把上一步编译生成的 java 文件放入到项目中

  - 编写代码来读写 message 数据

    Writer:

    ```java
    import java.io.FileOutputStream;
    
    /**
     * @author: siyunfei
     * @date: 2018/8/10 11:37
     */
    public class MessageWriter {
        public static void main(String[] args) {
            try {
                AddressBookProtos.Person.PhoneNumber.Builder phoneNumber = AddressBookProtos.Person.PhoneNumber.newBuilder();
                phoneNumber.setType(AddressBookProtos.Person.PhoneType.HOME).setNumber("18271633177");
    
                AddressBookProtos.Person.Builder person = AddressBookProtos.Person.newBuilder();
                person.setId(1).setName("xiaobai").addPhones(phoneNumber);
    
                AddressBookProtos.AddressBook.Builder addressBook = AddressBookProtos.AddressBook.newBuilder();
                addressBook.addPeople(person);
    
                addressBook.build().writeTo(new FileOutputStream("d:\\proto-buf.data"));
            } catch (Exception e) {
                e.printStackTrace();
            }
    
        }
    }
    ```

    Reader:

    ```java
    import java.io.FileInputStream;
    
    /**
     * @author: siyunfei
     * @date: 2018/8/10 11:50
     */
    public class MessageReader {
        public static void main(String[] args) {
            try {
                AddressBookProtos.AddressBook addressBook = AddressBookProtos.AddressBook.parseFrom(new FileInputStream("d:\\proto-buf.data"));
                System.out.println(addressBook);
                System.out.println(addressBook.getPeople(0).getPhones(0).getNumber());
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    ```

---

### 参考资料

- [Protocol Buffer Basics: Java](https://developers.google.com/protocol-buffers/docs/javatutorial)


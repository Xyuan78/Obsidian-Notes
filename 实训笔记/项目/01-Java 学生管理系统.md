功能如下：
![[Pasted image 20261010133237.png]]

（1）学生类包括：序号，学号，姓名，班级编号，Java成绩，Vue成绩，HTML成绩，总分等字段。

（2）使用集合类中ArrayList保存数据，

（3）使用序列化与反序列化方式，保存对象。第一次启动项目，手动添加学生信息，第二次启动后，通过读取序列化文件，初始化系统所需数据。

（3）正确使用三层开发的思想，创建对应的包。

（4）上述8个功能，每一个功能都要有运行截图。


## 1. 创建文件
![[Pasted image 20261010133142.png]]

## 2. 基础步骤
#### 1. 修改pom.xml
[[00 .创建Java项目后的首先操作]]
#### 2. 实体层Student
开头加入必须的[[00 .创建Java项目后的首先操作]]，同时添加属性
#### 3. 异常层
自定义异常
#### 4.  StudentDao
进行ArrayList和序列化
```java
//序列化与反序列化  
   //判断是第一次启动还是之后启动  
    public boolean isFirstStart() {  
        return new File("students.dat").exists();  
    }  
    //序列化：把整个ArrayList保存到文件  
    public void save(){  
        File file = new File("students.dat");  
        File parent = file.getParentFile();//获取父目录  
        try{  
            if(parent != null && !parent.exists()){  
                parent.mkdirs();//创建父目录  
            }  
        }catch (Exception e){  
            e.printStackTrace();  
        }  
        //序列化,使用try-with-resources  
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file))) {  
            oos.writeObject(students);  
            System.out.println("保存成功");  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
        System.out.println("保存成功");  
    }  
    //反序列化：从文件加载整个ArrayList  
    public void load() {  
        File file = new File("students.dat");  
        if (file.exists()) {  
            try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file))) {  
                students = (ArrayList<Student>) ois.readObject();  
                System.out.println("加载成功");  
            } catch (IOException | ClassNotFoundException e) {  
                e.printStackTrace();  
            }  
        }else {  
            System.out.println("文件不存在，无法加载数据");  
        }  
    }  
}
```

#### 5. 业务逻辑层StudentService

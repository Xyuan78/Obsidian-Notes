# 0. 项目设计知识点
- 封装JDBC[[06. JDBC]]
- 封装DAO[[02-Java项目 饿了么#^1b3569]]
- 领域模型中的实体类(model包下)
- 增删改查操作[[06. JDBC#^6986b0]]
- 多条件模糊查询
- JDBC事务管理
- 表的主外键关系
# 1. 数据库设计
### 1.1DB一览表

|     |          |      |                  |
| --- | -------- | ---- | ---------------- |
| No  | 表名称      | 中文名  | 说明               |
| 1   | business | 商家表  | 存储所有商家信息         |
| 2   | food     | 食品表  | 存储每个商家所拥有的所有食品信息 |
| 3   | admin    | 管理员表 | 存储管理员信息          |

### 1.2.表结构

约束类型标识：

- PK：primary key 主键
- FK：foreign key 外键
- NN：not null 非空
- UQ：unique 唯一索引
- AI：auto increment 自增

#### 1.2.1.business（商家表）

|   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|
|No|字段名|数据类型|size|默认値|约束|说明|
|1|businessId|int|||PK、AI、NN|商家编号|
|2|password|varchar|20||NN|密码|
|3|businessName|varchar|40||NN|商家名称|
|4|businessAddress|varchar|50|||商家地址|
|5|businessExplain|varchar|40|||商家介绍|
|6|starPrice|decimal|(5,2)|0.00||起送费|
|7|deliveryPrice|decimal|(5,2)|0.00||配送费|

#### 1.2.2.food（食品表）

|   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|
|No|字段名|数据类型|size|默认値|约束|说明|
|1|foodId|int|||PK、AI、NN|食品编号|
|2|foodName|varchar|30||NN|食品名称|
|3|foodExplain|varchar|30|||食品介绍|
|4|foodPrice|decimal|(5,2)||NN|食品价格|
|5|businessId|int|||FK、NN|所属商家编号|

#### 1.2.3.admin（管理员表）

|   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|
|No|字段名|数据类型|size|默认値|约束|说明|
|1|adminId|int|||PK、AI、NN|管理员编号|
|2|adminName|varchar|20||NN、UQ|管理员名称|
|3|password|varchar|20||NN|密码|

# 2. 项目总览
```text
src
└── main
    └── java
        └── com.elm.manager
            ├── dao(数据访问对象)
            │   ├── AdminDao
            │   ├── BusinessDao
            │   ├── DBUtil
            │   └── FoodDao
            ├── model(数据模型)
            │   ├── entity(实体类)
            │   │   ├── Admin
            │   │   ├── Business
            │   │   └── Food
            │   └── vo(视图对象)
            └── service(业务逻辑层)
                ├── impl(实现层)
                │   └── FoodServiceImpl
                └── FoodService
```

| model  | 会放各种数据载体                              |
| ------ | ------------------------------------- |
| vo     | **专门用于前端（视图层）展示或接口返回的数据载体**。自己构建需要的对象 |
| entity | **与数据库表一一映射**。                        |
|        |                                       |

# 3. 项目搭建
## 3.1 构建项目
### 1. 根据数据库中的表结构，设置包和类

在com.elm.manager.model.entity;包下创建三个实体类，根据三个表：
Admin，business，Food

| model  | 会放各种数据载体                              |
| ------ | ------------------------------------- |
| vo     | **专门用于前端（视图层）展示或接口返回的数据载体**。自己构建需要的对象 |
| entity | **与数据库表一一映射**。                        |

## 3.1 连接数据库
在dao包下创建设置数据库的操作
[[06. JDBC#^124d80]] ^1b3569

[[06. JDBC#^a955f4]]
在util包下创建MyProperties
```java
package com.elm.manager.util;  
  
import java.io.IOException;  
import java.io.InputStream;  
import java.util.Properties;  
  
public class MyProperties {  
    static final Properties PROPERTIES = new Properties();  
    static {  
        //加载文件  
        InputStream inputStream = MyProperties.class.getClassLoader().getResourceAsStream("db.properties");  
        try {  
            PROPERTIES.load(inputStream);//导入配置文件  
        } catch (IOException e) {  
            throw new RuntimeException(e);  
        }  
    }  
    public static String getProperties(String key){  
        return PROPERTIES.getProperty(key,"unkown");  
    }  
}
```

# 4. 管理员端功能
管理员端功能概述：

1. 管理员登录功能
2. 查询商家列表
3. 条件搜索商家（商家名称或商家地址模糊查询）
4. 新建商家
5. 删除商家
## 4.1 管理员登录系统
### 1. Login方法
##### 1.在AdminService接口中定义login方法
```java
package com.elm.manager.service;  
  
public interface AdminService {  
    String login();//登录功能  
}
```

##### 2. 先在dao下的AdminSDao中写login方法，判断账号密码对不对
```java
//作用：实现管理员登录功能，返回登录是否成功  
public boolean login(Admin admin){  
    //登录功能，假设登录失败  
    boolean isLogin = false;  
    //构架sql语句，查询对应的结果，如果有数据集合，就是匹配的用户名与密码-登录成功，否则登录失败  
    String sql = "select * from t_admin where admin_id=? and admin_password=?";  
    try {  
        //获得连接  
        Connection connection = DBUtil.getConnection();  
        //获得数据库操作对象  
        PreparedStatement preparedStatement = connection.prepareStatement(sql);  
        // 添加参数  
        preparedStatement.setObject(1,admin.getAdminId());  
        preparedStatement.setObject(2,admin.getAdminPassword());  
        //执行sql语句。  
        ResultSet resultSet = preparedStatement.executeQuery();  
        if(resultSet.next()){  
            isLogin = true;// 合法的用户  
        }  
  
        DBUtil.closeAll(connection,preparedStatement,resultSet);  
    } catch (SQLException e) {  
        throw new RuntimeException(e);  
    } catch (ClassNotFoundException e) {  
        throw new RuntimeException(e);  
    }  
  
    return isLogin;  
  
}
```
##### 3. 在AdminServiceImpl类中，实现接口
```java
public class AdminServiceImpl implements AdminService {  
    AdminDao adminDao = new AdminDao();  
    Scanner input = new Scanner(System.in);  
    @Override  
  
    public String login() {  
        //提示用户输入  
        System.out.println("admin_id:");  
        int id = input.nextInt();  
  
        System.out.println("admin_password:");  
        String password = input.next();  
  
        //封装对象(todo:密码加密)  
        Admin admin = new Admin();  
        admin.setAdminId(id).setAdminPassword(password);  
  
        //调用dao，判断是否是一组数据  
        boolean login = adminDao.login(admin);  
  
        if (login){  
            return "登录成功";  
        }else {  
            return "登录失败";  
        }  
    }
```

## 4.2 管理员注册功能
### 1. save方法
##### 1. 在AdminService接口中定义save方法
```java
public String save();
```
##### 2. 先在dao下的AdminSDao中写save方法（与数据库连接）
```java
public int save(Admin admin){  
    int count = 0; //行数  
    String sql = "insert into t_admin values(null,?,?)";  
    Object[] params = {admin.getAdminId(),admin.getAdminName(),admin.getAdminPassword()};  
    try {  
        count = DBUtil.executeUpdate(sql,params);  
    } catch (SQLException e) {  
        throw new RuntimeException(e);  
    } catch (ClassNotFoundException e) {  
        throw new RuntimeException(e);  
    }  
    return count;  
}
```
##### 3. 在AdminServiceImpl类中，实现接口

```java
public String save() {  
    System.out.println("admin_name:");  
    String name = input.next();  
    System.out.println("admin_password:");  
    String password = input.next();  
    Admin admin = new Admin(admin.setAdminName(name).setAdminPassword(password);  );  
    int count = adminDao.save(admin);  
    if(count == 1){  
        return "管理员注册成功";  
    }  
    return "管理员注册失败";  
}
```

## 4.3 管理员修改密码
### 1. updatePassword方法
##### 1. 在AdminService接口中定义updatePassword方法
```java
String updatePassword();//修改密码
```
##### 2. 先在dao下的AdminSDao中写updatePassword方法
```java
public int updatePassword(Admin admin){  
    int count = 0;  
    String sql = "update t_admin set admin_password = ?where admin_id = ?";  
    Object[] params = {admin.getAdminPassword(), admin.getAdminId()};  
    try {  
        count = DBUtil.executeUpdate(sql,params);  
    } catch (SQLException e) {  
        throw new RuntimeException(e);  
    } catch (ClassNotFoundException e) {  
        throw new RuntimeException(e);  
    }  
    return count;  
}
```
##### 3. 在AdminServiceImpl类中，实现接口
```java

```
## 4.4 管理员退出系统
直接输入4即可退出
# 5. 商家端功能
## 5.1 显示商家列表
### 1. getAll方法
##### 1. 在BusinessService接口中定义getAll方法
```java
public interface BusinessService {  
  
    //新建商家  
    public String save();  
    // 显示商家列表  
    public void list();  
}
```
##### 2. 在dao包下的BusinessDao类中实现getAll方法（与数据库连）
```java
public List<Business> getAll(){  
    List<Business> list = new ArrayList<>();  
    String sql ="select * from t_bussiness";  
    Connection connection =null;  
    PreparedStatement preparedStatement = null;  
    ResultSet resultSet = null;  
    try {  
        connection = DBUtil.getConnection();  
        preparedStatement = connection.prepareStatement(sql);  
        resultSet = preparedStatement.executeQuery();  
  
        //封装集合  
        while(resultSet.next()){  
            list.add(new Business(  
                    resultSet.getInt(1),  
                    resultSet.getString(2),  
                    resultSet.getString(3),  
                    resultSet.getString(4),  
                    resultSet.getString(5),  
                    resultSet.getDouble(6),  
                    resultSet.getDouble(7)  
            ));  
        }  
  
    } catch (SQLException e) {  
        throw new RuntimeException(e);  
    } catch (ClassNotFoundException e) {  
        throw new RuntimeException(e);  
    }finally {  
        try {  
            DBUtil.closeAll(connection,preparedStatement,resultSet);  
        } catch (SQLException e) {  
            throw new RuntimeException(e);  
        }  
    }  
    return list;  
}
```
##### 3. 在BusinessServiceImpl中实现方法（业务逻辑层）
```java
BusinessDao businessDao = new BusinessDao();  
Scanner input = new Scanner(System.in);  
public void list() {  
    List<Business> list = businessDao.getAll();  
    System.out.println("===========================");  
    list.forEach(System.out::println);  
    System.out.println("===========================");  
}
```
## 5.2 条件搜索商家（商家名称或商家地址模糊查询）
### 1. search方法
##### 1. 在BusinessService接口中增加search方法
```java
List<Business> search(String keyword);//条件搜索商家
```
##### 2. 在BusinessDao类中实现getByNameOrAddress方法
```java
public List<Business> search(String keyword){  
    List<Business> list = new ArrayList<>();  
    String sql ="select * from t_business where business_name like ? or business_address like ?";  
    Object[] params={keyword,keyword};  
    Connection connection = null;  
    PreparedStatement preparedStatement = null;  
    ResultSet resultSet = null;  
    try {  
        connection = DBUtil.getConnection();  
        preparedStatement = connection.prepareStatement(sql);  
        preparedStatement.setString(1, (String) params[0]);  
        preparedStatement.setString(2, (String) params[1]);  
        resultSet = preparedStatement.executeQuery();  
        while (resultSet.next()) {  
            list.add(new Business(  
                    resultSet.getInt(1),  
                    resultSet.getString(2),  
                    resultSet.getString(3),  
                    resultSet.getString(4),  
                    resultSet.getString(5),  
                    resultSet.getDouble(6),  
                    resultSet.getDouble(7)  
            ));  
        }  
    } catch (SQLException e) {  
        throw new RuntimeException(e);  
    } catch (ClassNotFoundException e) {  
        throw new RuntimeException(e);  
    } finally {  
        try {  
            DBUtil.closeAll(connection, preparedStatement, resultSet);  
        } catch (SQLException e) {  
            throw new RuntimeException(e);  
        }  
    }  
    return list;  
}
```
##### 3. ##### 在BusinessServiceImpl中实现search方法
```java

```
## 5.3 新建商家
### 1. save方法
##### 1. 在BusinessService接口中定义save方法
```java
//新建商家  
public String save();
```
##### 2. 在dao包下的BusinessDao类中实现save方法（与数据库连）


```java
//保存商家  
public int save(Business business){  
    int count = 0;//返回影响的行数  
    String sql="insert into t_bussiness values (null,?,?,?,?,?,?)";  
    Object[] params={business.getBusinessPassword(),business.getBusinessName(),  
            business.getBusinessAddress(),business.getBusinessExplain(),business.getStartPrice(),business.getDeliveryPrice()};  
    try {  
        count = DBUtil.executeUpdate(sql,params);  
    } catch (SQLException e) {  
        throw new RuntimeException(e);  
    } catch (ClassNotFoundException e) {  
        throw new RuntimeException(e);  
    }  
    return count;  
  
}
```
##### 3. 在BusinessServiceImpl中实现方法（业务逻辑层）
```java
BusinessDao businessDao = new BusinessDao();  
Scanner input = new Scanner(System.in);  
@Override  
public String save() {  
    int count = 0;  
    System.out.println("pwd:");  
    String pwd = input.next();  
  
    System.out.println("name:");  
    String name = input.next();  
  
    System.out.println("address:");  
    String address = input.next();  
  
    System.out.println("explain:");  
    String explain = input.next();  
  
    System.out.println("startPrice:");  
    double startPrice = input.nextDouble();  
  
    System.out.println("deliveryPrice:");  
    double deliveryPrice = input.nextDouble();  
  
    //封装（todo：数据校验，商家名称唯一，价格大于0）  
    Business business = new Business(null,pwd,name,address,explain,startPrice,deliveryPrice);  
  
    count = businessDao.save(business);  
  
    if(count==1){  
        return "商家信息添加成功";  
    }else{  
        return "商家信息添加失败";  
    }  
  
}
```
## 5.4 删除商家
### 1. delete方法
##### 1. 在BusinessService接口中定义delete方法
```java
String delete();
```
##### 2. 先在dao下的BusinessSDao中写delete方法，
```java
public int delete(Integer business_id){  
    int count= 0;  
    String sql = "delete from t_business where business_id = ?";  
    Object[] params = {business_id};  
    try {  
        count = DBUtil.executeUpdate(sql, params);  
    } catch (SQLException e) {  
        throw new RuntimeException(e);  
    } catch (ClassNotFoundException e) {  
        throw new RuntimeException(e);  
    }  
    return count;  
}
```
##### 3. 在AdminServiceImpl类中，实现接口
```java
public String delete() {  
    System.out.println("请输入要删除的商家ID：");  
    int business_id = input.nextInt();  
    int count = businessDao.delete(business_id);  
    if (count == 1) {  
        return "商家信息删除成功";  
    } else {  
        return "商家信息删除失败";  
    }     
}
```
# 6. 菜品端功能
## 6.1 显示菜品列表
### 1. list方法
##### 1. 在FoodService接口中定义list方法
```java
public void list();
```
##### 2. 先在dao下的FoodDao中写getAll方法
```java
public List<Food> getAll(){  
    List<Food> list = new ArrayList<>();  
    String sql ="select * from t_food";  
    Connection connection =null;  
    PreparedStatement preparedStatement = null;  
    ResultSet resultSet = null;  
    try {  
        connection = DBUtil.getConnection();  
        preparedStatement = connection.prepareStatement(sql);  
        resultSet = preparedStatement.executeQuery();  
        while (resultSet.next()) {  
            list.add(new Food(  
                    resultSet.getInt(1),  
                    resultSet.getString(2),  
                    resultSet.getString(3),  
                    resultSet.getDouble(4)  
            ));  
        }  
    } catch (SQLException e) {  
        throw new RuntimeException(e);  
    } catch (ClassNotFoundException e) {  
        throw new RuntimeException(e);  
    } finally {  
        try {  
            DBUtil.closeAll(connection, preparedStatement, resultSet);  
        } catch (SQLException e) {  
            throw new RuntimeException(e);  
        }  
    }  
    return list;  
}
```
##### 3. 在FoodServiceImpl类中，实现接口
```java
@Override  
//获取所有食物信息  
public void list() {  
    List<Food> list = foodDao.getAll();  
    System.out.println("===========================");  
    list.forEach(System.out::println);  
    System.out.println("===========================");  
}
```
## 6.2 条件搜索菜品（菜品名称模糊查询）
### 1. search方法
##### 1. 在FoodService接口中定义search方法
```java
public void search();
```
##### 2. 先在dao下的FoodDao中写login方法
```java
public List<Food> getByName(String keyword){  
    List<Food> list = new ArrayList<>();  
    String sql = "select * from t_food where food_name like ?";  
    Connection connection = null;  
    PreparedStatement preparedStatement = null;  
    ResultSet resultSet = null;  
    try {  
        connection = DBUtil.getConnection();  
        preparedStatement = connection.prepareStatement(sql);  
        preparedStatement.setObject(1, "%" + keyword + "%");  
        resultSet = preparedStatement.executeQuery();  
        while (resultSet.next()) {  
            list.add(new Food(  
                    resultSet.getInt(1),  
                    resultSet.getString(2),  
                    resultSet.getString(3),  
                    resultSet.getDouble(4)  
            ));  
        }  
    } catch (SQLException e) {  
        throw new RuntimeException(e);  
    } catch (ClassNotFoundException e) {  
        throw new RuntimeException(e);  
    } finally {  
        try {  
            DBUtil.closeAll(connection, preparedStatement, resultSet);  
        } catch (SQLException e) {  
            throw new RuntimeException(e);  
        }  
    }  
    return list;  
}
```
##### 3. 在AdminServiceImpl类中，实现接口
```java
@Override  
//模糊查询  
public void search() {  
    System.out.println("请输入商家名称或商家地址关键词：");  
    List<Food> list = foodDao.getByName(input.next());  
    System.out.println("===========================");  
    list.forEach(System.out::println);  
    System.out.println("===========================");  
}
```
## 6.3 修改菜品
### 1. update方法
##### 1. 在FoodService接口中定义update方法
```java
String update();
```
##### 2. 先在dao下的FoodDao中写update方法
```java

```
##### 3. 在FoodServiceImpl类中，实现接口
## 6.4 修改菜品
### 1. save方法
##### 1. 在FoodService接口中定义save方法
```java
//新建菜品  
public String save();
```
##### 2. 先在dao下的FoodDao中写login方法
```java
//保存食物信息  
public int save(Food food){  
    int count = 0;//返回影响的行数  
    String sql="insert into t_food values (null,?,?,?,?)";  
    Object[] params={food.getFoodName(), food.getFoodExplain(), food.getFoodPrice(), food.getBusinessId()};  
    try {  
        count = DBUtil.executeUpdate(sql,params);  
    } catch (SQLException e) {  
        throw new RuntimeException(e);  
    } catch (ClassNotFoundException e) {  
        throw new RuntimeException(e);  
    }  
    return count;  
  
}
```
##### 3. 在FoodServiceImpl类中，实现接口
```java
@Override  
//保存食物信息  
public String save() {  
    int count = 0;  
    System.out.println("foodName:");  
    String name = input.next();  
    System.out.println("foodExplain:");  
    String explain = input.next();  
    System.out.println("foodPrice:");  
    double price = input.nextDouble();  
    System.out.println("businessId:");  
    int businessId = input.nextInt();  
    Food food = new Food(null, name, explain, price, businessId);  
    count = foodDao.save(food);  
    if (count == 1) {  
        return "菜品添加成功";  
    } else {  
        return "菜品添加失败";  
    }  
}
```
## 6.5 删除菜品
### 1. delete方法
##### 1. 在FoodService接口中定义delete方法
```java

```
##### 2. 先在dao下的FoodDao中写delete方法
##### 3. 在FoodServiceImpl类中，实现接口
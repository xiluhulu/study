# Jdbc

大致步骤

1. JDBC开始需要对工程进行导对应的JDBC驱动包
2. 使用驱动链接数据库
3. 操作数据
4. 关闭链接

## 数据库的链接

- 步骤：（适用方法一、二，三、四不适用）
  1. 获取Driver驱动实现类
  2. 设置数据库链接使用的url（固定操作）
     - url（ jdbc:mysql://localhost:3306/test?characterEncoding=utf-8 ）
       1. jdbc:mysql( 协议 )
       2. localhost（ IP地址 ）
       3. 3306 （端口号）
       4. test （ 名为test数据库 ）
       5. characterEncoding=utf-8 （ 编码格式 ）
  3. 设置数据库链接使用的用户名、密码（固定操作）
     1. 获取Properties实现类
     2. 利用实现类设置用户名（user）、密码（password）
     3. 以键值对的方式告诉数据库
  4. 使用驱动链接数据库
  5. 查看链接是否成功

有四个方式可用于链接，只使用最终版

方式一：

```java
//    链接测试：方式一
    @Test
    public void testConnection1() throws SQLException {
//        java.sql下的包
//        新建jdbc驱动
        Driver driver = new com.mysql.jdbc.Driver();
        String url = "jdbc:mysql://localhost:3306/test?characterEncoding=utf-8";
//        用于设置属性，以键值对的格式存在，可设置多个，键与值都必须为字符串
//        在这主要用于设置数据库的用户名、密码
        Properties info = new Properties();
        info.setProperty("user","root");
        info.setProperty("password","123456");
//        使用驱动器建立链接参数分别为url（数据库的地址字符串）、info（账号密码对象）
        Connection connect = driver.connect(url, info);
//        输出该对象的内存地址即为链接成功，也就是不等于null
        System.out.println(connect);
    }
```

方式二：

```java
    /*
      链接测试：方式二
          与方式一相比较
              具有更好的可移植性，不使用第三方api
              使用了反射，获取Driver实现类对象（也就是说，第三方api更变时，不需要更换实例对象，只需要更改第三方api反射的路径）
     */
    @Test
    public void testConnection2() throws Exception {
//        1、使用反射，获取Driver实现类对象
        Class aClass = Class.forName("com.mysql.jdbc.Driver");
        Driver driver = (Driver) aClass.newInstance();//获取空参构造器
//        2、设置数据库链接的用户名、密码
        Properties info = new Properties();
        info.setProperty("user","root");
        info.setProperty("password","123456");
//        3、设置数据库链接的url
        String url = "jdbc:mysql://localhost:3306/test?characterEncoding=utf-8";
//        4、 使用驱动链接数据库
        Connection connect = driver.connect(url, info);
//        5、查看是否链接成功
        System.out.println(connect);

    }
```

方式三：

```java
    /*
      链接测试：方式三
          与方式二相比较
              使用DriverManager替换Driver获取链接
     */
    @Test
    public void testConnection3() throws Exception{
//        1、使用反射，获取Driver实现类对象
        Class aClass = Class.forName("com.mysql.jdbc.Driver");
        Driver driver = (Driver) aClass.newInstance();//获取空参构造器
//        2、提供url、用户名、密码三个基本信息（不使用Properties类，更方便）
        String url = "jdbc:mysql://localhost:3306/test?characterEncoding=utf-8";
        String user = "root";
        String password = "123456";

//        3、使用DriverManager（驱动管理器）注册驱动
        DriverManager.registerDriver(driver);
//        4、使用驱动管理器获取链接
        Connection connection = DriverManager.getConnection(url, user, password);
//        5、查看链接情况
        System.out.println(connection);
    }
```

方式四：

```java
    /*
  链接测试：方式四
      与方式三相比较
          优化了一些代码
 */
    @Test
    public void testConnection4() throws Exception{
//        1、提供url、用户名、密码三个基本信息（不使用Properties类，更方便）
        String url = "jdbc:mysql://localhost:3306/test?characterEncoding=utf-8";
        String user = "root";
        String password = "123456";

//        2、使用反射，获取Driver实现类对象(加载Driver驱动)
        //保留是应为只有Mysql不需要加载驱动，但是其他的需要
        Class.forName("com.mysql.jdbc.Driver");

//            相对于方式三，可以省略以下操作
        //Driver driver = (Driver) aClass.newInstance();//获取空参构造器
//        3、使用DriverManager（驱动管理器）注册驱动
       // DriverManager.registerDriver(driver);
        /*因为在mysql的Driver实现类中，声明如下操作会默认执行
           static {
                try {
                    DriverManager.registerDriver(new Driver());
                } catch (SQLException var1) {
                    throw new RuntimeException("Can't register driver!");
                }
            }
         */
//        3、使用驱动管理器获取链接
        Connection connection = DriverManager.getConnection(url, user, password);
//        4、查看链接情况
        System.out.println(connection);
    }
```

方式五：（终版）

```java
        /*
        链接测试：方式五
          与前面的相比较
              1、将数据库链接需要的4个基本信息声明在配置文件中，通过读取配置文件获取所有的链接信息
              2、实现了数据与代码分离
              3、实现了解耦
              4、如果需要修改配置文件信息，可以避免程序重新打包
    */
    @Test
    public void testConnection5() throws Exception {
        //1.加载配置文件
//        获取系统类的加载器Connection01.class.getClassLoader()
//        指明文件名getResourceAsStream（文件名）
//        is为加载的所返回的字节流
        InputStream is =  Connection01.class.getClassLoader().getResourceAsStream("jdbc.properties");
//        获取Properties实例化对象
        Properties pros = new Properties();
//        将字节流放入Properties实例化对象中加载用于读取
//        加载字节流（相当于读卡器）
        pros.load(is);
        //2.读取配置信息（一个一个利用键名读取值）
        String user = pros.getProperty("user");
        String password = pros.getProperty("password");
        String url = pros.getProperty("url");
        String driverClass = pros.getProperty("driverClass");
        //3.加载驱动
        Class.forName(driverClass);
        //4.获取连接
        Connection conn = DriverManager.getConnection(url,user,password);
        System.out.println(conn);
    }
```



配置文件（jdbc.properties）：

```properties
user=root
password=abc123
#characterEncoding=utf-8用于设置编码格式
url=jdbc:mysql://localhost:3306/test？characterEncoding=utf-8
driverClass=com.mysql.jdbc.Driver
```

## 数据库的操作

- 操作分为，增删改与查两大类
- 调用Connection对象的PreparedStatement（String sql）方法获取PreparedStatement对象
- 使用了PreparedStatement对象进行数据库操作，此方法不会有sql注入风险。
- 当然也需要进行链接
- 步骤如下：（针对增删改）
  1. 获取数据库的链接
  2. 预编译sql语句，返回PreparedStatement的实例
  3. 填充占位符
  4. 执行操作
  5. 关闭资源

### 数据库的增加操作：

```java
    //使用此PreparedStatement方法进行对数据库的操作
//    向customers表中添加数据
    @Test
    public void testInsert()    {
        Connection conn = null;
        PreparedStatement ps = null;
//        因有了资源的关闭，所以不使用throws抛出异常
        try {
//        1、获取链接、封装了的方法
//        Connection conn = Connection01.getConn();
//        1、获取链接，相比方式五有点小改动(Connection01.class.getClassLoader()与ClassLoader效果一样)
//        InputStream is =  Connection01.class.getClassLoader().getResourceAsStream("jdbc.properties");
            InputStream is = ClassLoader.getSystemResourceAsStream("jdbc.properties");
            Properties pros = new Properties();
            pros.load(is);
            String user = pros.getProperty("user");
            String password = pros.getProperty("password");
            String url = pros.getProperty("url");
            String driverClass = pros.getProperty("driverClass");
            Class.forName(driverClass);
            conn = DriverManager.getConnection(url,user,password);
//        System.out.println(conn);
//        2、获取prepareStatement实例
//        官方解释，预编译sql语句，返回prepareStatement实例        以下都为sql预编译包括步骤3
//          prepareStatement可以理解为信使，也就是将sql传递给数据库的快递员
            String sql="insert into customers(name,email,birth)value (?,?,?)";//？（问好）：占位符也就是参数
            ps = conn.prepareStatement(sql);
//        3、填充占位符，利用prepareStatement实例设置数据库对应数据类型的值
//        参数1：占位符的索引
//        参数2：占位符的值
            ps.setString(1,"欧阳阿斯弗");
            ps.setString(2,"asd@qwe.com");
//        格式化事件
            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
            java.util.Date parse = simpleDateFormat.parse("2022-04-16");
//        将格式化的时间转成时间戳
            ps.setDate(3,new Date(parse.getTime()));
//        4、执行sql操作
            ps.execute();
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//        5、关闭资源
            try {
//                避免没获取到链接还继续调用资源关闭方法
                if (ps!=null)
                    ps.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try {
//                避免没获取到链接还继续调用资源关闭方法
                if (conn!=null)
                    conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

    }
//到这
```

### 对数据库链接与资源挂壁的封装：

- 将数据库链接的方法五封装成一个getConn()方法
- 将资源关闭操作封装成closeResource(Connection conn, PreparedStatement ps)方法，参数均为java.sql下的包下的类型

工具类的封装：

```java
//操作数据库的工具类
public class JDBCUtils {
//    获取链接
    public static Connection getConn() throws Exception {
        //1.加载配置文件
//        获取系统类的加载器Connection01.class.getClassLoader()
//        指明文件名getResourceAsStream（文件名）
//        is为加载的所返回的字节流
        InputStream is =  Connection01.class.getClassLoader().getResourceAsStream("jdbc.properties");
//        获取Properties实例化对象
        Properties pros = new Properties();
//        将字节流放入Properties实例化对象中加载用于读取
//        加载字节流（相当于读卡器）
        pros.load(is);
        //2.读取配置信息（一个一个利用键名读取值）
        String user = pros.getProperty("user");
        String password = pros.getProperty("password");
        String url = pros.getProperty("url");
        String driverClass = pros.getProperty("driverClass");
        //3.加载驱动
        Class.forName(driverClass);
        //4.获取连接
        Connection conn = DriverManager.getConnection(url,user,password);
//        System.out.println(conn);
        return conn;
    }
/*----------------------------------------------------------------*/
//    关闭资源
//    下面自定义的资源关闭方法中的两个参数均来自java.sql下的包
    public static void closeResource(Connection conn, PreparedStatement ps){
        //        5、关闭资源
        try {
//                避免没获取到链接还继续调用资源关闭方法
            if (ps!=null)
                ps.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
//                避免没获取到链接还继续调用资源关闭方法
            if (conn!=null)
                conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

### 使用上方工具类对数据库的修改操作：

```java
//    修改customers表中的一条数据
    @Test
    public void testUpdate()   {
        Connection conn = null;
        PreparedStatement ps = null;
        try {
//        1、获取数据库的链接【调用自己封装好的工具类】
            conn = JDBCUtils.getConn();
//        2、预编译sql语句，返回PreparedStattement的实例
            String sql="update  customers set name = ? where id =?";
            ps = conn.prepareStatement(sql);
//        3、填充占位符,设为Object就不用过多的考虑类型，从而方便一点
            ps.setObject(1,"斯金纳*七武士");
            ps.setObject(2,22);
//        4、执行
            ps.execute();
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//        5、资源关闭【调用自己封装好的工具类】
            JDBCUtils.closeResource(conn,ps);
        }
    }
```

### 通用的增删改操作：【可封装成工具类】

```java
//    通用的增删改方法
//    sql中占位符的个数与可变形参的长度相同，相当于当前sql中有多少个占位符就在sql参数的后面添加用于填充多少个占位符的参数
    public void update(String sql,Object ...args)   {
        Connection conn = null;
        PreparedStatement ps = null;
        try {
//            1、获取链接
            conn = JDBCUtils.getConn();
//            2、预编译sql语句，返回PrepareStatement实例
            ps = conn.prepareStatement(sql);
//            3、填充占位符，利用可变形参的长度进行填充
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//            4、执行
            ps.execute();
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//            5、关闭资源
            JDBCUtils.closeResource(conn,ps);
        }
    }
```

### 使用通用的增删改封装后的方法进行增删改：

```java
//    测试通用增删改操作
    @Test
    public void testCommonUpdate(){
//        调用封装后的方法实现删除操作
        String sql="delete from customers where id = ?";
        update(sql,19);
//        调用封装后的方法实现修改操作【此修改的表名中为关键字，需要使用着重号引起来，否则会报错】
        String sql1="update `order` set order_name = ? where order_id = ?";
        update(sql1 ,"pp","4");
//        调用封装后的方法实现增加操作
        String sql2="insert into customers(name,email)value (?,?)";
        update(sql2,"圣骑士*丹佛斯","sqs@sqs.com");
    }
```



### 数据库的查询操作：

- 不同与增删改，查询操作有返回的结果集，也就是查询结果
- 结果集，需要使用javaBean对其进行暂时的存储并用与响应输出
  - javaBean的类名对应表的名字，属性名对应字段名
  - 包含空参构造器，有参构造器，get与set方法，toString方法
- 对数据库的执行方法不一样， 

#### 查询操作（简单的查询一条数据）：

查询的方法操作：

```java
//    对Customer表进行单条数据查询
    @Test
    public void testQuery1(){
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet resultSet = null;
        try {
//        1、获取链接
            conn = JDBCUtils.getConn();
//        预编译sql语句
            String sql="select id ,name , email,birth from customers where  id = ?";
            ps = conn.prepareStatement(sql);
//        填充占位符
            ps.setObject(1,2);
//        执行并返回结果集
            resultSet = ps.executeQuery();
//        处理结果集
//        判断结果集的下一条是否有数据，如果有数据返回true，并指针下移，如果返回false，指针不下移
            if(resultSet.next()){
    //            获取当前这条数据的各个字段值
//                相当于获取各个字段的值
                int id =resultSet.getInt(1);
                String name =resultSet.getString(2);
                String email = resultSet.getString(3);
                Date birth = resultSet.getDate(4);
    //            方式一【不推荐】
                System.out.println("id="+id+"name="+name+"email="+email+"birth="+birth);
    //            方式二【不推荐】
                Object[] data = new Object[]{id,name,email,birth};
                for (int i = 0 ; i < data.length;i++){
                    System.out.print(data[i]+" ");
                }
    //            方式三：将数据封装为一个对象【推荐】
                Customer customer = new Customer(id, name, email, birth);
                System.out.println(customer);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//         关闭资源【新封装的关闭方法】
//            此关闭资源的方法相对于增删改多了一个对结果集的关闭
            JDBCUtils.closeResource(conn,ps,resultSet);
        }

    }
```

用于处理结果集的JavaBean：

```java
/*
    ORM编程思想（object relational mapping）
    一个表对应一个Java类
    表中的一条记录对应java类的一个对象
    表中的一个字段对应java类的一个属性
 */
public class Customer {
    private int id;
    private String name;
    private String email;
    private Date birth;
    
    public Customer() {}
    public Customer(int id, String name, String email, Date birth) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.birth = birth;
    }
    public int getId() {return id;}
    public void setId(int id) {this.id = id;}
    public String getName() {return name;}
    public void setName(String name) {this.name = name;}
    public String getEmail() {return email;}
    public void setEmail(String email) {this.email = email;}
    public Date getBirth() {return birth;}
    public void setBirth(Date birth) {this.birth = birth;}
    @Override
    public String toString() {
        return "Customer{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", email='" + email + '\'' +
                ", birth=" + birth +
                '}';
    }
}
```



#### 查询的资源关闭操作【将此方法封装到JDBCUtils工具类中】

```java
//    关闭资源，多了个结果集作为参数（适用于查询，不适用增删改）
    public static void closeResource(Connection conn, PreparedStatement ps,ResultSet rs){
        //        5、关闭资源
        //                避免没获取到链接还继续调用资源关闭方法
        try {
//            关闭预编译资源
            if (ps!=null)
                ps.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
//            关闭链接
            if (conn!=null)
                conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
//            关闭结果集资源
            if (rs!=null)
                rs.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
```

#### 查询的通用操作的封装（无另起别名）【只对一个表通用】：

- 查询操作中使用到了映射
- 使用了通过结果集获取元数据的方法
  - getMetaData()：通过结果集获取元数据
- 元数据可以获取字段名，结果集中的字段名【也就是数据库表的字段名】的列数【总数】
  - getColumnName(int)：通过结果集获取字段名，参数为1以上
  - getColumnCount()：通过结果集获取结果集中的列数

```java
//    针对customer表的通用查询操作
    public Customer customerForCommonQuery(String sql , Object ...args)   {
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
//        1、获取链接
            conn = JDBCUtils.getConn();
//        2、预编译sql语句
            ps = conn.prepareStatement(sql);
//        3、填充占位符
            for (int i = 0 ; i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//        4、执行操作，并获取结果集
            rs = ps.executeQuery();
//        5、获取结果集的元数据，也就是一个参数的数据类型和变量名，相当于（int id = 2）int于id是数据2的元数据
            ResultSetMetaData rsmd = rs.getMetaData();
//        6、通过ResultSetMetaData获取结果集中的列数
            int columnCount = rsmd.getColumnCount();
//        7、判断结果集的下一条是否有数据，如果有数据返回true，并指针下移，如果返回false，指针不下移
            if (rs.next()){
    //            8、创建JavaBean对象用于存值
                Customer cust = new Customer();
                for (int i = 0;i<columnCount;i++){
    //                获取字段的值单行中的一个列值
    //                相当于获取各个字段的值，由于数据返回的结果集是乱序的
                    Object columnValue = rs.getObject(i + 1);
    //                所以要使用获取到的列名【数据库中的列名】，也就是字段名，用字段名拿去映射JavaBean中的属性名，若两者一样则
    //                  获取单个列的名，通过元数据获取
                    String columnName = rsmd.getColumnName(i + 1);
    //                  给cust对象指定的columnName属性【列名】，赋值为对应的columnValue，通过反射实现
                    //        使用此方法将Customer中的属性名，也就是变量名与当前columnName内的值相同的变量映射出来
                    Field field = Customer.class.getDeclaredField(columnName);
    //                          相当于在类的外面获取此类的私有成员变量内的值，需要权限，设置setAccessible(true)则是开启权限
                    field.setAccessible(true);
    //                设置对应的值
                    field.set(cust,columnValue);
                }
                return cust;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,ps,rs);
        }
        return null;
    }
    @Test
    public void testCustomerForCommonQuery(){
//        通过id查询对应的信息
        String sql="select id,name,birth,email from customers where id=?";
        Customer customer = customerForCommonQuery(sql, 10);
        System.out.println(customer);
//      通过名字查询对应的信息
        sql="select name,email from customers where name=?";
        Customer customer1 = customerForCommonQuery(sql, "周杰伦");
        System.out.println(customer1);

    }
```

对映射的大概理解

```java
    @Test
    public void asd() throws  Exception {
        Customer customer = new Customer();
//        使用此方法将Customer中的属性名，也就是变量名为emil的变量映射出来
        Field field = Customer.class.getDeclaredField("email");
//        相当于在类的外面获取此类的私有成员变量email的value时，需要权限，设置setAccessible(true)则是开启权限
        field.setAccessible(true);
//        将customer类中的email属性设置值为asdasd
        field.set(customer,"asdasd");
//        使用此方法将Customer中的属性名，也就是变量名为emil的变量映射出来
        Field field1 = Customer.class.getDeclaredField("name");
//        相当于在类的外面获取此类的私有成员变量email的value时，需要权限，设置setAccessible(true)则是开启权限
        field1.setAccessible(true);
//        将customer类中的name属性设置值为weqweqe
        field1.set(customer,"weqweqe");
        System.out.println(customer.getName());//得到weqweqe
        System.out.println(customer.getEmail());//得到asdasd
    }
```

#### 查询的通用操作的封装（有另起别名）【只对一个表通用】：

- 因为数据对表的列名，java中对属性或变量的命名方式不同意，所以在sql语句中需要使用==起别名==，对java中的属性名进行过度，否则因不统一协调而报错。
- 因为getColumnName()无法获取别名，只能获取表的列名，所以使用getColumnLabel()代替getColumnName()来获取别名。
- getColumnLabel()没有起别名的情况下可以直接获取表的列名，所以直接使用getColumnLabel()就行。
- 相比上一个封装多了别名，

用于处理结果集的JavaBean：

```java
public class Order {
    private int orderId;
    private String orderName;
    private Date orderDate;
    public Order() {}
    public Order(int orderId, String orderName, Date orderDate) {
        this.orderId = orderId;
        this.orderName = orderName;
        this.orderDate = orderDate;
    }
    public int getOrderId() {return orderId;}
    public void setOrderId(int orderId) {this.orderId = orderId;}
    public String getOrderName() {return orderName;}
    public void setOrderName(String orderName) {this.orderName = orderName;}
    public Date getOrderDate() {return orderDate;}
    public void setOrderDate(Date orderDate) {this.orderDate = orderDate;}
    @Override
    public String toString() {
        return "Order{" +
                "orderId=" + orderId +
                ", orderName='" + orderName + '\'' +
                ", orderDate=" + orderDate +
                '}';
    }
}
```

查询的通用操作的封装（有另起别名）：

```java
//    针对Order表的通用操作
    /*
    针对于表的字段名与类的属性名不相同的情况：
        1、必须声明sql时，使用类的属性名来命名字段的别名
        2、使用ResultSetMetaData时，需要使用getColumnLabel()来替换getColumnName()，来获取列的别名。
            说明：如果sql中没有给字段起别名，getColumnLabel()获取的就是列名
    * */
    public Order orderForQuery(String sql, Object ...args)  {
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
//        获取链接
            conn = JDBCUtils.getConn();
//        预编译sql语句
            ps = conn.prepareStatement(sql);
//        填充占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//        获取结果集
            rs = ps.executeQuery();
//        获取结果集的元数据
            ResultSetMetaData rsmd = rs.getMetaData();
//        获取列数
            int columnCount = rsmd.getColumnCount();
//        判断下一条是否有数据
            if (rs.next()){
                Order order = new Order();
                for (int i = 0;i < columnCount;i++){
    //                获取每个列的列值
                    Object columnValue = rs.getObject(i + 1);
    //                获取每个列的列名
//                    此方法只能获取表中的列名： getColumnName()【不推荐使用】，不能获取别名，
//					所以需要另一个方法获取别名
//                    String columnName = rsmd.getColumnName(i + 1);
//                    该方法为获取每个列的别名：getColumnLabel()【推荐使用】，当没有起别名则获取列名
                    String columnLabel = rsmd.getColumnLabel(i + 1);
                    //                通过反射，将对象指定名columnName的属性赋值为指定的值columnValue
                    Field field = Order.class.getDeclaredField(columnLabel);
    //                开启访问私有属性的权限
                    field.setAccessible(true);
    //                将指定类order中的指定属性名columnName设置成指定的值columnValue
                    field.set(order,columnValue);
                }
//                返回数据
                return order;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,ps,rs);
        }
        return null;
    }
    @Test
    public void testOrderrForQuery(){
//        此操作会报错，因为当前的获得结果集中的列名与当前JavaBean的属性名不一致
//        不一致的原因是数据库中的命名习惯以表名下划线加字段名命名，而java中的属性习惯以小驼峰命名法命名
//        所以需要在sql语句中设置别名，该别名为类所对应的属性名
        String sql="select order_id orderId,order_name orderName,order_date orderDate from `order` where order_id=?";
        Order order = orderForQuery(sql, 1);
        System.out.println(order);


    }
```

#### 单条数据查询的通用操作方法：【适合所有的表，且有无别名均可使用】

- 需要使用到泛型
- 只能返回一个对象也就是一条数据
- 通常不确定的信息写成占位符

```java
/*
* 使用PreparedStatementQueryTest针对于不同表的通用查询操作
*/
public class PreparedStatementQueryTest {
    //    Class有泛型，T决定了返回的对象为T类型，泛型参数，泛型方法
//    Class<T>相当于传了个类进去，返回的对象T为T类型的对象
    public <T>T getInstance(Class<T> clazz,String sql, Object ...args){
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
//        获取链接
            conn = JDBCUtils.getConn();
//        预编译sql语句
            ps = conn.prepareStatement(sql);
//        填充占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//        获取结果集
            rs = ps.executeQuery();
//        获取结果集的元数据
            ResultSetMetaData rsmd = rs.getMetaData();
//        获取列数
            int columnCount = rsmd.getColumnCount();
//        判断下一条是否有数据
            if (rs.next()){
//                --------------------------
//                Order order = new Order();//不够通用需要改成动态的，使用改为下面的
//                --------------------------
                T t = clazz.newInstance();//该方法为调用传进来的任何类的空参构造器

                for (int i = 0;i < columnCount;i++){
                    //                获取每个列的列值
                    Object columnValue = rs.getObject(i + 1);
                    //                获取每个列的列名
//                    此方法只能获取表中的列名： getColumnName()【不推荐使用】，不能获取别名，所以需要另一个方法获取别名
//                    String columnName = rsmd.getColumnName(i + 1);
//                    该方法为获取每个列的别名：getColumnLabel()【推荐使用】，当没有起别名则获取列名
                    String columnLabel = rsmd.getColumnLabel(i + 1);
                    //                通过反射，将t对象指定名columnName的属性赋值为指定的值columnValue
//                    Field field = Order.class.getDeclaredField(columnLabel);
//                    使用传进来的calss作为映射对象
                    Field field = clazz.getDeclaredField(columnLabel);
                    //                开启访问私有属性的权限
                    field.setAccessible(true);
                    //                将指定类order中的指定属性名columnName设置成指定的值columnValue
                    field.set(t,columnValue);
                }
//                返回数据
                return t;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,ps,rs);
        }
        return null;
    }
    @Test
    public void testGetInstance(){
//        参数1为数据库表对应的JavaBean类的class对象，参数2为sql语句，参数3为对应sql的占位符
        String sql="select id,name,email from customers where id =?";
        Customer customer = getInstance(Customer.class, sql, 10);
        System.out.println(customer);
//        参数1为数据库表对应的JavaBean类的class对象，参数2为sql语句，参数3为对应sql的占位符
//        当JavaBean的属性名与表的列名命名格式不一致时需要在sql语句中起别名
        String sql1="select order_id orderId,order_name orderName from `order` where order_id = ?";
        Order order = getInstance(Order.class, sql1, 2);
        System.out.println(order);
    }
}
```

## 为什么使用prepareStatement()

- 除了解决Statement的拼串、sql问题之外，prepareStatement()还有的好处
  1. prepareStatement()可以操作Blob的数据，而Statement做不到
  2. prepareStatement()可以实现更高效的批量操作

## 练习

### 一：练习一【插入一条数据】

- 使用了executeUpdate()的返回值，判断是否执行成功
- 返回值为0代表不成功。返回值为1或以上，代表执行成功，并影响了返回值大小的数据量（单位：条）

```java
//练习1：向customers中插入一条数据
public class ExercisesTest01 {
    //    通用的增删改方法
//    sql中占位符的个数与可变形参的长度相同，相当于当前sql中有多少个占位符就在sql参数的后面添加用于填充多少个占位符的参数
    public int update(String sql, Object ...args)   {
        Connection conn = null;
        PreparedStatement ps = null;
        try {
//            1、获取链接
            conn = JDBCUtils.getConn();
//            2、预编译sql语句，返回PrepareStatement实例
            ps = conn.prepareStatement(sql);
//            3、填充占位符，利用可变形参的长度进行填充
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//            4、执行
//            ps.execute();//虽然此方法能返回布尔型的返回值，但是它所代表的是执行后，有无返回结果集，true为有结果集，false为无结果集，而不是是否执行成功
//            执行成功返回影响的行数
            return ps.executeUpdate();//返回的数据类型为int型，其中的返回的值代表的是影响数据库的行数
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//            5、关闭资源
            JDBCUtils.closeResource(conn,ps);
        }
//        执行不成功返回0
        return 0;
    }
    @Test
    public void testInsert(){
        Scanner scanner = new Scanner(System.in);
        System.out.println("输入用户名：");
        String name = scanner.next();
        System.out.println("输入邮箱：");
        String email = scanner.next();
        System.out.println("输入生日：");
        String birth = scanner.next();//当输入格式1999-02-03格式的话会有隐式转换，数据库中可以使用，但是其他格式不可以
        scanner.close();
        String sql="insert into customers(name,email,birth)values(?,?,?)";
        int i = update(sql, name, email, birth);
        if (i>0){
            System.out.println("添加成功");
        }else {
            System.out.println("添加失败");
        }
    }
}
```

### 二：练习二【增删查一条学生数据】

```java
//练习2：向examstudent中插入一条数据，查询一条数据，删除一条数据
public class ExercisesTest02 {
    //    通用的增删改方法
//    sql中占位符的个数与可变形参的长度相同，相当于当前sql中有多少个占位符就在sql参数的后面添加用于填充多少个占位符的参数
    public int update(String sql, Object ...args)   {
        Connection conn = null;
        PreparedStatement ps = null;
        try {
//            1、获取链接
            conn = JDBCUtils.getConn();
//            2、预编译sql语句，返回PrepareStatement实例
            ps = conn.prepareStatement(sql);
//            3、填充占位符，利用可变形参的长度进行填充
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//            4、执行
//            ps.execute();//虽然此方法能返回布尔型的返回值，但是它所代表的是执行后，有无返回结果集，true为有结果集，false为无结果集，而不是是否执行成功
//            执行成功返回影响的行数
            return ps.executeUpdate();//返回的数据类型为int型，其中的返回的值代表的是影响数据库的行数
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//            5、关闭资源
            JDBCUtils.closeResource(conn,ps);
        }
//        执行不成功返回0
        return 0;
    }
    //    Class有泛型，T决定了返回的对象为T类型，泛型参数，泛型方法
//    Class<T>相当于传了个类进去，返回的对象T为T类型的对象
    public <T>T getOneQuery(Class<T> clazz,String sql, Object ...args){
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
//        获取链接
            conn = JDBCUtils.getConn();
//        预编译sql语句
            ps = conn.prepareStatement(sql);
//        填充占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//        获取结果集
            rs = ps.executeQuery();
//        获取结果集的元数据
            ResultSetMetaData rsmd = rs.getMetaData();
//        获取列数
            int columnCount = rsmd.getColumnCount();
//        判断下一条是否有数据
            if (rs.next()){
//                --------------------------
//                Order order = new Order();//不够通用需要改成动态的，使用改为下面的
//                --------------------------
                T t = clazz.newInstance();//该方法为调用传进来的任何类的空参构造器
                for (int i = 0;i < columnCount;i++){
                    //                获取每个列的列值
                    Object columnValue = rs.getObject(i + 1);
                    //                获取每个列的列名
//                    此方法只能获取表中的列名： getColumnName()【不推荐使用】，不能获取别名，所以需要另一个方法获取别名
//                    String columnName = rsmd.getColumnName(i + 1);
//                    该方法为获取每个列的别名：getColumnLabel()【推荐使用】，当没有起别名则获取列名
                    String columnLabel = rsmd.getColumnLabel(i + 1);
                    //                通过反射，将t对象指定名columnName的属性赋值为指定的值columnValue
//                    Field field = Order.class.getDeclaredField(columnLabel);
//                    使用传进来的calss作为映射对象
                    Field field = clazz.getDeclaredField(columnLabel);
                    //                开启访问私有属性的权限
                    field.setAccessible(true);
                    //                将指定类order中的指定属性名columnName设置成指定的值columnValue
                    field.set(t,columnValue);
                }
//                返回数据
                return t;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,ps,rs);
        }
        return null;
    }
//    问题一：向examstudent表中添加一条数据
    @Test
    public void testInsert(){
        Scanner scanner = new Scanner(System.in);
        System.out.print("四级/六级：");
        int type = scanner.nextInt();
        System.out.print("身份证：");
        String IDCard = scanner.next();
        System.out.print("学生证：");
        String examCard = scanner.next();
        System.out.print("学生姓名：");
        String studenName = scanner.next();
        System.out.print("所在城市：");
        String location = scanner.next();
        System.out.print("成绩：");
        int grade = scanner.nextInt();
        String sql="insert examstudent(Type,IDCard,ExamCard,StudentName,Location,Grade)values(?,?,?,?,?,?)";
        int update = update(sql, type, IDCard, examCard, studenName, location, grade);
        if (update>0){
            System.out.println("添加成功");
        }else {
            System.out.println("添加失败");
        }
    }
//    问题二：通过身份证或准考证查询学生信息
    @Test
    public void testIDCardorExamCardQuery(){
        /*1、创建对应的JavaBean类
        * 2、输出提示消息
        * 3、获取键盘输入对应的值作为参数
        * 4、调用封装的查询方法
        * */
        System.out.println("请输入您要输入的类型：");
        System.out.println("a：准考证号");
        System.out.println("b：身份证号");
        Scanner scanner = new Scanner(System.in);
        String selectuon = scanner.next();
//        equalsIgnoreCase不考虑大小写
        if ("a".equalsIgnoreCase(selectuon)){
            System.out.println("请输入准考证号：");
            String examCard = scanner.next();
            String sql="select FlowID flowID,Type type,IDCard,ExamCard examCard,StudentName studentName,Location location,Grade grade from examstudent where ExamCard=?";
            ExamStudent oneQuery = getOneQuery(ExamStudent.class, sql, examCard);
            if (oneQuery!=null){
                System.out.println("===========查询结果============");
                System.out.println("流水号：\t"+oneQuery.getFlowID()+"\n四级/六级:\t"+oneQuery.getType()+
                        "\n身份证号:\t"+oneQuery.getIDCard()+"\n准考证号:\t"+oneQuery.getExamCard()
                        +"\n学生姓名:\t"+oneQuery.getStudentName()+"\n区域:\t"+oneQuery.getLocation()
                        +"\n成绩:\t"+oneQuery.getGrade());
            }else {
                System.out.println("查无此人");
            }
        }else if ("b".equalsIgnoreCase(selectuon)){
            System.out.println("请输入身份证号：");
            String IDCard = scanner.next();
            String sql="select FlowID flowID,Type type,IDCard,ExamCard examCard,StudentName studentName,Location location,Grade grade from examstudent where IDCard=?";
            ExamStudent oneQuery = getOneQuery(ExamStudent.class, sql, IDCard);
//            System.out.println(oneQuery);
            if (oneQuery!=null){
                System.out.println("===========查询结果============");
                System.out.println("流水号：\t"+oneQuery.getFlowID()+"\n四级/六级:\t"+oneQuery.getType()+
                        "\n身份证号:\t"+oneQuery.getIDCard()+"\n准考证号:\t"+oneQuery.getExamCard()
                        +"\n学生姓名:\t"+oneQuery.getStudentName()+"\n区域:\t"+oneQuery.getLocation()
                        +"\n成绩:\t"+oneQuery.getGrade());
            }else {
                System.out.println("查无此人");
            }
        }else {
            System.out.println("您的输入有误！");
        }
    }
//    问题三：删除指定学生信息
//        方法一
    @Test
    public void testDeleteByExamcard(){
        System.out.println("请输入学生的考号：");
        Scanner scanner = new Scanner(System.in);
        String examCard = scanner.next();
//        查询指定注考证号的学生
        String sql="select FlowID flowID,Type type,IDCard,ExamCard examCard,StudentName studentName,Location location,Grade grade from examstudent where ExamCard=?";
        ExamStudent query = getOneQuery(ExamStudent.class, sql, examCard);
        if (query==null){
            System.out.println("查无此人");
        }else {
            String sqldel="delete from examstudent where examCard = ?";
            int update = update(sqldel, examCard);
            if (update > 0){
                System.out.println("删除成功");
            }
        }
    }
//    优化后的操作（比方法一优化了写步骤及代码量）
//    方法二
    @Test
    public void testDeleteByExamcard2(){
        System.out.println("请输入学生的考号：");
        Scanner scanner = new Scanner(System.in);
        String examCard = scanner.next();
        String sql="delete from examstudent where ExamCard = ?";
        int update = update(sql, examCard);
        if (update > 0){
            System.out.println(" 删除成功");
        }else {
            System.out.println("查无此人，请重新输入");
        }
    }
}
```

## 操作BLOB类型字段

- MySQL中，BLOB是一个二进制大型对象，是一个可以存储大量数据的容器，它能容纳不同大小的数据。

- 插入BLOB类型的数据必须使用PreparedStatement，因为BLOB数据类型的数据无法使用字符串拼接写的。

- MySQL的四种BLOB类型（除了在存储的最大信息量上不同外，他们是等同的）

  - 类型				      大小（单位、字节）
  - TinyBlob		       最大255
  - Blob				      最大65k
  - MediumBlob		 最大16M
  - LongBlob			  最大5G

- 实际开发中根据需要存入的数据大小定义不同的BLOB类型

- 需要注意的是：如果存储的文件过大，数据库的性能会下降

- 如果在指定了相关的Blob类型以后，还会报错：xxx too large，那么在mysql的安装目录下，

  找my.ini文件加上如下的配置参数：max_allowed_packet=16M.同时注意：修改了后需要重启服务。

- ==一般不使用数据库存媒体文件，而是存路径==

  

### 一、插入指定的BLOB类型jpg格式的媒体文件

```java
//    向数据库中的customers存储BLOB类型的媒体文件（图片）
    @Test
    public void testInsert() throws Exception {
//        1、获取链接
        Connection conn = JDBCUtils.getConn();
//        2、预编译sql语句
        String sql="insert into customers(name,email,birth,photo)values (?,?,?,?)";
        PreparedStatement ps = conn.prepareStatement(sql);
//        3、填充参数
        ps.setObject(1,"阿斯顿");
        ps.setObject(2,"asd@qq.com");
        ps.setObject(3,"1992-02-08");
//        读取文件的io流，将其解析【具体不清楚】
        FileInputStream is = new FileInputStream(new File("download.jpg"));
        ps.setBlob(4,is);
//        4、执行操作
        ps.execute();
//        5、关闭资源
        JDBCUtils.closeResource(conn,ps);
    }
```

### 二、查询一条含有BLOB类型的数据，并将其下载到本地中

```java
    @Test
    public void testQuery() {
        InputStream is= null;
        FileOutputStream fos= null ;
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
//            获取链接
            conn = JDBCUtils.getConn();
//            预编译sql语句
            String sql="select id,name,email,birth,photo from customers where id = ? ";
            ps = conn.prepareStatement(sql);
//            填充参数
            ps.setInt(1,16);
//            执行操作，获得结果集
            rs = ps.executeQuery();
//            判断数据库中获取到的下一条数据是否为空【第一条数据为字段名，但不作为第一条数据，即下一条数据为第一条】
            if (rs.next()){
//                用于将对应的数据赋值给对应的参数作为输出或使用
    //            方式一：
    //            int id = rs.getInt(1);
    //            String name = rs.getString(2);
    //            String email = rs.getString(3);
    //            Date birth = rs.getDate(4);
    //            方式二：
                int id = rs.getInt("id");
                String name = rs.getString("name");
                String email = rs.getString("email");
                Date birth = rs.getDate("birth");
//                将数据存入bean类中
                Customer cust = new Customer(id, name, email, birth);
                System.out.println(cust);
    //            将Blob类型的字段下载下来，以文件的方式保存在本地
//                此处使用了io流的操作方法
                Blob photo = rs.getBlob("photo");
                is = photo.getBinaryStream();
                fos = new FileOutputStream("asd.jpg");
                byte[] bytes = new byte[1024];
                int len;
                while ((len = is.read(bytes)) != -1){
                    fos.write(bytes,0,len);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//            关闭各种资源
            try {
                if (is !=null)
                    is.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (fos != null)
                    fos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            JDBCUtils.closeResource(conn,ps,rs);
        }
    }
```

## 使用prepareStatement批量添加

```java
/*
 *使用prepareStatement实现批量数据的操作
 * update、delete本身就具有批量操作的效果
 * 这里的批量操作，主要是指批量插入，使用prepareStatement如何实现更高效的批量插入
 * 题目：想goods表中插入20000条数据
 * CREATE TABLE goods(	id INT PRIMARY KEY AUTO_INCREMENT,name VARCHAR(25));【创建表的sql语句】
 */
public class BatchInsert {
//    方法一：使用Statement
    @Test
    public void batchInsertTest1() throws Exception {
        Connection conn = null;
        Statement st = null;
        try {
            long start = System.currentTimeMillis();
            conn = JDBCUtils.getConn();
            st = conn.createStatement();
            for (int i= 0;i<20000;i++){
                String sql="insert into goods(name)values('name_"+i+"')";
                st.execute(sql);
            }
            long end = System.currentTimeMillis();
            System.out.println("耗时："+(end-start));
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            try {
                if (st!=null)
                    st.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try {
                if (conn!=null)
                    conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
//    方法二：使用prepareStatement
    @Test
    public void batchInsertTest2(){
        Connection conn = null;
        PreparedStatement ps = null;
        try {
            long start = System.currentTimeMillis();
            conn = JDBCUtils.getConn();
            String sql="insert into goods(name)values (?)";
            ps = conn.prepareStatement(sql);
            for (int i = 0;i<1000000;i++){
                ps.setObject(1,"name_"+i);
                ps.execute();
            }
            long end = System.currentTimeMillis();
            System.out.println("耗时："+(end-start));
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,ps);
        }
    }
/*
 * 方式三：
 * 在方式二的基础上使用积攒sql一次多量运行的方法实现
 * 1、addBatch()、executeBatch()、clearBatch()
 * 2、mysql服务器默认是关闭处理的，我们需要通过一个参数，让mysql开启批处理的支持
 *      ?rewriteBatchedStatements=true【写在配置文件的url后面】
 * 3、此方法需要使用mysql-connector-java-5.1.37以上的jar包
 */
    @Test
    public void batchInsertTest3(){
        Connection conn = null;
        PreparedStatement ps = null;
        try {
            long start = System.currentTimeMillis();
            conn = JDBCUtils.getConn();
            String sql="insert into goods(name)values (?)";
            ps = conn.prepareStatement(sql);
            for (int i = 1;i<1000001;i++){
                ps.setObject(1,"name_"+i);
//                积攒成一批
//                相当于积攒一批sql语句
                ps.addBatch();
//                判断一批的量为1000时执行并清空
                if (i%1000==0){
//                    执行该批次
                    ps.executeBatch();
//                    清空该批次
                    ps.clearBatch();
                }
            }
            long end = System.currentTimeMillis();
            System.out.println("耗时："+(end-start));
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,ps);
        }

    }
    /*
     * 方式四：【推荐使用】
     * 在方式三的基础上进行了优化
     * 设置了链接不允许自动提交数据
     */
    @Test
    public void batchInsertTest4(){
        Connection conn = null;
        PreparedStatement ps = null;
        try {
            long start = System.currentTimeMillis();
            conn = JDBCUtils.getConn();
//            设置不允许自动提交数据
            conn.setAutoCommit(false);
            String sql="insert into goods(name)values (?)";
            ps = conn.prepareStatement(sql);
            for (int i = 1;i<1000001;i++){
                ps.setObject(1,"name_"+i);
                ps.addBatch();
                if (i%10000==0){
                    System.out.println(i);
                    ps.executeBatch();
                    ps.clearBatch();
                }
            }
//            统一提交数据
            conn.commit();
            long end = System.currentTimeMillis();
            System.out.println("耗时："+(end-start));
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,ps);
        }

    }
}
```

## 数据库事务的处理

- 数据库事务即是相当于两个操作互动。一方动了，另一方必须动，如果中间出现错误则必须回到未动状态。
- 当在处理事务是，链接不能提交数据，也不能关闭，必须当整个事务快执行完毕才进行提交关闭，若期间出现错误则要回滚数据，使其回到最初状态。
- 资源关闭前需要把链接默认提交数据的设置设置回true，主要针对于使用数据库链接池的使用

```java
//事务测试
/**
 * 1、什么叫数据库事务
 * 事务：一组逻辑操作单元，使数据从一种状态变到另一种状态。
 *      一组逻辑操作单元：一个或多个DML操作
 * 2、事务处理的原则：保证所有事务都作为一个工作单元来执行，即使出现了故障，都不能改变这种执行方式。
 *    当在一个事务中执行多个操作时，要么所有的事务都被提交（commit），那么这些修改就永久的保存下来；
 *    要么数据库管理系统将放弃所有的修改，整个事务回滚（rollback）到最初状态
 * 3、数据一旦提交，就不可回滚
 * 4、会导致数据的自动提交的操作：
 *      1--DDL操作一旦执行，都会自动提交。
 *          set autocommit = false 对DDL操作失效
 *      2--DML默认情况下，一旦执行，就会自动提交。
 *          可以通过set autocommit = fales的方式取消DML操作的自动提交。
 *      3--默认在关闭连接时，会自动的提交数据
 */
public class TransactionTest {
    /*------------------------------------------未考虑数据库事务的转账操作------------------------------------------------------*/
    //    通用的增删改方法 version-1.0
//    sql中占位符的个数与可变形参的长度相同，相当于当前sql中有多少个占位符就在sql参数的后面添加用于填充多少个占位符的参数
    public int update(String sql,Object ...args)   {
        Connection conn = null;
        PreparedStatement ps = null;
        try {
//            1、获取链接
            conn = JDBCUtils.getConn();
//            2、预编译sql语句，返回PrepareStatement实例
            ps = conn.prepareStatement(sql);
//            3、填充占位符，利用可变形参的长度进行填充
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//            4、执行
            return ps.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//            5、关闭资源
            JDBCUtils.closeResource(conn,ps);
        }
        return  0;
    }
    /**
     * 针对于数据表user_table来说；
     * AA用户给BB用户转账100
     * update user_table set balance = balance - 100 where user = 'AA'
     * update user_table set balance = balance + 100 where user = 'BB'
     * 此测试对应方法为1.0的通用增删改
     * 问题：当AA中减了一百后出现异常，BB中并不会加一百，不符合该事务的执行。但需要回滚时，因链接已经默认提交，无法回滚数据，因此此方法【不可用】
     */
    @Test
    public  void testUpdate(){
        String sql1="update user_table set balance = balance - 100 where user = ?";
        update(sql1,"AA");
//        模拟异常
        int a =10/0;
        String sql2="update user_table set balance = balance + 100 where user = ?";
        update(sql2,"BB");
        System.out.println("ok");
    }
    /*------------------------------------------虑数据库事务的转账操作------------------------------------------------------*/
    //    通用的增删改方法 version-2.0【考虑数据库事务】
//    sql中占位符的个数与可变形参的长度相同，相当于当前sql中有多少个占位符就在sql参数的后面添加用于填充多少个占位符的参数
    /*
     *改动：
     * 1、链接作为参数传进
     * 2、外边传进的外边关闭资源，里面创建的里面关闭资源
     * 3、将链接链接起来，避免出错时提交数据，造成数据错误时不能回滚的现象导致数据错误
     */
    public int update(Connection conn,String sql,Object ...args)   {
        PreparedStatement ps = null;
        try {
//            1、预编译sql语句，返回PrepareStatement实例
            ps = conn.prepareStatement(sql);
//            2、填充占位符，利用可变形参的长度进行填充
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//            3、执行
            return ps.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//            4、关闭资源
            JDBCUtils.closeResource(null,ps);
        }
        return  0;
    }
    @Test
    public  void testUpdate2(){
        Connection conn = null;
        try {
            conn = JDBCUtils.getConn();
//            取消数据的自动提交
            conn.setAutoCommit(false);
            String sql1="update user_table set balance = balance - 100 where user = ?";
            update(conn,sql1,"AA");
//        模拟异常
            int a =10/0;
            System.out.println(a);
            String sql2="update user_table set balance = balance + 100 where user = ?";
            update(conn,sql2,"BB");
            System.out.println("ok");
//            当所有操作完成时时数据提交
            conn.commit();
        } catch (Exception e) {
            e.printStackTrace();
//            当出现异常时数据回滚
            try {
                if (conn!=null)
                    conn.rollback();
            } catch (SQLException ex) {
                ex.printStackTrace();
            }
        }finally {
            //资源关闭前需要把链接默认提交数据的设置设置回true
            //            主要针对于使用数据库链接池的使用
            try {
                if (conn!=null)
                    conn.setAutoCommit(false);
            } catch (SQLException e) {
                e.printStackTrace();
            }
            //        关闭资源，当前只有链接需要关闭
            JDBCUtils.closeResource(conn,null);
        }

    }
}
```

## 事务的ACID属性

1. 原子性（Atomicity）
   - 原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生
2. 一致性（Consistency）
   - 事务必须使数据库从一个一致性状态变换到另一个一致性状态。
3. 隔离性（Isolation）
   - 事务的隔离性是指一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。
4. 持久性（Durability）
   - 持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来的其他操作和数据库故障不应该对其有任何影响。

#### 数据库的并发问题

- 对于同时运行的多个事，当这些事务访问数据库中相同的数据时，如果没有采取不要的隔离机制，就会导致各种并发问题
  - **脏读**：对于两个事务T1，T2，T1读取了已经被T2更新但还没有被提交的字段。之后，若T2回滚，T1读取的内容就是临时且无效的。
  - **不可重复读**：对于两个事务T1，T2，T1读取了一个字段，然后T2更新了该字段。之后T1再次读取同一字段，值就不同了。
  - **幻读**：对于两个事务T1，T2，T1从一个表中读取了一个字段，然后T2在该表中插入了一些新的行。之后，如果T1再次读取同一个表，就会多出几行。
- **数据库事务的隔离性**：数据库系统必须具有隔离并发运行各个事务的能力，使他们不会相互影响，避免各种并发问题。
- 一个事务与其他事务隔离的程度称为隔离级别。数据库规定了多种事务隔离级别，不同隔离级别对应不同的干扰程度，隔离级别越高，数据一致性就越好，但并发性越弱。

#### 四种隔离级别

数据库提供的4种事务隔离级别：

| 隔离级别                                             | 描述                                                         |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| READ UNCOMMITTED                  （读未提交数据）   | 允许事务读取未被其他事务提交的更变。脏读，不可重复读和幻读的问题都会出现 |
| READ COMMITED                       （读已提交数据） | 只允许事务读取已经被其他事务提交的变更。可以避免脏读，但不可重复读和幻读问题任然可能出现 |
| REPEATABLE READ                    （可重复读）      | 确保事务可以多次从一个字段中读取相同的值。在这个事务持续期间，禁止其他事务对这个字段进行更新。可以避免脏读和不可重复读，但幻读的问题仍然存在。 |
| SERIALIZABLE                             （串行化）  | 确保事务可以从一个表中读取相同的行。在这个事务持续期间，禁止其他事务对该表执行插入、更新和删除操作。所有并发问题都可以避免，但性能十分低下 |

- Oracle支持的2种事务隔离级别：READ COMMITED，SERIALIZABLE。Oracle默认的事务隔离级别为：READ COMMITED。
- Mysql支持4种隔离级别。Mysql默认的事务隔离级别为：REPEATABLE READ。

##### 在Mysql中设置隔离级别

- 每启动一个mysql程序，就会获得一个单独的数据库连接，每个数据库连接都有一个全局变量@@tx_isolation，表示当前的隔离级别

- 查看当前的隔离级别：

  ```mysql
  SELECT @@tx_isolation;
  ```

- 设置当前Mysql连接的隔离级别：

  ```mysql
  set transaction isolation level rerad committed;
  ```

- 设置数据库系统的全局的隔离级别：

  ```mysql
  set global transaction isolation level read committed;
  ```

##### 数据库隔离级别注意事项

- 在数据库中直接使用sql语句设置，只对当前运行的Mysql服务启动时有效，如果重启则会恢复到默认状态。
- 在java程序中设置，只在当前执行的程序连接中有效。
- 一般情况下在服务器中的数据库不考虑重启（否则需要设置很多的配置项）
- 在写java代码一定要避免脏读问题

##### 隔离级别的操作演示示例

```java
public class TransactionTest {
    /*------------------------------------------虑数据库事务的转账操作------------------------------------------------------*/
    //    通用的增删改方法 version-2.0【考虑数据库事务】
//    sql中占位符的个数与可变形参的长度相同，相当于当前sql中有多少个占位符就在sql参数的后面添加用于填充多少个占位符的参数
    /*
     *改动：
     * 1、链接作为参数传进
     * 2、外边传进的外边关闭资源，里面创建的里面关闭资源
     * 3、将链接链接起来，避免出错时提交数据，造成数据错误时不能回滚的现象导致数据错误
     */
    public int update(Connection conn,String sql,Object ...args)   {
        PreparedStatement ps = null;
        try {
//            1、预编译sql语句，返回PrepareStatement实例
            ps = conn.prepareStatement(sql);
//            2、填充占位符，利用可变形参的长度进行填充
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//            3、执行
            return ps.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//            4、关闭资源
            JDBCUtils.closeResource(null,ps);
        }
        return  0;
    }
    /*----------------------------------------数据隔离性的测试----------------------------------------------------*/
//    通用的查询操作，用于返回数据表中的一条记录 version-2.0【考虑数据库事务】
    //    Class有泛型，T决定了返回的对象为T类型，泛型参数，泛型方法
//    Class<T>相当于传了个类进去，返回的对象T为T类型的对象
    public <T>T getInstance(Connection conn,Class<T> clazz,String sql, Object ...args){
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
//        预编译sql语句
            ps = conn.prepareStatement(sql);
//        填充占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//        获取结果集
            rs = ps.executeQuery();
//        获取结果集的元数据
            ResultSetMetaData rsmd = rs.getMetaData();
//        获取列数
            int columnCount = rsmd.getColumnCount();
//        判断下一条是否有数据
            if (rs.next()){
//                --------------------------
//                Order order = new Order();//不够通用需要改成动态的，使用改为下面的
//                --------------------------
                T t = clazz.newInstance();//该方法为调用传进来的任何类的空参构造器

                for (int i = 0;i < columnCount;i++){
                    //                获取每个列的列值
                    Object columnValue = rs.getObject(i + 1);
                    //                获取每个列的列名
//                    此方法只能获取表中的列名： getColumnName()【不推荐使用】，不能获取别名，所以需要另一个方法获取别名
//                    String columnName = rsmd.getColumnName(i + 1);
//                    该方法为获取每个列的别名：getColumnLabel()【推荐使用】，当没有起别名则获取列名
                    String columnLabel = rsmd.getColumnLabel(i + 1);
                    //                通过反射，将t对象指定名columnName的属性赋值为指定的值columnValue
//                    Field field = Order.class.getDeclaredField(columnLabel);
//                    使用传进来的calss作为映射对象
                    Field field = clazz.getDeclaredField(columnLabel);
                    //                开启访问私有属性的权限
                    field.setAccessible(true);
                    //                将指定类order中的指定属性名columnName设置成指定的值columnValue
                    field.set(t,columnValue);
                }
//                返回数据
                return t;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(null,ps,rs);
        }
        return null;
    }

    /**
     * 脏读测试
     * 脏读可以理解为：当你使用一个连接第一次查询数据未提交，值为1000；
     *               然后使用第二个连接更改数据未提交且更改数据后延迟程序结束时间，值为2000；
     *               再次使用第一个连接查询时还是未提交，值为2000；
     *               当第二个连接的程序结束时，再次使用第一个连接查询时，值会因为未提交导致数据无效而便会1000；
     * 当隔离界别为：READ UNCOMMITTED（读未提交数据）会有脏读问题，这个问题必须避免
     *可以通过设置隔离级别达到效果，通常设置为READ_COMMITTED（读已提交）就行，此设置只对当前运行程序（连接）有效。 
     */
    @Test
    public void testTransactionSelect() throws Exception {
        Connection conn = JDBCUtils.getConn();
//        获取当前事务的隔离级别
        System.out.println(conn.getTransactionIsolation());
//        设置数据库隔离级别
        conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
//        获取当前事务的隔离级别
        System.out.println(conn.getTransactionIsolation());
//        取消自动提交数据
        conn.setAutoCommit(false);
        String sql="select user,password,balance from user_table where user=?";
        UserTable user = getInstance(conn, UserTable.class, sql, "cc");
        System.out.println(user);
    }
    @Test
    public void testTransactionUpdate() throws Exception {
        Connection conn = JDBCUtils.getConn();
//        取消自动提交数据
        conn.setAutoCommit(false);
        String sql="update user_table set balance =? where user =?";
        update(conn,sql,15000,"CC");
        Thread.sleep(30000);
        System.out.println("修改结束");
    }
}
```

## DAO及实现类

- 这里有三个文件，分别时BaseDao、对应表的Dao、对应表的Dao的实现类
  - BaseDao：此类是一个抽象类，用于被实现类继承后实现其方法的。
  - 对应表的Dao：此类是一个接口，用于被实现类实现，用于定义方法规范
  - Dao的实现类：此类是一个实现类，用于继承BaseDao和实现Dao的方法
    						【可理解为在此类中使用对应表的Dao规范的方法来使用BaseDao的操作方法，从而供给具体服务或操作使用的具体方法】

#### 对应的代码示例【未优化版】

BaseDao

```java
/**
 *  封装了数据表的通用操作
 */
public abstract class BaseDao {
    //    通用的增删改方法 version-2.0【考虑数据库事务】
    public int update(Connection conn, String sql, Object ...args)   {
        PreparedStatement ps = null;
        try {
//            1、预编译sql语句，返回PrepareStatement实例
            ps = conn.prepareStatement(sql);
//            2、填充占位符，利用可变形参的长度进行填充
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//            3、执行
            return ps.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//            4、关闭资源
            JDBCUtils.closeResource(null,ps);
        }
        return  0;
    }
    //    通用的查询操作，用于返回数据表中的一条记录 version-2.0【考虑数据库事务】
    public <T>T getInstance(Connection conn,Class<T> clazz,String sql, Object ...args){
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
//        预编译sql语句
            ps = conn.prepareStatement(sql);
//        填充占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//        获取结果集
            rs = ps.executeQuery();
//        获取结果集的元数据
            ResultSetMetaData rsmd = rs.getMetaData();
//        获取列数
            int columnCount = rsmd.getColumnCount();
//        判断下一条是否有数据
            if (rs.next()){
                T t = clazz.newInstance();//该方法为调用传进来的任何类的空参构造器

                for (int i = 0;i < columnCount;i++){
                    //                获取每个列的列值
                    Object columnValue = rs.getObject(i + 1);
                    //                获取每个列的列名
//                    该方法为获取每个列的别名：getColumnLabel()，当没有起别名则获取列名
                    String columnLabel = rsmd.getColumnLabel(i + 1);
//                    使用传进来的calss作为映射对象
                    Field field = clazz.getDeclaredField(columnLabel);
                    //                开启访问私有属性的权限
                    field.setAccessible(true);
                    //                将指定类order中的指定属性名columnName设置成指定的值columnValue
                    field.set(t,columnValue);
                }
//                返回数据
                return t;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(null,ps,rs);
        }
        return null;
    }
    //    针对不同表的通用查询操作【可查询多条数据】version2.0【考虑事务】
    public <T> List<T> getForList(Connection conn,Class<T> clazz, String sql, Object ...args){
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
//        预编译sql语句
            ps = conn.prepareStatement(sql);
//        填充占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//        获取结果集
            rs = ps.executeQuery();
//        获取结果集的元数据
            ResultSetMetaData rsmd = rs.getMetaData();
//        获取列数
            int columnCount = rsmd.getColumnCount();
//            创建集合对象
            ArrayList<T> list = new ArrayList<>();

//        判断下一条是否有数据
            while (rs.next()){
                T t = clazz.newInstance();//该方法为调用传进来的任何类的空参构造器
//                处理结果集一行数据中的每一个列：给t对象指定的属性赋值
                for (int i = 0;i < columnCount;i++){
                    //                获取每个列的列值
                    Object columnValue = rs.getObject(i + 1);
//                    该方法为获取每个列的别名：getColumnLabel()【推荐使用】，当没有起别名则获取列名
                    String columnLabel = rsmd.getColumnLabel(i + 1);
//                    使用传进来的calss作为映射对象
                    Field field = clazz.getDeclaredField(columnLabel);
                    //                开启访问私有属性的权限
                    field.setAccessible(true);
                    //                将指定类order中的指定属性名columnName设置成指定的值columnValue
                    field.set(t,columnValue);
                }
                list.add(t);
            }
            return list;
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(null,ps,rs);
        }
        return null;
    }
//    用于查询特殊值的通用方法
//    此处用到了泛型来确认返回不确定返回类型的值
    public <E>E getValue(Connection conn, String sql, Object...args){
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            ps = conn.prepareStatement(sql);
            for (int i = 0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
            rs = ps.executeQuery();
            if (rs.next()){
                return (E) rs.getObject(1);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(null,ps,rs);
        }
        return null;
    }
}
```

对应表的Dao

```java
/*
 * 此接口用于规范针对customers表的常用操作
 */
public interface CustomerDao {
    /**
     * 将cust对象添加到数据库中
     * @param conn
     * @param cust
     */
    void insert(Connection conn, Customer cust);
    /**
     * 针对指定id，删除表中的一条记录
     * @param conn
     * @param id
     */
    void deleteByid(Connection conn,int id);
    /**
     * 针对内存中的cust对象，去修改数据表中指定的记录
     * @param conn
     * @param cust
     */
    void update(Connection conn,Customer cust);
    /**
     * 针对指定的id查询得到对应的Customer对象
     * @param conn
     * @param id
     * @return
     */
    Customer getCustomerById(Connection conn,int id);
    /**
     * 查询表中的所有记录构成的集合
     * @param conn
     * @return
     */
    List<Customer> getAll(Connection conn);
    /**
     * 返回数据表中的数据条目数
     * @param conn
     * @return
     */
    Long getCount(Connection conn);
    /**
     * 返回数据表中最大的生日
     * @param conn
     * @return
     */
    Date getMaxBirth(Connection conn);
}
```

Dao的实现类

```java
/**
 * CustomerDao接口的实现类，继承BaseDao实现方法
 */
public class CustomerDAOImpl extends BaseDao implements CustomerDao {
    @Override
    public void insert(Connection conn, Customer cust) {
        String sql="insert into customers(name,email,birth)values(?,?,?)";
        update(conn,sql,cust.getName(),cust.getEmail(),cust.getBirth());
    }
    @Override
    public void deleteByid(Connection conn, int id) {
        String sql="delete from customers where id = ?";
        update(conn,sql,id);
    }
    @Override
    public void update(Connection conn, Customer cust) {
        String sql="update customers set name = ?,email = ?,birth = ? where id = ?";
        update(conn,sql,cust.getName(),cust.getEmail(),cust.getBirth(),cust.getId());
    }
    @Override
    public Customer getCustomerById(Connection conn, int id) {
        String sql="select id,name,email,birth from customers where id = ?";
        Customer customer = getInstance(conn, Customer.class, sql, id);
        return customer;
    }
    @Override
    public List<Customer> getAll(Connection conn) {
        String sql="select id,name,email,birth from customers";
        return  getForList(conn, Customer.class, sql);
    }
    @Override
    public Long getCount(Connection conn) {
        String sql="select count(*) from customers";
        return getValue(conn, sql);
    }
    @Override
    public Date getMaxBirth(Connection conn) {
        String sql="select max(birth) from customers";
        return getValue(conn,sql);
    }
}
```

具体服务或操作方法

```java
public class CustomerDAOImplTest {
    private CustomerDAOImpl dao= new CustomerDAOImpl();
    @Test
    public void insert() throws Exception {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConn();
            Customer cust=new Customer(1,"asdasd111","asdasd@qweqe.ccc",new Date(1231231232));
            dao.insert(conn,cust);
            System.out.println("添加成功");
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
    @Test
    public void deleteByid() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConn();
            dao.deleteByid(conn,20);
            System.out.println("删除成功");
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
    /**
     * 此方法改的话全都改了，可以另造一个方法单独改一个数据的
     */
    @Test
    public void update() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConn();
            Customer cust=new Customer(19,"asd","asd@asd1111111.com",new Date());
            dao.update(conn,cust);
            System.out.println("修改成功");
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
    @Test
    public void getCustomerById() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConn();
            Customer customerById = dao.getCustomerById(conn, 6);
            System.out.println("查找成功，"+customerById);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
    @Test
    public void getAll() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConn();
            List<Customer> all = dao.getAll(conn);
            all.forEach(System.out::println);
            System.out.println("查找成功");
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
    @Test
    public void getCount() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConn();
            Long count = dao.getCount(conn);
            System.out.println("查找成功，总条目数为："+count);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
    @Test
    public void getMaxBirth() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConn();
            Date maxBirth = dao.getMaxBirth(conn);
            System.out.println("查找成功，最大的生日为："+maxBirth);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
}
```

#### 对应的代码示例【优化版】

BaseDao

```java
//【dao包下的为未优化的版本】
//【dao2包下的为优化的版本】
/**
 *  封装了数据表的通用操作
 */
//--------------------比原来堆了个泛型参数T
public abstract class BaseDao<T> {
    private Class<T> clazz=null;
    //造一个非静态的代码块，因为非静态的才能调用非静态的clazz且他的子类的实现可以被多次初始化
    {   //获取当前BaseDao的子类继承的父类中的泛型
//        此处的this的指向并不是当前这个类，而是谁继承这个类就会启动这块程序，所以指向的是继承该类的类，也就是此类的子类。
        Type genericSuperclass = this.getClass().getGenericSuperclass();//当前类的带泛型的父类
        ParameterizedType parameType = (ParameterizedType) genericSuperclass;//泛型的父类的参数
        Type[] typeArguments = parameType.getActualTypeArguments();//获取父类的泛型参数
        clazz = (Class<T>) typeArguments[0];//泛型的第一个参数
    }
    //    通用的增删改方法 version-2.0【考虑数据库事务】
    public int update(Connection conn, String sql, Object ...args)   {
        PreparedStatement ps = null;
        try {
//            1、预编译sql语句，返回PrepareStatement实例
            ps = conn.prepareStatement(sql);
//            2、填充占位符，利用可变形参的长度进行填充
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//            3、执行
            return ps.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//            4、关闭资源
            JDBCUtils.closeResource(null,ps);
        }
        return  0;
    }
    //    通用的查询操作，用于返回数据表中的一条记录 version-2.0【考虑数据库事务】
    //------------------------------------当已经指明类时，就不需要Customer.class了
//    public <T>T getInstance(Connection conn,Class<T> clazz,String sql, Object ...args){
    public T getInstance(Connection conn,String sql, Object ...args){
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
//        预编译sql语句
            ps = conn.prepareStatement(sql);
//        填充占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//        获取结果集
            rs = ps.executeQuery();
//        获取结果集的元数据
            ResultSetMetaData rsmd = rs.getMetaData();
//        获取列数
            int columnCount = rsmd.getColumnCount();
//        判断下一条是否有数据
            if (rs.next()){
                T t = clazz.newInstance();//该方法为调用传进来的任何类的空参构造器
                for (int i = 0;i < columnCount;i++){
                    //                获取每个列的列值
                    Object columnValue = rs.getObject(i + 1);
                    //                获取每个列的列名
//                    该方法为获取每个列的别名：getColumnLabel()，当没有起别名则获取列名
                    String columnLabel = rsmd.getColumnLabel(i + 1);
//                    使用传进来的calss作为映射对象
                    Field field = clazz.getDeclaredField(columnLabel);
                    //                开启访问私有属性的权限
                    field.setAccessible(true);
                    //                将指定类order中的指定属性名columnName设置成指定的值columnValue
                    field.set(t,columnValue);
                }
//                返回数据
                return t;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(null,ps,rs);
        }
        return null;
    }
    //    针对不同表的通用查询操作【可查询多条数据】version2.0【考虑事务】
    //------------------------------------当已经指明类时，就不需要Customer.class了
//    public <T> List<T> getForList(Connection conn,Class<T> clazz, String sql, Object ...args){
    public  List<T> getForList(Connection conn, String sql, Object ...args){
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
//        预编译sql语句
            ps = conn.prepareStatement(sql);
//        填充占位符
            for (int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//        获取结果集
            rs = ps.executeQuery();
//        获取结果集的元数据
            ResultSetMetaData rsmd = rs.getMetaData();
//        获取列数
            int columnCount = rsmd.getColumnCount();
//            创建集合对象
            ArrayList<T> list = new ArrayList<>();

//        判断下一条是否有数据
            while (rs.next()){
                T t = clazz.newInstance();//该方法为调用传进来的任何类的空参构造器
//                处理结果集一行数据中的每一个列：给t对象指定的属性赋值
                for (int i = 0;i < columnCount;i++){
                    //                获取每个列的列值
                    Object columnValue = rs.getObject(i + 1);
//                    该方法为获取每个列的别名：getColumnLabel()【推荐使用】，当没有起别名则获取列名
                    String columnLabel = rsmd.getColumnLabel(i + 1);
//                    使用传进来的calss作为映射对象
                    Field field = clazz.getDeclaredField(columnLabel);
                    //                开启访问私有属性的权限
                    field.setAccessible(true);
                    //                将指定类order中的指定属性名columnName设置成指定的值columnValue
                    field.set(t,columnValue);
                }
                list.add(t);
            }
            return list;
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(null,ps,rs);
        }
        return null;
    }
//    用于查询特殊值的通用方法
//    此处用到了泛型来确认返回不确定返回类型的值
    public <E>E getValue(Connection conn, String sql, Object...args){
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            ps = conn.prepareStatement(sql);
            for (int i = 0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
            rs = ps.executeQuery();
            if (rs.next()){
                return (E) rs.getObject(1);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(null,ps,rs);
        }
        return null;
    }
}
```

对应表的Dao

```java
//【dao包下的为未优化的版本】
//【dao2包下的为优化的版本】
/*
 * 此接口用于规范针对customers表的常用操作
 */
public interface CustomerDao {
    /**
     * 将cust对象添加到数据库中
     * @param conn
     * @param cust
     */
    void insert(Connection conn, Customer cust);
    /**
     * 针对指定id，删除表中的一条记录
     * @param conn
     * @param id
     */
    void deleteByid(Connection conn, int id);
    /**
     * 针对内存中的cust对象，去修改数据表中指定的记录
     * @param conn
     * @param cust
     */
    void update(Connection conn, Customer cust);
    /**
     * 针对指定的id查询得到对应的Customer对象
     * @param conn
     * @param id
     * @return
     */
    Customer getCustomerById(Connection conn, int id);
    /**
     * 查询表中的所有记录构成的集合
     * @param conn
     * @return
     */
    List<Customer> getAll(Connection conn);
    /**
     * 返回数据表中的数据条目数
     * @param conn
     * @return
     */
    Long getCount(Connection conn);
    /**
     * 返回数据表中最大的生日
     * @param conn
     * @return
     */
    Date getMaxBirth(Connection conn);
}
```

Dao的实现类

```java
 //【dao包下的为未优化的版本】
//【dao2包下的为优化的版本】
/**
 * CustomerDao接口的实现类，继承BaseDao实现方法
 */
//-----------------------------------继承时指明操作的哪个表也就是哪个类
public class CustomerDAOImpl extends BaseDao<Customer> implements CustomerDao {
    @Override
    public void insert(Connection conn, Customer cust) {
        String sql="insert into customers(name,email,birth)values(?,?,?)";
        update(conn,sql,cust.getName(),cust.getEmail(),cust.getBirth());
    }
    @Override
    public void deleteByid(Connection conn, int id) {
        String sql="delete from customers where id = ?";
        update(conn,sql,id);
    }
    @Override
    public void update(Connection conn, Customer cust) {
        String sql="update customers set name = ?,email = ?,birth = ? where id = ?";
        update(conn,sql,cust.getName(),cust.getEmail(),cust.getBirth(),cust.getId());
    }
    @Override
    public Customer getCustomerById(Connection conn, int id) {
        String sql="select id,name,email,birth from customers where id = ?";
        //------------------------------------当已经指明类时，就不需要Customer.class了
//        Customer customer = getInstance(conn, Customer.class, sql, id);
        return getInstance(conn,sql, id);
    }
    @Override
    public List<Customer> getAll(Connection conn) {
        String sql="select id,name,email,birth from customers";
        //------------------------------------当已经指明类时，就不需要Customer.class了
//        return  getForList(conn, Customer.class, sql);
        return  getForList(conn, sql);
    }
    @Override
    public Long getCount(Connection conn) {
        String sql="select count(*) from customers";
        return getValue(conn, sql);
    }
    @Override
    public Date getMaxBirth(Connection conn) {
        String sql="select max(birth) from customers";
        return getValue(conn,sql);
    }
}
```

具体服务或操作方法

```java
public class CustomerDAOImplTest {
    private CustomerDAOImpl dao= new CustomerDAOImpl();
    @Test
    public void insert() throws Exception {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConn();
            Customer cust=new Customer(1,"asdasd111","asdasd@qweqe.ccc",new Date(1231231232));
            dao.insert(conn,cust);
            System.out.println("添加成功");
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
    @Test
    public void deleteByid() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConn();
            dao.deleteByid(conn,20);
            System.out.println("删除成功");
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
    /**
     * 此方法改的话全都改了，可以另造一个方法单独改一个数据的
     */
    @Test
    public void update() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConn();
            Customer cust=new Customer(19,"asd","asd@asd1111111.com",new Date());
            dao.update(conn,cust);
            System.out.println("修改成功");
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
    @Test
    public void getCustomerById() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConn();
            Customer customerById = dao.getCustomerById(conn, 6);
            System.out.println("查找成功，"+customerById);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
    @Test
    public void getAll() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConn();
            List<Customer> all = dao.getAll(conn);
            all.forEach(System.out::println);
            System.out.println("查找成功");
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
    @Test
    public void getCount() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConn();
            Long count = dao.getCount(conn);
            System.out.println("查找成功，总条目数为："+count);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
    @Test
    public void getMaxBirth() {
        Connection conn = null;
        try {
            conn = JDBCUtils.getConn();
            Date maxBirth = dao.getMaxBirth(conn);
            System.out.println("查找成功，最大的生日为："+maxBirth);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
}
```

## 数据库连接池

### 优点

1. 资源重用：由于数据库连接得以重用，避免了频繁创建，释放连接引起的大量性能开销。在减少系统消耗的基础上，另一方面也增加了系统运行环境的平稳性。
2. 更快的系统反应速度：数据库连接池在初始化过程中，往往已经创建了若干数据库连接置于连接池中备用。此时连接的初始化工作均已完成。对于业务请求处理而言，直接利用现有可用连接，避免了数据库连接初始化和释放过程的时间开销，从而减少了系统的响应时间。
3. 新的资源分配手段：对于多应用共享同一数据库的系统而言，可在应用层通过数据库连接池的配置，实现某一应用最大可用数据库连接数的限制，避免某一应用独占所有的数据库资源。
4. 统一的连接管理，避免数据库连接泄露：在较为完善的数据库连接池实现中，可根据预先的占用超市设定，强制回收被占用连接，从而避免了常规数据库连接操作中可能出现的资源泄露。

### 多种开源的数据库连接池

- JDBC的数据库连接池使用java.sql.DataSource来表示，DataSource只是一个接口，该接口通常由服务器（Weblogic，WebSphere，Tomcat）提供实现。
  - **DBCP**：Apache提供的数据库连接池。tomcat服务器自带dbcp数据库连接池。速度相对C3P0较快，因自身存在bug，Hibernate3不再提供支持。
  - **C3P0**：一个开源组织提供的一个数据库连接池，速度相对较慢，稳定性还可以。Hibernate官方推荐使用。
  - **Peoxool**：是sourceforge下的一个开源项目数据库连接池，有监控连接池状态的功能，稳定性较C3P0差一点。
  - **BoneCP**：是一个开源组织提供的数据库连接池，速度快。
  - **Druid**：阿里提供的数据库连接池，据说是集DBCP、C3P0、Proxool有点于一身的数据库连接池，但是速度不确定是否有BoneCP快
- DataSource通常被称为数据源，它包含连接池和连接池管理两个部分，习惯上也经常把DataSource称为连接池。
- DataSource用来取代DriverManager来获取Connection，获取速度快，同时可以大幅度提高数据库访问速度。
- 注意：
  - 数据源和数据库连接不同，数据源无需创建多个，它是生产数据库连接的工厂，因此整个应用只需要一个数据源即可。
  - 当数据库访问结束后，程序还是像以前一样关闭数据库连接：conn.close();但conn.close()并没有关闭数据库的物理连接，它仅仅把数据库连接释放，归还给数据库连接池。

### C3P0（连接池）

使用c3p0获取连接的方法

```java
public class C3P0Test1 {
//    方式一：【不推荐硬配置】
    @Test
    public void testGetConnection1() throws PropertyVetoException, SQLException {
        ComboPooledDataSource cpds = new ComboPooledDataSource();
        cpds.setDriverClass("com.mysql.jdbc.Driver");
//        url可以省略其中的很大一部简写为：jdbc:mysql:///test?characterEncoding=utf-8&rewriteBatchedStatements=true
        cpds.setJdbcUrl("jdbc:mysql://localhost:3306/test?characterEncoding=utf-8&?rewriteBatchedStatements=true");
        cpds.setUser("root");
        cpds.setPassword("root");
//      设置初始时数据库连接池中的连接数
        cpds.setInitialPoolSize(10);
        Connection conn = cpds.getConnection();
        System.out.println(conn);
//        销毁c3p0数据库连接池【一般不会关闭，因为服务器一直启动】
//        DataSources.destroy(cpds);
    }
//    方式二：【推荐，配置文件配置】
    @Test
    public void testGetConnection2() throws Exception {
        ComboPooledDataSource cpds = new ComboPooledDataSource("hellc3p0");
        Connection conn = cpds.getConnection();
        System.out.println(conn);
    }
}
```

方式二的配置文件：必须命名为**c3p0.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<c3p0-config>
<!--named-config的name：可自定义用于给c3p0的ComboPooledDataSource读取-->
    <named-config name="hellc3p0">
<!--        先是配置4个基本信息：JDBC驱动类、JDBC连接数据库的url、用户名、密码-->
        <property name="driveClass">com.mysql.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://localhost:3306/test?characterEncoding=utf-8</property>
        <property name="user">root</property>
        <property name="password">root</property>
<!--        数据库连接池管理的基本信息-->
<!--        当前数据库连接池中的连接数不够时，c3p0一次性向数据库服务器申请的连接数-->
        <property name="acquireIncrement">5</property>
<!--        c3p0数据库连接池中初始化时的连接数-->
        <property name="initialPoolSize">10</property>
<!--        c3p0数据库连接池维护的最少连接数-->
        <property name=" minPoolSize">10</property>
<!--        c3p0数据库连接池维护的最多的连接数-->
        <property name="maxPoolSize">100</property>
<!--        c3p0数据库连接池维护的最多的Statement的个数-->
        <property name="maxStatements">50</property>
<!--        每个连接中可以最多使用的Statement的个数-->
        <property name="maxStatementsPerConnection">2</property>

    </named-config>
</c3p0-config>
```

C3P0工具类的封装

```java
public class C3P0Util {
//    因此处是造连接池，不是获取连接，使用只需要执行一次，不需要反复造，所以提取出来
    private static ComboPooledDataSource cpds = new ComboPooledDataSource("hellc3p0");
    public static Connection getConn() throws SQLException {
        Connection conn = cpds.getConnection();
        return conn;
    }
}
```

C3P0的应用

```java
    @Test
    public void teseGetConn3() throws SQLException {
        Connection conn = C3P0Util.getConn();
        System.out.println(conn);
    }
```



### DBCP（连接池）

- 此连接池需要两个jar包，分别是commons-dbcp-1.4.jar和commins-pool-1.5.5.jar

使用DBCP获取连接的方法

```java
public class DBCPTest {
    /**
     * 测试DBCP连接池连接技术
     */

//    方式一：不推荐，硬编码配置信息
    @Test
    public void testGetConn1() throws SQLException {
//        创建DBCP连接池
        BasicDataSource source = new BasicDataSource();
//        设置基本信息，分别为JDBC驱动、数据库连接url、用户名、密码
        source.setDriverClassName("com.mysql.jdbc.Driver");
        source.setUrl("jdbc:mysql:///test?characterEncoding=utf-8&rewriteBatchedStatements=true");
        source.setUsername("root");
        source.setPassword("root");
//        数据库连接池管理的相关属性 等等
        source.setInitialSize(10);
        source.setMaxActive(10);
//        获取连接
        Connection connection = source.getConnection();
        System.out.println(connection);
    }
    //    方式二：推荐，配置文件配置信息
    @Test
    public void testGetConn2() throws SQLException, Exception {
//        创建一个Properties用于加载配置信息
        Properties pros=new Properties();

//        方式一：使用类的加载器加载配置文件获取输入流
//        InputStream is = ClassLoader.getSystemClassLoader().getResourceAsStream("dbcp.properties");
//        方式二：使用io流的文件输入流方法获取配置文件的输入流
        FileInputStream is = new FileInputStream(new File("src/dbcp.properties"));
//        加载配置文件的输入流
        pros.load(is);
        DataSource source = BasicDataSourceFactory.createDataSource(pros);
        Connection connection = source.getConnection();
        System.out.println(connection);
    }
}
```

方式二的配置文件：必须命名为**dbcp.properties**

```properties
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql:///test?characterEncoding=utf-8&rewriteBatchedStatements=true
username=root
password=root

```

DBCP工具类的封装

```java
public class DBCPUtil {
    //    因连接池只需要一个，不需要反复创建，所以提取出来
    private static DataSource source;
    static {
        try {
            //        创建一个Properties用于加载配置信息
            Properties pros=new Properties();
            //        方式一：使用类的加载器加载配置文件获取输入流
//        InputStream is = ClassLoader.getSystemClassLoader().getResourceAsStream("dbcp.properties");
//        方式二：使用io流的文件输入流方法获取配置文件的输入流
            FileInputStream is = new FileInputStream(new File("src/dbcp.properties"));
//        加载配置文件的输入流
            pros.load(is);
            source = BasicDataSourceFactory.createDataSource(pros);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public static Connection getConn() throws Exception {
            return source.getConnection();
    }
}
```

DBCP的应用

```java
    @Test
    public void testGetConn3() throws Exception {
        Connection conn = DBCPUtil.getConn();
        System.out.println(conn);
    }
```

### Druid（连接池）【译为：德鲁伊】

- 此连接池需要jar包：druid-1.1.10.jar

使用Druid获取连接的方法

```java
public class DruidTest {
    /**
     * 测试DBCP连接池连接技术
     */
//    方法一：不推荐，硬配置
    @Test
    public void getConn1() throws Exception {
        DruidDataSource source = new DruidDataSource();
        source.setDriverClassName("com.mysql.jdbc.Driver");
        source.setUrl("jdbc:mysql:///test?characterEncoding=utf-8&rewriteBatchedStatements=true");
        source.setUsername("root");
        source.setPassword("root");
        DruidPooledConnection connection = source.getConnection();
        System.out.println(connection);

    }
//    方法二：【推荐，使用配置文件连接】
    @Test
    public void getConn2() throws Exception {
        Properties pros = new Properties();
        InputStream is = ClassLoader.getSystemClassLoader().getResourceAsStream("druid.properties");
        pros.load(is);
        DataSource source = DruidDataSourceFactory.createDataSource(pros);
        Connection connection = source.getConnection();
        System.out.println(connection);
    }
}
```

方式二的配置文件：可自定义命名，但是建议命名为**druid.properties**

```properties
url=jdbc:mysql:///test?characterEncoding=utf-8&rewriteBatchedStatements=true
username=root
password=root
driverClassName=com.mysql.jdbc.Driver
```

Druid工具类的封装

```java
public class DruidUtil {
    //    因此处是造连接池，不是获取连接，使用只需要执行一次，不需要反复造，所以提取出来
    private static DataSource source=null;
    static {
        try {
            Properties pros = new Properties();
            InputStream is = ClassLoader.getSystemClassLoader().getResourceAsStream("druid.properties");
            pros.load(is);
            source = DruidDataSourceFactory.createDataSource(pros);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public static Connection getConn() throws Exception {
        Connection connection = source.getConnection();
        return connection;
    }
}
```

Druid的应用

```java
    @Test
    public void getConn3() throws Exception {
        Connection conn = DruidUtil.getConn();
        System.out.println(conn);
    }
```

## DButil

### 增删改

#### 使用QueryRunner中的update方法添加一条数据

```java
    @Test
    public  void  testInsert(){
        Connection conn = null;
        try {
//            获取dbutil中的QueryRunner
            QueryRunner runner = new QueryRunner();
            conn = DruidUtil.getConn();
            String sql="insert into customers(name,email,birth)values(?,?,?)";
//            使用QueryRunner中的update方法【此方法考虑了事务】进行添加数据，并返回影响了的数据行数
//            因考虑事务原因，所以连接需要自己传入
            int i = runner.update(conn, sql, "蔡徐坤", "cxk@qq.com", "1232-08-21");
            System.out.println("添加了"+i+"条记录");
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//            关闭连接
            JDBCUtils.closeResource(conn,null);
        }
    }
```

### 查询

- 

#### 使用QueryRunner中的Query方法查询customers表中的一条数据【BeanHandler：以bean对象形式返回数据】

```java
    /**
     * BeanHander:是RusultSetHandler接口的实现类，用于封装表中的一条记录
     */
    @Test
    public void testQuery1(){
        Connection conn = null;
        try {
//            获取dbutil中的QueryRunner
            QueryRunner runner = new QueryRunner();
            conn = DruidUtil.getConn();
            String sql="select id,name,email,birth from customers where id = ?";
//            创建一个Handler（译为：处理器或处理程序），当然Handler有很多，根据情况使用
//            此处的Handler为一个BeanHandler，用于处理一条有关Customer的数据，并以对像形式返回
//            BeanHandler中需要传入对应表的bean类
            BeanHandler<Customer> handler = new BeanHandler<>(Customer.class);
//            使用QueryRunner获取query方法，并返回数据，以对像形式返回
            Customer customer = runner.query(conn, sql, handler,13);
            System.out.println(customer);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//            关闭连接
            JDBCUtils.closeResource(conn,null);
        }
    }
```

#### 使用QueryRunner中的Query方法查询customers表中的多条数据【BeanListHandler：以list形式返回数据】

```java
	/**
     * BeanListHander:是RusultSetHandler接口的实现类，用于封装表中的多条记录
     */
    @Test
    public void testQuery2(){
        Connection conn = null;
        try {
//            获取dbutil中的QueryRunner
            QueryRunner runner = new QueryRunner();
            conn = DruidUtil.getConn();
            String sql="select id,name,email,birth from customers where id  < ?";
//            创建一个Handler（译为：处理器或处理程序），当然Handler有很多，根据情况使用
//            此处的Handler为一个BeanListHandler，用于处理多条有关Customer的数据，并以集合列表形式返回
//            BeanListHandler中需要传入对应表的bean类
            BeanListHandler<Customer> handler = new BeanListHandler<>(Customer.class);
//            使用QueryRunner获取query方法，并返回数据，以集合列表形式返回
            List<Customer> list = runner.query(conn, sql, handler, 13);
            list.forEach(System.out::println);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//            关闭连接
            JDBCUtils.closeResource(conn,null);
        }
    }
```

#### 使用QueryRunner中的Query方法查询customers表中的一条数据【MapHandler：以map形式返回数据（键值对）】

```java
    /**
     * MapHandler:是RusultSetHandler接口的实现类，对应表中的一条记录
     * 将字段及相应字段的值作为map中的key和value
     */
    @Test
    public void testQuery3(){
        Connection conn = null;
        try {
//            获取dbutil中的QueryRunner
            QueryRunner runner = new QueryRunner();
            conn = DruidUtil.getConn();
            String sql="select id,name,email,birth from customers where id  = ?";
//            创建一个Handler（译为：处理器或处理程序），当然Handler有很多，根据情况使用
//            此处的Handler为一个MapHandler，用于处理一条有关Customer的数据，并以map形式返回【键值对】
            MapHandler handler = new MapHandler();
//            使用QueryRunner获取query方法，并返回数据，以map形式返回【键值对】
            Map<String, Object> map = runner.query(conn, sql, handler, 13);
            System.out.println(map);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//            关闭连接
            JDBCUtils.closeResource(conn,null);
        }
    }
```

#### 使用QueryRunner中的Query方法查询customers表中的多条数据【MapHandler：以map形式返回数据（键值对）】

```java
    /**
     * MapListHandler:是RusultSetHandler接口的实现类，对应表中的多条记录
     * 将字段及相应字段的值作为map中的key和value并封装在list中
     */
    @Test
    public void testQuery4(){
        Connection conn = null;
        try {
//            获取dbutil中的QueryRunner
            QueryRunner runner = new QueryRunner();
            conn = DruidUtil.getConn();
            String sql="select id,name,email,birth from customers where id  < ?";
//            创建一个Handler（译为：处理器或处理程序），当然Handler有很多，根据情况使用
//            此处的Handler为一个MapListHandler，用于处理多条有关Customer的数据，并以list形式返回【键值对】
            MapListHandler handler = new MapListHandler();
//            使用QueryRunner获取query方法，并返回数据，以list形式返回【键值对】
            List<Map<String, Object>> maps = runner.query(conn, sql, handler, 13);
            maps.forEach(System.out::println);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//            关闭连接
            JDBCUtils.closeResource(conn,null);
        }
    }
```

#### 使用QueryRunner中的Query方法查询customers表中的特殊值【ScalarHandler：以强转类型的值为返回值的 类型】

```java
/**
     * ScalarHandler:是RusultSetHandler接口的实现类
     * ScalarHandler:用于查询特殊值如：使用sql函数查询得出的值或单个date类型的值等等。
     */
    @Test
    public void testQuery5(){
        Connection conn = null;
        try {
//            获取dbutil中的QueryRunner
            QueryRunner runner = new QueryRunner();
            conn = DruidUtil.getConn();
            String sql="select count(*) from customers";
//            创建一个Handler（译为：处理器或处理程序），当然Handler有很多，根据情况使用
//            此处的Handler为一个ScalarHandler，查询特殊值，类型需要字段强转成对应的类型。
            ScalarHandler handler = new ScalarHandler();
//            使用QueryRunner获取query方法，并返回数据，以自定义强转后的类型为返回的类型
            Long query = (Long) runner.query(conn, sql, handler);
            System.out.println(query);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//            关闭连接
            JDBCUtils.closeResource(conn,null);
        }
    }
```

#### 自定义ResultSetHandler的实现类

```java
    /**
     * 自定义ResultSetHandler的实现类
     */
    @Test
    public void testQuery7(){
        Connection conn = null;
        try {
//            获取dbutil中的QueryRunner
            QueryRunner runner = new QueryRunner();
            conn = DruidUtil.getConn();
            String sql="select count(*) from customers";
//            自定义handler，需要针对哪个表进行操作就传哪个表的bean类或类型类
            ResultSetHandler<Customer> handler = new ResultSetHandler<Customer>() {
                @Override
                public Customer handle(ResultSet resultSet) throws SQLException {
                    return null;
                }
            };
//            使用QueryRunner获取query方法，并返回数据，以自定义强转后的类型为返回的类型
            Customer query = runner.query(conn, sql, handler);
            System.out.println(query);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//            关闭连接
            JDBCUtils.closeResource(conn,null);
        }
    }
```

### 资源关闭

```java
public void close(Connection conn, PreparedStatement ps, ResultSet rs){
//        dbutil有两种关闭方法：一种相对麻烦，一种相对简单
        try {
            DbUtils.close(conn);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
            DbUtils.close(ps);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
            DbUtils.close(rs);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        DbUtils.closeQuietly(conn);
        DbUtils.closeQuietly(ps);
        DbUtils.closeQuietly(rs);
    }
```


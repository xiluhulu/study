# Jdbc

大致步骤

1. JDBC开始需要对工程进行导对应的JDBC驱动包
2. 使用驱动链接数据库
3. 操作数据
4. 关闭链接

## 数据库的链接

有四个方式可用于链接，只使用最终版

```java
@Test
public void testConnection5() throws Exception {
	//1.加载配置文件
	InputStream is =  ConnectionTest.class.getClassLoader().getResourceAsStream("jdbc.properties");
	Properties pros = new Properties();
	pros.load(is);
	//2.读取配置信息
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



配置文件：

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

数据库的增加操作：

```java
//    添加操作
    @Test
    public void insert() throws Exception {
        Connection conn = null;
        PreparedStatement ps = null;
        try {
//    获取类的加载器（加载配置文件）
           InputStream is = ClassLoader.getSystemClassLoader().getResourceAsStream("jdbc.properties");
            
//		   新建一个配置资源的对象 用于获取配置信息            
            Properties pros = new Properties();
//          使用配置对象加载配置文件
            pros.load(is);
//          获取单个配置信息
            String user = pros.getProperty("user");
            String password = pros.getProperty("password");
            String url = pros.getProperty("url");
            String driver = pros.getProperty("driver");
//        加载驱动
            Class.forName(driver);
//        获取链接(使用驱动获取链接)
            conn = DriverManager.getConnection(url, user, password);
//          sql语句
            String sql="insert into customers(name,email,birth)values(?,?,?)";
//        预编译sql语句返回prepareStatement实例
            ps = conn.prepareStatement(sql);
//        填充占位符
            ps.setString(1,"阿萨的");//用于填充占位符，数值指的是对第n个占位符的填充
            ps.setString(2,"aaa@asd.com");
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
            java.util.Date parse = sdf.parse("2021-01-01");
            ps.setDate(3,new Date(parse.getTime()));
//        执行操作
            ps.execute();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
//        关闭操作
            try {
//                判断避免空指针异常
                if (ps!=null)
                ps.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
//        关闭链接
            try {
//                判断避免空指针异常
                if (conn!=null)
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
```

对增删改的通用封装：

- 先封装配置文件的加载、提取配置文件信息、加载驱动、链接数据库等功能为一可调用的个方法
- 然后封装操作及资源关闭功能位一个可调用的方法
- 再然后对可变的sql语句及需要填充的参数另外封装
- 最后封装对增删改的操作为一可调用的个方法

工具类的封装：

```java
//封装成工具类
public class JDBCutils {
//此方法用于链接
    public static Connection getConnection() throws Exception {
//        配置文件的的加载
        InputStream resource = ClassLoader.getSystemClassLoader().getResourceAsStream("jdbc.properties");
//        资源解析器对象
        Properties properties = new Properties();
//        解析所加载的配置文件
        properties.load(resource);
//        进行信息提取
        String user = properties.getProperty("user");
        String password = properties.getProperty("password");
        String url = properties.getProperty("url");
        String driver = properties.getProperty("driver");
//加载驱动
        Class.forName(driver);
//链接
        Connection conn = DriverManager.getConnection(url, user, password);
//返回链接信息
        return conn;
    }
//    关闭资源操作
    public static void   closeResource(Connection conn, PreparedStatement ps){
        try {
//            关闭数据库操作
            if(ps!=null)
                ps.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
//            关闭链接
            if(conn!=null)
                conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
//    增删该的通用操作
    public static void zsg(String sql,Object ...args) throws Exception {
        Connection conn = null;
        PreparedStatement ps = null;
        try {
//        获取链接
            conn = getConnection();
//        预编译sql语句
            ps = conn.prepareStatement(sql);
//        填充占位符
            for (int i = 0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
//          对数据库执行sql语句
            ps.execute();
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            closeResource(conn,ps);
        }
    }
}

```

使用方法：

```java
public void fff() throws Exception {
    //通过sql语句执行增删改操作，不能执行查询
        String sql="update `order` set order_name=? where order_id=?";
        JDBCutils.zsg(sql,"asd","4");
    }
```

数据库的查询操作：

- 不同与增删改，查询操作有返回的结果集，也就是查询结果
- 结果集，需要使用javaBean对其进行暂时的存储并用与响应输出
  - javaBean的类名对应表的名字，属性名对应字段名
  - 包含空参构造器，有参构造器，get与set方法，toString方法
- 对数据库的执行方法不一样， 

查询操作（封装）：

```java
//    对表的查询一条数据的操作（链接和资源关闭调用了JDBC工具类，其中的资源关闭操作的方法改了）
    @Test
    public void testSelect()  {
        Connection conn = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        try {
//        获取链接
            conn = JDBCutils.getConnection();
//        预编译sql语句
            String sql="select id,name,email,birth from customers where id=?";
            preparedStatement = conn.prepareStatement(sql);
            preparedStatement.setObject(1,1);
//        执行并返回结果集（此方法与之前的增删改不一样,会有结果集返回）
            resultSet = preparedStatement.executeQuery();
//        结果集的处理
            if(resultSet.next()){//next()的作用：判断是否结果集是否有数据，如果有数据返回true，并指针下移；如果返回flase，指针不下移直接结束。
    //            获取当前这条数据各个的值
                int id = resultSet.getInt(1);
                String name = resultSet.getString(2);
                String email = resultSet.getString(3);
                Date birth = resultSet.getDate(4);
    //            方式一：(直接显示)
    //            System.out.println("id="+id+",name="+name+",email="+email+",birth="+birth);

    //            方式二：（使用数组）
//                Object[] objects =new Object[]{id, name, email, birth};
//                for (int i = 0 ; i < objects.length;i++){
//                    System.out.print(objects[i]+" ");
//                }
    //            方式三：将数据封装成一个对象（推荐）
                Customer customer = new Customer(id,name,email,birth);
                System.out.println(customer);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
//            关闭资源
            JDBCutils.selectCloseResource(conn,preparedStatement,resultSet);
        }
    }
```

查询的资源关闭操作

```java
 //    关闭资源操作(用于查询的关闭资源操作)
    public static void   selectCloseResource(Connection conn, PreparedStatement ps,ResultSet rs){
        try {
//            关闭数据库操作
            if(ps!=null)
                ps.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
//            关闭链接
            if(conn!=null)
                conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
            if (rs!=null)
                rs.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
```

查询通用操作的封装：

```java

```
















































































#### 1. 入门篇

需求： 

1. 根据用户id查询一个用户信息
2. 根据用户名称模糊查询用户信息列表
3. 插入用户信息
4. 根据用户id更新用户信息
5. 根据用户id删除用户信息

开工：

1. 新建maven工程mybatis-demo

   引入相关依赖（pom.xml）

   ```
   <dependencies>
           <!-- mybatis依赖 -->
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis</artifactId>
               <version>3.4.6</version>
           </dependency>
   
           <!-- mysql依赖 -->
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>5.1.35</version>
           </dependency>
   
           <!-- 单元测试 -->
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
           </dependency>
   
   				<!-- lombok插件 引入该依赖，需要在ide工具中安装对应的lombok插件-->
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
               <version>1.18.10</version>
           </dependency>
       </dependencies>
   ```

2. 新建SqlMapConfig.xml（mybatis的全局配置文件）

   ```
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration PUBLIC
           "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
   
       <!-- 数据库配置 -->
       <properties resource="db.properties"></properties>
       <environments default="development">
           <environment id="development">
               <transactionManager type="JDBC" />
               <dataSource type="POOLED">
                   <property name="driver" value="${db.driver}" />
                   <property name="url" value="${db.url}" />
                   <property name="username" value="${db.username}" />
                   <property name="password" value="${db.password}" />
               </dataSource>
           </environment>
       </environments>
   
       <!-- 对应的单个配置文件 -->
       <mappers>
           <mapper resource="UserMapper.xml" />
       </mappers>
   
   </configuration>
   ```

3. 新建db.properties文件，存储数据库的配置信息

   ```
   db.driver=com.mysql.jdbc.Driver
   db.url=jdbc:mysql://localhost:3306/ssm?characterEncoding=utf-8
   db.username=root
   db.password=123456
   ```

4. 新建数据库，并建立对应的用户表

   ```
   create table user (
   	id int(10) not null AUTO_INCREMENT,
   	username varchar(20),
   	birthday date,
   	sex varchar(1),
   	address varchar(50),
   	PRIMARY KEY (id)
   );
   ```

5. 对应的pojo

   ```
   // Data为lombok的注解，添加该注解会自动生成getter/setter/toString方法，简化程序
   @Data
   public class User implements Serializable {
   
       private static final long serialVersionUID = 2132125129327057568L;
   
       // 用户id
       private int id;
   
       // 用户名
       private String username;
   
       // 生日
       private Date birthday;
   
       // 性别
       private String sex;
   
       // 地址
       private String address;
   }
   ```

6. UserDao接口

   ```
   public interface UserDao {
   
       /**
        * 根据id查询用户
        * @param id
        * @return
        * @throws Exception
        */
       User findUserById(int id) throws Exception;
   
       /**
        * 根据用户名模糊查询用户信息
        * @param name
        * @return
        * @throws Exception
        */
       List<User> findUsersByName(String name) throws Exception;
   
       /**
        * 插入用户，并返回对应的用户id
        * @param user
        * @return
        * @throws Exception
        */
       Integer insertUser(User user) throws Exception;
   
       /**
        * 根据用户id更新用户信息
        * @param user
        * @throws Exception
        */
       void updateUser(User user) throws Exception;
   
       /**
        * 根据用户id删除用户信息
        * @param id
        * @return
        * @throws Exception
        */
       Integer deleteUser(Integer id) throws Exception;
   }
   ```

7. UserDaoImpl实现类

   ```
   public class UserDaoImpl implements UserDao {
   
       private SqlSessionFactory sqlSessionFactory;
   
       public UserDaoImpl (SqlSessionFactory sqlSessionFactory) {
           this.sqlSessionFactory = sqlSessionFactory;
       }
   
       @Override
       public User findUserById(int id) throws Exception {
           SqlSession session = sqlSessionFactory.openSession();
           User user = null;
           try {
   
               user = session.selectOne("test.findUserById", id);
               System.out.println(user);
           } finally {
               session.close();
           }
           return user;
       }
   
       @Override
       public List<User> findUsersByName(String name) throws Exception {
   
           SqlSession session = sqlSessionFactory.openSession();
           List<User> users = null;
           try {
               users = session.selectList("test.findUsersByName", name);
               System.out.println(users);
           } finally {
               session.close();
           }
           return users;
       }
   
       @Override
       public Integer insertUser(User user) throws Exception {
           SqlSession sqlSession = sqlSessionFactory.openSession();
           try {
               sqlSession.insert("test.insertUser", user);
               sqlSession.commit();
           } finally {
               sqlSession.close();
           }
           return user.getId();
       }
   
       @Override
       public void updateUser(User user) throws Exception {
           SqlSession sqlSession = sqlSessionFactory.openSession();
           try {
               sqlSession.update("test.updateUser", user);
               sqlSession.commit();
           } finally {
               sqlSession.close();
           }
       }
   
       @Override
       public Integer deleteUser(Integer id) throws Exception {
           SqlSession sqlSession = sqlSessionFactory.openSession();
           try {
               sqlSession.delete("test.deleteUser", id);
               sqlSession.commit();
           } finally {
               sqlSession.close();
           }
           return null;
       }
   }
   ```

8. UserMapper.xml

   ```
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="test">
   
       <!-- 根据id获取用户信息 -->
       <select id="findUserById" parameterType="int" resultType="com.mw.monster.po.User">
           select * from user where id = #{id}
       </select>
   
       <!-- 根据名称模糊查询用户列表 -->
       <select id="findUsersByName" parameterType="java.lang.String" resultType="com.mw.monster.po.User">
           select * from user where username like '%${value}%'
       </select>
   
       <!-- 插入用户 -->
       <insert id="insertUser" parameterType="com.mw.monster.po.User">
           <!-- 自增主键，存储到id字段中，直接使用user.getId()就可以获取到最新的id值 -->
           <selectKey keyProperty="id" order="AFTER" resultType="java.lang.Integer">
               select LAST_INSERT_ID()
           </selectKey>
           insert into user(username, birthday, sex, address) values (#{username}, #{birthday}, #{sex}, #{address})
       </insert>
   
       <!-- 更新用户 -->
       <update id="updateUser" parameterType="com.mw.monster.po.User">
           update user set username=#{username}, birthday=#{birthday},sex=#{sex},address=#{address} where id=#{id}
       </update>
   
       <delete id="deleteUser" parameterType="java.lang.Integer">
           delete from user where id =#{id}
       </delete>
   </mapper>
   ```

   配置说明：

   ```
   parameterType：定义输入参数的Java类型，
   
   resultType：定义结果映射类型。
   
   #{}：相当于JDBC中的？占位符
   
   #{id}表示使用preparedstatement设置占位符号并将输入变量id传到sql。
   
   ${value}：取出参数名为value的值。将${value}占位符替换。
   ```

   添加selectKey标签实现主键返回。

   ```
   keyProperty: 指定返回的主键，存储在pojo中的哪个属性
   order: selectKey标签中的sql的执行顺序，是相对与insert语句来说。由于mysql的自增原理，执行完insert语句 之后才将主键生成，所以这里selectKey的执行顺序为after。
   resultType: 返回的主键对应的JAVA类型
   LAST_INSERT_ID():是mysql的函数，返回auto_increment自增列新记录id值。
   ```

9. 测试类

   ```
   public class MybatisTest {
   
       private SqlSessionFactory sqlSessionFactory;
   
       @Before
       public void init() throws Exception {
           // 初始化：读取配置文件，获取连接数据库的信息，获取sqlSession
           SqlSessionFactoryBuilder sessionFactoryBuilder = new SqlSessionFactoryBuilder();
           InputStream inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
           sqlSessionFactory = sessionFactoryBuilder.build(inputStream);
       }
   
       @Test
       public void testFindUserById() throws Exception {
           UserDao userDao = new UserDaoImpl(sqlSessionFactory);
           User user = userDao.findUserById(1);
           System.out.println(user);
       }
   
       @Test
       public void testFindUsersByName() throws Exception {
           UserDao userDao = new UserDaoImpl(sqlSessionFactory);
           List<User> users = userDao.findUsersByName("MwMonster");
           System.out.println(users);
       }
   
       @Test
       public void testInsertUser() throws Exception {
           User user = new User();
           user.setUsername("MwMonster");
           user.setBirthday(new Date());
           user.setSex("1");
           user.setAddress("山西");
           UserDao userDao = new UserDaoImpl(sqlSessionFactory);
           Integer id = userDao.insertUser(user);
           System.out.println(id);
       }
   
       @Test
       public void testUpdateUser() throws Exception {
           User user = new User();
           user.setId(2);
           user.setUsername("小名");
           user.setBirthday(new Date());
           user.setAddress("山西");
           user.setSex("1");
   
           UserDao userDao = new UserDaoImpl(sqlSessionFactory);
           userDao.updateUser(user);
   
           User newUser = userDao.findUserById(2);
           System.out.println(newUser);
       }
   
       @Test
       public void testDeleteUser() throws Exception {
           UserDao userDao = new UserDaoImpl(sqlSessionFactory);
           userDao.deleteUser(2);
   
           User user = userDao.findUserById(2);
           System.out.println(user);
       }
   }
   ```


#### 2. XML开发方式

开发方式 只需要开发Mapper接口（dao接口）和Mapper映射文件，不需要编写实现类。

Mapper接口开发方式需要遵循以下规范： 

​	1、 Mapper接口的类路径与Mapper.xml文件中的namespace相同。

​	2、 Mapper接口方法名称和Mapper.xml中定义的每个statement的id相同。 

​	3、 Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql 的parameterType的类型相同。 

​	4、 Mapper接口方法的返回值类型和mapper.xml中定义的每个sql的resultType的类型相同。

1. mapper接口

   ```
   public interface StudentMapper {
       Integer insertStudent(Student student);
   
       Integer updateStudent(Student student);
   
       Integer deleteStudent(Integer id);
   
       Student getStudentByStudentNum(Integer studentNum);
   
       List<Student> getStudentsByName(String name);
   }
   ```

2. mapper.xml

   ```
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="com.mw.monster.dao.StudentMapper">
   
       <!-- 插入学生信息 -->
       <insert id="insertStudent" parameterType="com.mw.monster.po.Student">
           insert into student (name, student_num, address) values (#{name}, #{studentNum}, #{address})
       </insert>
   
       <!-- 更新学生信息 -->
       <update id="updateStudent" parameterType="com.mw.monster.po.Student">
           update student set name = #{name}, student_num = #{studentNum}, address = #{address} where id = #{id}
       </update>
   
       <!--- 删除学生信息 -->
       <delete id="deleteStudent" parameterType="java.lang.Integer">
           delete from student where id = #{id}
       </delete>
   
       <!-- 查询学生信息 -->
       <resultMap id="studentMap" type="com.mw.monster.po.Student">
           <id column="id" property="id"/>
           <result column="name" property="name"/>
           <result column="student_num" property="studentNum"/>
           <result column="address" property="address"/>
       </resultMap>
       <select id="getStudentByStudentNum" parameterType="java.lang.Integer" resultMap="studentMap">
           select * from student where student_num = #{studentNum}
       </select>
   
       <!-- 查询学生信息 -->
       <select id="getStudentsByName" parameterType="java.lang.String" resultMap="studentMap">
           select * from student where name = #{name}
       </select>
   </mapper>
   ```

   配置说明：

   **parameterType:**（输入类型）

   ​		可以映射的输入参数Java类型有：简单类型、POJO类型、Map类型、List类型

   **resultType**: (输出类型)

   ​		可以映射的Java类型有：简单类型、POJO类型、Map类型。

   ​		使用resultType进行输出映射时，要求sql语句中查询的列名和要映射的pojo属性一致。

   **resultMap**: 

   ​		如果sql查询出来的列名和pojo的属性名不一致，通过resultMap将列名和属性名作一个对应关系，最终将结果映射到pojo上。

   ```
   id: 主键
   column: 查询sql的列名
   property: 映射结果的属性名
   
   result: 映射查询结果的普通列
   ```

   

3. 在全局配置文件中添加该mapper文件

   ```
   <mappers>
      <mapper resource="StudentMapper.xml" />
   </mappers>
   ```

4. 测试类

   ```
   public class StudentMapperTest {
   
       private SqlSessionFactory sqlSessionFactory;
   
       @Before
       public void init() throws Exception {
           // 初始化：读取配置文件，获取连接数据库的信息，获取sqlSession
           SqlSessionFactoryBuilder sessionFactoryBuilder = new SqlSessionFactoryBuilder();
           InputStream inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
           sqlSessionFactory = sessionFactoryBuilder.build(inputStream);
       }
   
       @Test
       public void insertStudent() {
           Student student = new Student();
           student.setName("MwMonster");
           student.setStudentNum(123456);
           student.setAddress("山西");
           SqlSession session = sqlSessionFactory.openSession();
           StudentMapper mapper = session.getMapper(StudentMapper.class);
           Integer result = mapper.insertStudent(student);
           System.out.println(result);
           session.commit();
           session.close();
       }
   
       @Test
       public void updateStudent () {
           Student student = new Student();
           student.setId(1);
           student.setAddress("陕西");
           student.setStudentNum(123456);
           student.setName("MwMonster");
   
           SqlSession session = sqlSessionFactory.openSession();
           StudentMapper mapper = session.getMapper(StudentMapper.class);
           Integer result = mapper.updateStudent(student);
           System.out.println(result);
           session.commit();
           session.close();
       }
   
       @Test
       public void deleteStudent() {
           SqlSession session = sqlSessionFactory.openSession();
           StudentMapper mapper = session.getMapper(StudentMapper.class);
           Integer result = mapper.deleteStudent(1);
           System.out.println(result);
           session.commit();
           session.close();
       }
   
       @Test
       public void getStudentByStudentNum() {
           SqlSession session = sqlSessionFactory.openSession();
           StudentMapper mapper = session.getMapper(StudentMapper.class);
           Student student = mapper.getStudentByStudentNum(2222);
           System.out.println(student);
           session.close();
       }
   
       @Test
       public void getStudentsByName() {
           SqlSession session = sqlSessionFactory.openSession();
           StudentMapper mapper = session.getMapper(StudentMapper.class);
           List<Student> students = mapper.getStudentsByName("MwMonster");
           System.out.println(students);
           session.close();
       }
   }
   ```

#### 3. 注解开发方式

​	只需要编写接口类即可

1. MemberMapper接口

   ```
   public interface MemberMapper {
   
       @Insert("insert into member (username, password, phone) values (#{userName}, #{password}, #{phone})")
       Integer insertMember(Member member);
   
       @Update("update member set username =#{userName}, password = #{password}, phone = #{phone} where id = #{id}")
       Integer updateMember(Member member);
   
       @Delete("delete from member where id = #{id}")
       Integer deleteMember(Integer id);
   
       @Select("select * from member where id = #{id}")
       Member getMemberById(Integer id);
   
       @Select("select * from member where username = #{username}")
       List<Member> getMembersByName(String username);
   }
   ```

2. 在全局配置文件中添加该接口路径

   ```
   <mappers>
   		<mapper class="com.mw.monster.dao.MemberMapper"/>
   </mappers>
   ```

3. 测试类

   ```
   public class MemberMapperTest {
   
       private SqlSessionFactory sqlSessionFactory;
   
       @Before
       public void init() throws Exception {
           // 初始化：读取配置文件，获取连接数据库的信息，获取sqlSession
           SqlSessionFactoryBuilder sessionFactoryBuilder = new SqlSessionFactoryBuilder();
           InputStream inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
           sqlSessionFactory = sessionFactoryBuilder.build(inputStream);
       }
   
       @Test
       public void insertMember() {
           Member member = new Member();
           member.setUserName("MwMonster");
           member.setPassword("123456");
           member.setPhone("1234567890123");
           SqlSession session = sqlSessionFactory.openSession();
           MemberMapper mapper = session.getMapper(MemberMapper.class);
           Integer result = mapper.insertMember(member);
           System.out.println(result);
           session.commit();
           session.close();
       }
   
       @Test
       public void updateMember () {
           Member member = new Member();
           member.setId(1);
           member.setUserName("mwMonster");
           member.setPassword("111111");
           member.setPhone("18388888888");
   
           SqlSession session = sqlSessionFactory.openSession();
           MemberMapper mapper = session.getMapper(MemberMapper.class);
           Integer result = mapper.updateMember(member);
           System.out.println(result);
           session.commit();
           session.close();
       }
   
       @Test
       public void deleteMember() {
           SqlSession session = sqlSessionFactory.openSession();
           MemberMapper mapper = session.getMapper(MemberMapper.class);
           Integer result = mapper.deleteMember(1);
           System.out.println(result);
           session.commit();
           session.close();
       }
   
       @Test
       public void getMemberById() {
           SqlSession session = sqlSessionFactory.openSession();
           MemberMapper mapper = session.getMapper(MemberMapper.class);
           Member member = mapper.getMemberById(2);
           System.out.println(member);
           session.close();
       }
   
       @Test
       public void getMembersByName() {
           SqlSession session = sqlSessionFactory.openSession();
           MemberMapper mapper = session.getMapper(MemberMapper.class);
           List<Member> members = mapper.getMembersByName("MwMonster");
           System.out.println(members);
           session.close();
       }
   }
   ```

#### 4. 关联查询

**一对一查询**

​	需求：一个订单对应一个用户，查询订单信息及对应的用户信息

​	主信息：订单信息

​	从信息：用户信息

创建order类

```
@Data
public class Orders implements Serializable {

    private static final long serialVersionUID = 505095422284622013L;

    private Integer id;

    private Integer userId;

    private String orderNum;

    private Date createTime;

    private String description;
}
```

1. 创建订单用户信息(类中添加用户信息)

   ```
   @Data
   public class OrderExt extends Orders{
   
       private User user;
   
   		// 添加该方法可以直观的打印出订单信息和用户信息
       @Override
       public String toString() {
           return "OrderExt{" +
                   "user=" + user +
                   super.toString()+'}';
       }
   }
   ```

2. 创建OrderMapper接口

   ```
   /**
   * 查询订单信息及对应的用户信息
   * @return
   */
   List<OrderExt> getOrderExts();
   ```

3. 创建OrderMapper.xml

   ```
   <resultMap id="ordersAndUsersMap" type="com.mw.monster.po.OrderExt">
       <id column="id" property="id"/>
       <result column="user_id" property="userId"/>
       <result column="order_num" property="orderNum"/>
       <result column="create_time" property="createTime"/>
       <result column="description" property="description"/>
       <!-- 一对一关联映射 -->
       <!--
           property: OrderExt对象的user属性
           javaType: user属性对应的类型
       -->
       <association property="user" javaType="com.mw.monster.po.User">
           <!-- column: user表的对应的列 property: user对象对应的属性  -->
           <id column="id" property="id"/>
           <result column="username" property="username"/>
           <result column="birthday" property="birthday"/>
           <result column="sex" property="sex"/>
           <result column="address" property="address"/>
       </association>
   </resultMap>
   
   <select id="getOrderExts" resultMap="ordersAndUsersMap">
       select o.id, 0.user_id, o.create_time, o.description,
       u.username, u.birthday, u.sex, u.address
       from orders o
       join user u
       on u.id = o.user_id
   </select>
   ```

4. 测试类

   ```
   @Test
   public void getOrderExts() {
   		SqlSession session = sqlSessionFactory.openSession();
       OrderMapper mapper = session.getMapper(OrderMapper.class);
       List<OrderExt> orderExts = mapper.getOrderExts();
       System.out.println(orderExts);
       session.close();
   }
   ```

**一对多查询**

​	需求：一个用户对应多个订单，查询用户信息及名下的订单信息

​	主信息：用户信息

​	从信息：订单信息

1. 创建用户订单信息类

   ```
   @Data
   public class UserExt extends User{
   
       private List<Orders> ordersList;
   
   		// 直观的打印出用户信息及名下的订单信息
       @Override
       public String toString() {
           return "UserExt{" +
                   "ordersList=" + ordersList +
                   super.toString() + '}';
       }
   }
   ```

2. 创建接口

   ```
   /**
   * 查询用户及名下的订单信息
   * @return
   */
   List<UserExt> getUserExts();
   ```

3. 创建OrderMapper.xml

   ```
   <resultMap id="userAndOrdersMap" type="com.mw.monster.po.UserExt">
           <!-- 用户信息映射 -->
           <id column="id" property="id"/>
           <result column="username" property="username"/>
           <result column="birthday" property="birthday"/>
           <result column="sex" property="sex"/>
           <result column="address" property="address"/>
           <!-- 一对多关联映射 -->
           <collection property="ordersList" ofType="com.mw.monster.po.Orders">
               <id column="oid" property="id"/>
               <result column="user_id" property="userId"/>
               <result column="order_num" property="orderNum"/>
               <result column="create_time" property="createTime"/>
               <result column="description" property="description"/>
           </collection>
       </resultMap>
       <select id="getUserExts" resultMap="userAndOrdersMap">
           select u.*, o.id oid, o.user_id, o.order_num, o.create_time, o.description
           from user u
           left join orders o on u.id = o.user_id
       </select>
   ```

4. 测试类

   ```
   @Test
   public void getUserExts() {
       SqlSession session = sqlSessionFactory.openSession();
       OrderMapper mapper = session.getMapper(OrderMapper.class);
       List<UserExt> userExts = mapper.getUserExts();
       System.out.println(userExts);
       session.close();
   }
   ```

#### 5. 动态sql

1. if标签

   需求：页面传过来的查询条件可能输入用户名称，也可能不输入用户名称

   ```
   <select id="findUserList" parameterType="com.mw.monster.po.User" resultType="com.mw.monster.po.User">
   		select * from user where 1 = 1
   		<if test="username != null and username != ''">
   				and username like '%${username}%'
   		</if>
   </select>
   ```

2. where标签

   ​		上边的sql中的1=1，虽然可以保证sql语句的完整性，但是存在性能问题，mybatis提供where标签来解决该问题。

   ```
   <select id="findUserList2" parameterType="com.mw.monster.po.User" resultType="com.mw.monster.po.User">
       select * from user
       <where>
           <if test="username != null and username != ''">
           		and username like '%${username}%'
           </if>
       </where>
   </select>
   ```

3. sql片段

   在映射文件中可使用sql标签将重复的sql提取出来，然后使用include标签引用即可，最终达到sql重用的目的。

   ```
   <sql id="query_user_where">
       <if test="username != null and username != ''">
       and username like '%${username}%'
       </if>
   </sql>
   
   <select id="findUserList3" parameterType="com.mw.monster.po.User" 								resultType="com.mw.monster.po.User">
   		select * from user
   		<where>
   				<include refid="query_user_where"></include>
   		</where>
   </select>
   ```

   注： 如果引用其他mapper.xml的sql片段，则在引用时需加上namesapace

   ```
   <include refid="namespace.sql片段”/>
   ```

4. foreach

   ```
   <select id="findUserList4" parameterType="java.util.List" resultType="com.mw.monster.po.User">
           select * from user
           <where>
               <if test="list != null and list.size() > 0">
                   <foreach collection="list" item="id" open=" and id in ( "
                            close=" ) " separator=",">
                       #{id}
                   </foreach>
               </if>
           </where>
       </select>
   ```

   标签说明：

   ```
   collection: 指定输入的集合参数的参数名称
   item: 声明集合参数中的元素变量名
   open: 集合遍历时，需要拼接到遍历sql语句的前面
   close: 集合遍历时，需要拼接到遍历sql语句的后面
   separator: 集合遍历时，需要拼接到遍历sql语句之间的分隔符号
   ```

   注：如果parameterType不是pojo类型，是List或Array时，那么collection属性值固定为list或array。
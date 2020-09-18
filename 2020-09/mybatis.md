# MyBatis

### 1.什么是MyBatis？

MyBatis是一个基于sql开发的ORM持久层框架，其内部封装了jdbc，使得开发者只需要关注sql语句本身，不需要加载驱动，建立连接。它的前身是ibatis。

MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以对配置和原生Map使用简单的 XML 或注解，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

### 2.创建mybatis项目

（1）创建maven项目

（2）导入pom依赖

（3）创建指定的包结构

（4）创建核心配置文件mybatis.xml（名字可改）

（5）再利用反向生成工具生成mapper接口、映射文件、实体类

（6）通过工作原理测试

### 3.mybatis工作原理

（1）加载配置文件，通过InputStream

（2）创建sqlSessionFactory

（3）创建sqlSession对象

（4）通过sqlSession对象调用mapper接口的对象

（5）通过mapper接口对象，执行持久化操作

（6）事务提交或回滚

（7）关闭资源sqlSession.close()

**通过jdbc做事务**

### 4.mapper接口向XML文件传值和取值的方式

（1）传一个参数：直接通过#{任意名称}

如select id，name，pw from  表   where  id=#{xxxx}

（2）传多个参数：为接口中的方法参数加上注解@Param(别名)

然后在XML文件中就可以通过#{别名}获取到

```xml
//mapper接口中
HUser queryUserMess(@Param("un") String username, @Param("pw") String password);
//mapper.xml文件中
from H_USER
where username = #{un} and password = #{pw}
```

（3）传对象：

\#{}，括号里面写对象中的属性名

```xml
//mapper接口中
HUser login(HUser u);
//mapper.xml文件中
from H_USER where username=#{username} and password=#{password}
```

如果使用注解 @Param() 给对象设置了别名，就得通过#{别名.属性名}

```xml
//mapper接口中
HUser login(@Param("u") HUser u);
//mapper.xml文件中
from H_USER where username=#{u.username} and password=#{u.password}
```

（4）传递参数是List集合

（5）参数是Map

（6）参数是数组

### 5.mybatis的关联映射

描述表跟表之间的关系

1. 一对一
2. 一对多
3. 多对一
4. 多对多

实现关联映射步骤：

a.让表跟表建立关系  主外键

b.让类跟类建立关系  添加关联的属性

c.修改对应的映射文件（resultMap，修改sql）

#### 多对一

建立HUser和HDept的多对一关系，实现关联映射两种方式：

1.通过一个sql语句做多表查询，将所有的数据全部查询出来，然后再进行封装

先在HUser类中配置：

```java
private HDept dept;

public HDept getDept() {
    return dept;
}

public void setDept(HDept dept) {
    this.dept = dept;
}
```

在HUserMapper.xml中配置：

注意：如果做多表连接，存在列名重复，需要为不同列取别名来区分

```xml
<resultMap id="MyResultMap" type="com.seecen.sc1911.pojo.HUser">
  <id column="userid" property="id"/>
  <result column="username" property="username"/>
  <result column="password" property="password"/>
  <result column="createtime" property="createtime"/>
  <result column="did" property="did"/>
  <!-- association关联的是一个对象
   property：关联的属性名
   javaType：关联的属性类型
   第一种通过多表连接的方式实现n：1 映射
   -->
  <association property="dept" javaType="com.seecen.sc1911.pojo.HDept">
    <id column="deptid" property="id"/>
    <result column="deptname" property="deptname"/>
  </association>
</resultMap>

<sql id="userAndDept">
  u.id userid,u.username,u.password,u.createtime,u.did,d.id deptid,d.deptname
</sql>
<!-- 关联查询的sql语句 -->
<select id="queryUserDept" resultMap="MyResultMap">
  select
  <include refid="userAndDept"/>
  from H_USER u, H_DEPT d where u.did=d.id
</select>
```

查询结果：

![查询结果](.\001.png)

2.分两个sql语句查询

```xml
<resultMap id="MyResultMap" type="com.seecen.sc1911.pojo.HUser">
  <id column="id" property="id"/>
  <result column="username" property="username"/>
  <result column="password" property="password"/>
  <result column="createtime" property="createtime"/>
  <result column="did" property="did"/>
  <association property="dept" column="did"
             select="com.seecen.sc1911.mapper.HDeptMapper.selectByPrimaryKey"></association>
</resultMap>

<select id="queryUserBase" resultMap="MyResultMap">
  select
  id, username, password, createtime, did
  from H_USER where id=#{id}
</select>
```

第一次：先将HUser查询出来，得到它的did字段，将did的值作为参数传入HDeptMapper的selectByPrimaryKey(short id)方法中，第二次：查询出对应的HDept信息

![结果2](.\002.png)

#### 多对多

建立HStudent和HTeacher的多对多关系

**第一种方式**

1.首先要建立三张表：

student表

teacher表

stu_tea表

第三张表的字段有两个，一个是student表的主键，一个是teacher表的主键



2.在类中表示他们的关系

HTeacher.java

```java
private List<HStudent> listStudent;//一个老师对应多个学生

public List<HStudent> getListStudent() {
    return listStudent;
}

public void setListStudent(List<HStudent> listStudent) {
    this.listStudent = listStudent;
}
```

HStudent.java

```java
private List<HTeacher> teacherList;//一个学生对应多个老师

public List<HTeacher> getTeacherList() {
    return teacherList;
}

public void setTeacherList(List<HTeacher> teacherList) {
    this.teacherList = teacherList;
}
```

3.在mapper.xml文件中配置

```xml
<resultMap id="BaseResultMap" type="com.seecen.sc1911.pojo.HStudent">
  <id column="ID" jdbcType="DECIMAL" property="id" />
  <result column="NAME" jdbcType="VARCHAR" property="name" />
  
   <!--第一种方式 -->
  <collection property="teacherList" ofType="com.seecen.sc1911.pojo.HTeacher">
    <id column="teacher_id" property="id"/>
    <result column="teacher_name" property="name"/>
  </collection>
</resultMap>

<select id="queryStudentTeacher" resultMap="BaseResultMap">
  <!-- 三表关联 -->
  select
  s.id,s.name,t.id teacher_id,t.name teacher_name
  from H_STUDENT s, H_TEACHER t, STU_TEA st
  where s.id=st.sid and t.id=st.tid and s.id=#{id}
</select>
```

4.在HStudentMapper.java中写一个抽象方法

HStudent queryStudentTeacher(short id);

5.写一个测试方法

```java
public class TestStudent {
    SqlSession session;
    @Before
    public void init() throws IOException {
        InputStream is = Resources.getResourceAsStream("mybatis.xml");
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
        session = factory.openSession();
    }
    @After
    public void destroy(){
        session.commit();
        session.close();
    }
    @Test
    public void test1(){
        HStudentMapper mapper = session.getMapper(HStudentMapper.class);
        HStudent stu = mapper.queryStudentTeacher((short) 1);
        System.out.println(stu);
        System.out.println(stu.getTeacherList());
    }
}
```

**第二种方式**

在HStudentMapper.xml中配置

```xml
<resultMap id="BaseResultMap" type="com.seecen.sc1911.pojo.HStudent">
  <id column="ID" jdbcType="DECIMAL" property="id" />
  <result column="NAME" jdbcType="VARCHAR" property="name" />
  <!-- 第二种方式 -->
  <collection property="teacherList" column="id"
              select="com.seecen.sc1911.mapper.HTeacherMapper.queryTeachers">
  </collection>
</resultMap>
```

在HTeacherMapper.xml中配置

```xml
<select id="queryTeachers" resultMap="BaseResultMap">
  select t.id,t.name from H_TEACHER t, STU_TEA st
  where t.id=st.tid and st.sid=#{id}
</select>
```

### 6.MyBatis在mapper中如何传递多个参数？

第1种：使用占位符的思想，在映射文件中使用#{arg0}，#{arg1}代表传递进来的参数。

使用@param注解:来命名参数

```java
//HUserMapper.java
HUser queryUserMess(String username, String password);

//在HUserMapper.xml中
//1
where username = #{arg0} and password = #{arg1}

//2
where username = #{param1} and password = #{param2}
```

`更推荐使用下面这种，给参数起别名`

```java
//方法参数上加入@param注解
public interface usermapper {
    user selectuser(@param(“username”)String username, @param(“password”)String password);
}
// xml文件中
<select id=”selectUser” resulttype=”user”> 
    select id, username, password from user where username = #{username} and password = #{password} 
</select>
```

第2种：使用Map集合作为参数来装载

mapper接口中

```java
//动态查询，根据map集合
List<HUser> queryMapSearch(Map<String, Object> map);
```

映射文件中

```xml
<select id="queryMapSearch" resultType="com.seecen.sc1911.pojo.HUser">
  select
  <include refid="Base_Column_List"/>
  from H_USER
  <where>
    <!-- 当传入参数是map时，test的条件是map中的key -->
    <if test="likeName != null">
      and username=#{likeName}
    </if>
    <if test="likePassword != null">
      and password=#{likePassword}
    </if>
  </where>
</select>
```

### 7.mybatis的缓存机制



**缓存的目的：**

提高速度，节省资源，因为查询的时候会先查看缓存，如果存在就是直接使用，不需要连接数据库，如果没有再使用sql语句查询数据库。



**mybatis缓存分为两种**

1.一级缓存：sqlSession级别的缓存，默认开启，在同一个sqlSession中有效。对数据进行更新、插入、删除操作时，会清空缓存，为了防止脏读。

2.二级缓存：为mapper级别的缓存，不同的sqlSession会共享二级缓存，默认不开启，需要手动配置。通过配置映射文件，或者写注解的方式，来配置二级缓存。



**二级缓存**



1.导入jar包

2.在对应的映射文件中加一个标签<cache 属性="">

​    也可以在mapper接口中添加注解

### 8.mybatis的动态sql

mybatis动态sql是通过使用类似jstl标签的方式来根据不同条件产生不同的sql语句

```xml
<update id="updateByPrimaryKeySelective" parameterType="com.seecen.sc1911.pojo.HUser">
  update H_USER
  <!-- 根据传递的参数对象中的属性是否为null，来确定如何修改数据
      if标签中判断的条件是参数对象中的属性
   -->
  <set>
    <if test="username != null">
      USERNAME = #{username,jdbcType=VARCHAR},
    </if>
    <if test="password != null">
      PASSWORD = #{password,jdbcType=VARCHAR},
    </if>
    <if test="createtime != null">
      CREATETIME = #{createtime,jdbcType=TIMESTAMP},
    </if>
    <if test="did != null">
      DID = #{did,jdbcType=DECIMAL},
    </if>
  </set>
  where ID = #{id,jdbcType=DECIMAL}
</update>
```

几种标签的用法：

1.if  经常用来判断属性值是否为空 或 空字符串

```xml
<if test="属性!=null">  </if>
```

2.where    类似于 where  1=1，mybatis会根据是否存在条件动态地添加where关键字

```xml
<select id="querySearch" resultType="com.seecen.sc1911.pojo.HUser">
  select
  <include refid="Base_Column_List"/>
  from H_USER
  <where>
    <if test="id != null">
      and id=#{id}
    </if>
    <if test="username != null">
      and username=#{username}
    </if>
    <if test="password!=null">
      and password=#{password}
    </if>
  </where>
</select>
```

3.foreach

写一个批量删除

HUserMapper.java

```java
//传入参数是集合，实现批量删除
void deleteAll(List<Short> ids);
```

HUserMapper.xml

```xml
<delete id="deleteAll">
  <!-- delete from 表 where id in (?,?,...) -->
  delete
  from H_USER where id in
  <!-- 使用foreach遍历集合中的参数
      collection：指遍历的集合，值一般是map、list、array
      item：集合遍历出来的临时变量，类似于jstl中的var
      separator：指定数据项之间的分隔符
      open：指定开始位置的内容
      close：指定结束的内容
   -->
  <foreach collection="list" item="uid" separator="," open="(" close=")">
    #{uid}
  </foreach>
</delete>
```

4.set  类似于更新语句中的set关键字

5.trim   类似于字符串拼接

6.bind：用来获取传递的参数并对其做一些操作，然后生成一个新的变量。经常用于模糊查询

```xml
<if test="username != null">
  <bind name="un" value="'%'+username+'%'"></bind>
  and username like #{un}
</if>
```


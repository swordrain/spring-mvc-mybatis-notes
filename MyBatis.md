## MyBatis入门

**创建Maven项目**

**修改`pom`文件**

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
	                    http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>tk.mybatis</groupId>
	<artifactId>simple</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<properties>
		<java.version>1.6</java.version>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>

	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>3.3.0</version>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.38</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>1.7.12</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.7.12</version>
		</dependency>
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>1.2.17</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<configuration>
					<source>${java.version}</source>
					<target>${java.version}</target>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
```

**准备数据库**

**配置MyBatis**

在`src/main/resources`下面创建`mybatis-config.xml`

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="logImpl" value="LOG4J"/>
    </settings>
    
     <typeAliases>
        <package name="tk.mybatis.simple.model"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC">
                <property name="" value=""/>
            </transactionManager>
            <dataSource type="UNPOOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://192.168.16.137:3306/mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value=""/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="tk/mybatis/simple/mapper/CountryMapper.xml"/>
    </mappers>
</configuration>

```

**创建POJO**

**Mapper.xml文件**

在`src/main/resources`创建`tk/mybatis/simple/mapper`目录，然后创建`CountryMapper.xml`文件

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
					"http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="tk.mybatis.simple.mapper.CountryMapper">
	<select id="selectAll" resultType="Country">
		select id,countryname,countrycode from country
	</select>
</mapp
```

**配置Log4j**

在`src/main/resources`中添加`log4j.properties`配置文件

```
#全局配置
log4j.rootLogger=ERROR, stdout

#MyBatis 日志配置
log4j.logger.tk.mybatis.simple.mapper=TRACE

#控制台输出配置
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

**代码**

```
package tk.mybatis.simple.mapper;

import java.io.IOException;
import java.io.Reader;
import java.util.List;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.BeforeClass;
import org.junit.Test;

import tk.mybatis.simple.model.Country;

public class CountryMapperTest {
	
	private static SqlSessionFactory sqlSessionFactory;
	
	@BeforeClass
	public static void init(){
		try {
            Reader reader = Resources.getResourceAsReader("mybatis-config.xml");
            //InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml")
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
            reader.close();
        } catch (IOException ignore) {
        	ignore.printStackTrace();
        }
	}
	
	@Test
	public void testSelectAll(){
		SqlSession sqlSession = sqlSessionFactory.openSession();
		try {
			List<Country> countryList = sqlSession.selectList("selectAll");
			printCountryList(countryList);
		} finally {
			sqlSession.close();
		}
	}
	
	private void printCountryList(List<Country> countryList){
		for(Country country : countryList){
			System.out.printf("%-4d%4s%4s\n",country.getId(), country.getCountryname(), country.getCountrycode());
		}
	}
}
```

**运行结果**

```
DEBUG [main] - ==>  Preparing: select id, countryname, countrycode from country 
DEBUG [main] - ==> Parameters: 
TRACE [main] - <==    Columns: id, countryname, countrycode
TRACE [main] - <==        Row: 1, 中国, CN
TRACE [main] - <==        Row: 2, 美国, US
TRACE [main] - <==        Row: 3, 英国, GB
TRACE [main] - <==        Row: 4, 俄罗斯, RU
TRACE [main] - <==        Row: 5, 法国, FR
DEBUG [main] - <==      Total: 5
1     中国  CN
2     美国  US
3     英国  GB
4    俄罗斯  RU
5     法国  FR
```

## MyBatis基本用法
### MyBatis的体系结构
**SqlSessionFactory**

从XML文件或`Configuration`的实例构建出来，建议单例，常用的方法`openSession`

```
SqlSession openSession()
SqlSession openSession(boolean autoCommit)
SqlSession openSession(Connection connection)
SqlSession openSession(TransactionIsolationLevel level)
SqlSession openSession(ExecutorType execType,TransactionIsolationLevel level)
SqlSession openSession(ExecutorType execType)
SqlSession openSession(ExecutorType execType, boolean autoCommit)
SqlSession openSession(ExecutorType execType, Connection connection)
```

[参考](http://www.mybatis.org/mybatis-3/zh/java-api.html#SqlSessionFactory)

**SqlSession**

是持久化操作的对象，线程不安全，不要放入静态字段或者实例字段中，也不要放入任何类型的管理范围中比如Session，用完之后确保关闭

常用的方法

* insert
* update
* delete 
* selectOne
* selectList
* selectMap
* select
* commit
* rollback
* close
* getConnection
* getMapper

[参考](http://www.mybatis.org/mybatis-3/zh/java-api.html#SqlSession)

### MyBatis配置文件
SqlSessionFactoryBuilder根据传入的数据流生成`Configuration`对象，然后根据`Configuration`对象创建默认的`SqlSessionFactory`实例

1. 调用`SqlSessionFactoryBuilder`对象的`build`方法
2. `SqlSessionFactoryBuilder`会根据输入流信息创建`XMLConfigBuilder`对象
3. `SqlSessionFactoryBuilder`调用`XMLConfigBuilder`对象的`parse`方法
4. `XMLConfigBuilder`对象解析XML配置文件返回`Configuration`对象
5. `SqlSessionFactoryBuilder`根据`Configuration`对象创建一个`DefaultSessionFactory`对象
6. `SqlSessionFactoryBuilder`返回`DefaultSessionFactory`对象

**配置文件结构**

[参考](http://www.mybatis.org/mybatis-3/zh/configuration.html)

**Mapper XML映射文件**

[参考](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html)

## MyBatis的关联映射和动态SQL
### MyBatis的关系映射
**一对一**

假设person(人)与card(身份证)是一对一的关系

建表语句，注意card_id有UNIQUE属性

```
CREATE TABLE tb_card (
id INT PRIMARY KEY AUTO_INCREMENT,
CODE VARCHAR(18)
);

CREATE TABLE tb_person(
id INT PRIMARY KEY AUTO_INCREMENT,
NAME VARCHAR(18),
sex VARCHAR(18),
card_id INT UNIQUE,
FOREIGN KEY (card_id) REFERENCES tb_card (id)
);
```

POJO

```
public class Card implements Serializable {
	
	private Integer id;  // 主键id
	private String code; // 身份证编号
	...
}

public class Person implements Serializable {

	private Integer id;  // 主键id
	private String name; // 姓名
	private String sex;  // 性别
	private Integer age; // 年龄
	
	// 人和身份证是一对一的关系，即一个人只有一个身份证
	private Card card; 
	...
}
```

映射文件

```
//CardMapper.xml
<mapper namespace="org.fkit.mapper.CardMapper">

	<!-- 根据id查询Card，返回Card对象 -->
  <select id="selectCardById" parameterType="int" resultType="org.fkit.domain.Card">
  	SELECT * from tb_card where id = #{id} 
  </select>
  
</mapper>

//PersonMapper.xml
<mapper namespace="org.fkit.mapper.PersonMapper">

	<!-- 根据id查询Person，返回resultMap -->
  <select id="selectPersonById" parameterType="int" 
  	resultMap="personMapper">
  	SELECT * from tb_person where id = #{id} 
  </select>

  <!-- 映射Peson对象的resultMap -->
	<resultMap type="org.fkit.domain.Person" id="personMapper">
		<id property="id" column="id"/>
		<result property="name" column="name"/>
		<result property="sex" column="sex"/>
		<result property="age" column="age"/>
		<!-- 一对一关联映射:association   -->
		<association property="card" column="card_id"
		select="org.fkit.mapper.CardMapper.selectCardById" 
		javaType="org.fkit.domain.Card"/>
	</resultMap>

</mapper>
```

创建Mapper接口，Mapper接口对象的类名必须和XML的mapper的`namespace`一致，而方法名和参数必须和`<select />`的`id`属性和`parameterType`属性一致

```
public interface PersonMapper {
	
	/**
	 * 根据id查询Person
	 * 方法名和参数必须和XML文件中的<select.../>元素的id属性和parameterType属性一致
	 * @param id
	 * @return Person对象
	 * */
	Person selectPersonById(Integer id);
	
}
```

测试类

```
public class OneToOneTest {

	public static void main(String[] args) throws Exception {
		// 读取mybatis-config.xml文件
		InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
		// 初始化mybatis，创建SqlSessionFactory类的实例
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder()
				.build(inputStream);
		// 创建Session实例
		SqlSession session = sqlSessionFactory.openSession();
		
		/*Person p 
		= session.selectOne("org.fkit.mapper.PersonMapper.selectPersonById",1);
		System.out.println(p);
		System.out.println(p.getCard().getCode());*/
		// 获得mapper接口的代理对象
		PersonMapper pm = session.getMapper(PersonMapper.class);
		// 直接调用接口的方法，查询id为1的Peson数据
		Person p = pm.selectPersonById(1);
		// 打印Peson对象
		System.out.println(p);
		// 打印Person对象关联的Card对象
		System.out.println(p.getCard());
		
		// 提交事务
		session.commit();
		// 关闭Session
		session.close();
	}
}
```

运行结果

```
DEBUG [main] - ==>  Preparing: SELECT * from tb_person where id = ? 
DEBUG [main] - ==> Parameters: 1(Integer)
TRACE [main] - <==    Columns: id, NAME, sex, age, card_id
TRACE [main] - <==        Row: 1, jack, 男, 23, 1
DEBUG [main] - ====>  Preparing: SELECT * from tb_card where id = ? 
DEBUG [main] - ====> Parameters: 1(Integer)
TRACE [main] - <====    Columns: id, CODE
TRACE [main] - <====        Row: 1, 432801198009191038
DEBUG [main] - <====      Total: 1
DEBUG [main] - <==      Total: 1
Person [id=1, name=jack, sex=男, age=23]
Card [id=1, code=432801198009191038]
```

**一对多**

班级和学生的关系

建表语句

```
 CREATE TABLE tb_clazz(
id INT PRIMARY KEY AUTO_INCREMENT,
CODE VARCHAR(18),
NAME VARCHAR(18)
);

INSERT INTO tb_clazz(CODE,NAME) VALUES('j1601','Java就业班');

CREATE TABLE tb_student(
id INT PRIMARY KEY AUTO_INCREMENT,
NAME VARCHAR(18),
sex VARCHAR(18),
age INT,
clazz_id INT,
FOREIGN KEY (clazz_id) REFERENCES tb_clazz(id)
);

INSERT INTO tb_student(NAME,sex,age,clazz_id) VALUES('jack','男',23,1);
INSERT INTO tb_student(NAME,sex,age,clazz_id) VALUES('rose','女',18,1);
INSERT INTO tb_student(NAME,sex,age,clazz_id) VALUES('tom','男',21,1);
INSERT INTO tb_student(NAME,sex,age,clazz_id) VALUES('alice','女',20,1);
```

POJO

```
public class Student implements Serializable {

	private Integer id; // 学生id，主键
	private String name; // 姓名
	private String sex;  // 性别
	private Integer age; // 年龄
	
	// 学生和班级是多对一的关系，即一个学生只属于一个班级
	private Clazz clazz;
	...
}

public class Clazz implements Serializable {
	
	private Integer id; // 班级id，主键
	private String code; // 班级编号
	private String name; // 班级名称
	
	// 班级和学生是一对多的关系，即一个班级可以有多个学生
	private List<Student> students;
	...
}
```

Mapper文件

```
//ClazzMapper
<mapper namespace="org.fkit.mapper.ClazzMapper">

	<!-- 根据id查询班级信息，返回resultMap -->
	  <select id="selectClazzById" parameterType="int" resultMap="clazzResultMap">
	  	SELECT * FROM tb_clazz  WHERE id = #{id}
	  </select>
	  
	   <!-- 映射Clazz对象的resultMap -->
	<resultMap type="org.fkit.domain.Clazz" id="clazzResultMap">
		<id property="id" column="id"/>
		<result property="code" column="code"/>
		<result property="name" column="name"/>
		<!-- 一对多关联映射:collection fetchType="lazy"表示懒加载  -->
		<collection property="students" javaType="ArrayList"
	  column="id" ofType="org.fkit.domain.Student"
	  select="org.fkit.mapper.StudentMapper.selectStudentByClazzId"
	  fetchType="lazy">
	  	<id property="id" column="id"/>
	  	<result property="name" column="name"/>
	  	<result property="sex" column="sex"/>
	  	<result property="age" column="age"/>
	  </collection>
	</resultMap>
</mapper>

//StudentMapper
<mapper namespace="org.fkit.mapper.StudentMapper">

	<!-- 根据id查询学生信息，多表连接，返回resultMap -->
  <select id="selectStudentById" parameterType="int" resultMap="studentResultMap">
  	SELECT * FROM tb_clazz c,tb_student s
  	WHERE c.id = s.clazz_id
  	 AND s.id = #{id}
  </select>
  
  <!-- 根据班级id查询学生信息，返回resultMap -->
  <select id="selectStudentByClazzId" parameterType="int" 
  resultMap="studentResultMap">
  	SELECT * FROM tb_student WHERE clazz_id = #{id}
  </select>
  
   <!-- 映射Student对象的resultMap -->
	<resultMap type="org.fkit.domain.Student" id="studentResultMap">
		<id property="id" column="id"/>
	  	<result property="name" column="name"/>
	  	<result property="sex" column="sex"/>
	  	<result property="age" column="age"/>
		<!-- 多对一关联映射:association   -->
		<association property="clazz" javaType="org.fkit.domain.Clazz">
			<id property="id" column="id"/>
			<result property="code" column="code"/>
			<result property="name" column="name"/>
		</association>
	</resultMap>

</mapper>
```

mybatis-config.xml中要开启懒加载

```
<setting name="logImpl" value="LOG4J"/>
	<!-- 要使延迟加载生效必须配置下面两个属性 -->
	<setting name="lazyLoadingEnabled" value="true"/>
	<setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

Mapper接口

```
public interface ClazzMapper {

	// 根据id查询班级信息
	Clazz selectClazzById(Integer id);
	
}

public interface StudentMapper {

	// 根据id查询学生信息
	Student selectStudentById(Integer id);
	
}
```

测试类

```
public class OneToManyTest {

	public static void main(String[] args) throws Exception {
		// 读取mybatis-config.xml文件
		InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
		// 初始化mybatis，创建SqlSessionFactory类的实例
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder()
				.build(inputStream);
		// 创建Session实例
		SqlSession session = sqlSessionFactory.openSession();
		
		OneToManyTest t = new OneToManyTest();
		
		t.testSelectClazzById(session);
//		t.testSelectStudentById(session);
		
		// 提交事务
		session.commit();
		// 关闭Session
		session.close();
	}
	
	// 测试一对多，查询班级Clazz（一）的时候级联查询学生Student（多）  
	public void testSelectClazzById(SqlSession session){
		// 获得ClazzMapper接口的代理对象
		ClazzMapper cm = session.getMapper(ClazzMapper.class);
		// 调用selectClazzById方法
		Clazz clazz = cm.selectClazzById(1);
		// 查看查询到的clazz对象信息
		System.out.println(clazz.getId() + " "+ clazz.getCode() + " "+clazz.getName());
		// 查看clazz对象关联的学生信息
		List<Student> students = clazz.getStudents();
		for(Student stu : students){
			System.out.println(stu);
		}
	}
	
	// 测试多对一，查询学生Student（多）的时候级联查询 班级Clazz（一）
	public void testSelectStudentById(SqlSession session){
		// 获得StudentMapper接口的代理对象
		StudentMapper sm = session.getMapper(StudentMapper.class);
		// 调用selectStudentById方法
		Student stu = sm.selectStudentById(1);
		// 查看查询到的Student对象信息
		System.out.println(stu);
		// 查看Student对象关联的班级信息
		System.out.println(stu.getClazz());
	}

}
```

由于是懒加载，当Clazz查询后不访问Student时，并不会去查询Student表，如果访问Student，则执行两次查询，而`testSelectStudentById`方法里执行的是一个关联查询

结果


```
//没有调用List<Student> students = clazz.getStudents();时
DEBUG [main] - ==>  Preparing: SELECT * FROM tb_clazz WHERE id = ? 
DEBUG [main] - ==> Parameters: 1(Integer)
TRACE [main] - <==    Columns: id, CODE, NAME
TRACE [main] - <==        Row: 1, j1601, Java就业班
DEBUG [main] - <==      Total: 1
1 j1601 Java就业班

//调用List<Student> students = clazz.getStudents();后
DEBUG [main] - ==>  Preparing: SELECT * FROM tb_clazz WHERE id = ? 
DEBUG [main] - ==> Parameters: 1(Integer)
TRACE [main] - <==    Columns: id, CODE, NAME
TRACE [main] - <==        Row: 1, j1601, Java就业班
DEBUG [main] - <==      Total: 1
1 j1601 Java就业班
DEBUG [main] - ==>  Preparing: SELECT * FROM tb_student WHERE clazz_id = ? 
DEBUG [main] - ==> Parameters: 1(Integer)
TRACE [main] - <==    Columns: id, NAME, sex, age, clazz_id
TRACE [main] - <==        Row: 1, jack, 男, 23, 1
TRACE [main] - <==        Row: 2, rose, 女, 18, 1
TRACE [main] - <==        Row: 3, tom, 男, 21, 1
TRACE [main] - <==        Row: 4, alice, 女, 20, 1
DEBUG [main] - <==      Total: 4
Student [id=1, name=jack, sex=男, age=23]
Student [id=2, name=rose, sex=女, age=18]
Student [id=3, name=tom, sex=男, age=21]
Student [id=4, name=alice, sex=女, age=20]

//testSelectStudentById的输出
DEBUG [main] - ==>  Preparing: SELECT * FROM tb_clazz c,tb_student s WHERE c.id = s.clazz_id AND s.id = ? 
DEBUG [main] - ==> Parameters: 1(Integer)
TRACE [main] - <==    Columns: id, CODE, NAME, id, NAME, sex, age, clazz_id
TRACE [main] - <==        Row: 1, j1601, Java就业班, 1, jack, 男, 23, 1
DEBUG [main] - <==      Total: 1
Student [id=1, name=Java就业班, sex=男, age=23]
Clazz [id=1, code=j1601, name=Java就业班]
```

**多对多**

略

### 动态SQL

[参考](http://www.mybatis.org/mybatis-3/zh/dynamic-sql.html)

Mapper.xml

```
<mapper namespace="org.fkit.mapper.EmployeeMapper">

	<select id="selectEmployeeWithId" parameterType="int" resultType="org.fkit.domain.Employee">
  	SELECT * FROM tb_employee where id = #{id}
  </select>
	
  <!-- if -->
  <select id="selectEmployeeByIdLike" 
  	resultType="org.fkit.domain.Employee">
  	SELECT * FROM tb_employee WHERE state = 'ACTIVE'
  	<!-- 可选条件，如果传进来的参数有id属性，则加上id查询条件 -->
  	<if test="id != null ">
  		and id = #{id}
  	</if>
  </select>
  
  <!-- if -->
  <select id="selectEmployeeByLoginLike" 
  	resultType="org.fkit.domain.Employee">
  	SELECT * FROM tb_employee WHERE state = 'ACTIVE'
  	<!-- 两个可选条件，例如登录功能的登录名和密码查询 -->
  	<if test="loginname != null and password != null">
  		and loginname = #{loginname} and password = #{password}
  	</if>
  </select>
  
  <!-- choose（when、otherwise） -->
  <select id="selectEmployeeChoose" 
  	parameterType="hashmap" 
  	resultType="org.fkit.domain.Employee">
  	SELECT * FROM tb_employee WHERE state = 'ACTIVE'
  	<!-- 如果传入了id，就根据id查询，没有传入id就根据loginname和password查询，否则查询sex等于男的数据 -->
  	<choose>
  		<when test="id != null">
  			and id = #{id}
  		</when>
  		<when test="loginname != null and password != null">
  			and loginname = #{loginname} and password = #{password}
  		</when>
  		<otherwise>
  			and sex = '男'
  		</otherwise>
  	</choose>
  </select>
  
  <select id="findEmployeeLike"  
  	resultType="org.fkit.domain.Employee">
  	SELECT * FROM tb_employee WHERE 
  	<if test="state != null ">
  		state = #{state}
  	</if>
  	<if test="id != null ">
  		and id = #{id}
  	</if>
  	<if test="loginname != null and password != null">
  		and loginname = #{loginname} and password = #{password}
  	</if>
  </select>
  
  <!-- where -->
  <select id="selectEmployeeLike" 
  	resultType="org.fkit.domain.Employee">
  	SELECT * FROM tb_employee  
  	<where>
  		<if test="state != null ">
  			state = #{state}
	  	</if>
	  	<if test="id != null ">
	  		and id = #{id}
	  	</if>
	  	<if test="loginname != null and password != null">
	  		and loginname = #{loginname} and password = #{password}
	  	</if>
  	</where>
  </select>
  
  <!-- set -->
  <update id="updateEmployeeIfNecessary" 
  	parameterType="org.fkit.domain.Employee">
	  update tb_employee
	    <set>
	      <if test="loginname != null">loginname=#{loginname},</if>
	      <if test="password != null">password=#{password},</if>
	      <if test="name != null">name=#{name},</if>
	      <if test="sex != null">sex=#{sex},</if>
	      <if test="age != null">age=#{age},</if>
	      <if test="phone != null">phone=#{phone},</if>
	      <if test="sal != null">sal=#{sal},</if>
	      <if test="state != null">state=#{state}</if>
	    </set>
	  where id=#{id}
	</update>
  
  <!-- foreach -->
  <select id="selectEmployeeIn" resultType="org.fkit.domain.Employee">
	  SELECT *
	  FROM tb_employee
	  WHERE ID in
	  <foreach item="item" index="index" collection="list"
	      open="(" separator="," close=")">
	        #{item}
	  </foreach>
  </select>
  
  <!-- bind -->
	<select id="selectEmployeeLikeName"  resultType="org.fkit.domain.Employee">
	  <bind name="pattern" value="'%' + _parameter.getName() + '%'" />
	  	SELECT * FROM tb_employee
	  	WHERE loginname LIKE #{pattern}
	</select>

</mapper>
```

传入参数的方式有两种，一种是通过HashMap，一种是通过JavaBean

```
public void testSelectEmployeeByIdLike(SqlSession session){
	// 获得EmployeeMapper接口的代理对象
	EmployeeMapper em = session.getMapper(EmployeeMapper.class);
	// 创建一个HashMap存储参数
	HashMap<String, Object> params = new HashMap<String, Object>();
	// 设置id属性
	params.put("id", 1);
	// 调用EmployeeMapper接口的selectEmployeeByIdLike方法
	List<Employee> list = em.selectEmployeeByIdLike(params);
	// 查看返回结果
	list.forEach(employee -> System.out.println(employee));
}

public void testUpdateEmployeeIfNecessary(SqlSession session){
	EmployeeMapper em = session.getMapper(EmployeeMapper.class);
	Employee employee = em.selectEmployeeWithId(4);
	// 设置需要修改的属性
	employee.setLoginname("mary");
	employee.setPassword("123");
	employee.setName("玛丽");
	em.updateEmployeeIfNecessary(employee);
}
	
public void testSelectEmployeeIn(SqlSession session){
	EmployeeMapper em = session.getMapper(EmployeeMapper.class);
	// 创建List集合
	List<Integer> ids = new ArrayList<Integer>();
	// 往List集合中添加两个测试数据
	ids.add(1);
	ids.add(2);
	List<Employee> list = em.selectEmployeeIn(ids);
	list.forEach(employee -> System.out.println(employee));
}
	
public void testSelectEmployeeLikeName(SqlSession session){
	EmployeeMapper em = session.getMapper(EmployeeMapper.class);
	Employee employee = new Employee();
	// 设置模糊查询的参数
	employee.setName("o");
	List<Employee> list = em.selectEmployeeLikeName(employee);
	list.forEach(result -> System.out.println(result));
}
```

## MyBatis的事务管理和缓存机制
### MyBatis两种事务管理方式

* 使用JDBC的事务管理机制
* 使用MANAGED的事务管理机制（借用容器WebLogic等实现事务管理）

**配置**

`<environment>`里的`<transactionManager>`

**创建事务工厂TransactionFactory**

**从TransactionFactory获得Transaction对象**

**JdbcTranaction**

**ManagedTransaction**

```
SqlSession session = ssf.openSession(false); //true 为自动提交事务
try {  
    SUser SUser = (SUser) session.selectOne("selectSUser", 1);
    SUser SUser2 = session.selectOne("com.springdemo.usermgr.vo.SUserMapper.selectSUser", 2);
    System.out.println(SUser);  
    System.out.println(SUser2.getPwd()); 
    
    SUserMapper mapper = session.getMapper(SUserMapper.class);
    SUser blog = mapper.getSUser("2");
    System.out.println(blog.getUserName()); 
    SUser user3 = new SUser();  
    user3.setUserName("中文名zhou");  
    user3.setPwd("xxxx");  
    user3.setSignUpTime(new Date());
    System.out.println("插入前主键为："+user3.getId());  
    mapper.insertSUser(user3);//插入操作  
    System.out.println("插入后主键为："+user3.getId());
    session.commit(true);
} catch (Exception e) { 
    session.rollback(true);
    e.printStackTrace();  
} finally {  
    session.close();  
}  

```

### Mybatis的缓存机制
#### 一级缓存
一级缓存是SqlSession级别的缓存，当执行两次相同的sql语句时，第一次执行完毕后会将查询的数据写入缓存，第二次查询会从缓存中获取（可以通过查看日志确认），当SqlSession执行了DML操作并**提交**后，会清空SqlSession中的一级缓存。一级缓存默认开启，不需要任何配置

#### 二级缓存
二级缓存是`mapper`级别的缓存，多个`SqlSession`共享，作用域是`mapper`的同一个`namespace`。不同`SqlSession`两次执行相同的`namespace`下的sql语句，且向sql中传递的参数也相同，则第一次执行完毕会将数据库中查询的数据写到缓存，第二次查询会从缓存中获取。

开启二级缓存

```
//mybatis-config.xml
<settings>
	<setting name="logImpl" value="LOG4J"/>
	<!-- 开启二级缓存 -->
	<setting name="cacheEnabled" value="true"/>
</settings>

//UserMapper.xml
<!-- 开启二级缓存 
   	回收策略为先进先出
   	自动刷新时间60s
   	最多缓存512个引用对象
   	只读
   -->
<cache 
	eviction="LRU"  
	flushInterval="60000" 
	size="512" 
	readOnly="true"/> 
```
属性设置[参考](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html#cache)

使用二级缓存时，与查询结果映射的Java对象必须实现`java.io.Serializable`接口的序列化和反序列化操作，如果存在父类，其成员都需要实现序列化接口。

## MyBatis的注解配置
### 常用Annotation注解
位于`org.apache.ibatis.annotations`包下

* Select - 映射查询的SQL语句
* SelectProvider - Select语句的动态SQL映射。允许指定一个类名和一个方法在执行时返回运行的查询语句。两个属性：type是类的完全限定名，method是该类中的方法名
* Insert - 映射插入的SQL语句
* InsertProvider - Insert语句的动态SQL映射，同SelectProvider
* Update - 映射更新的SQL语句
* UpdateProvider - Update语句的动态SQL映射，同SelectProvider
* Delete - 映射删除的SQL语句
* DeleteProvider - Delete语句的动态SQL映射，同SelectProvider
* Result - 在列和属性之间的单独结果映射。属性包括id、column、property、javaType、jdbcType、type Handler、one、many。id属性是一个布尔值，表示是否被用于主键映射。one是单独的联系，和xml配置的`<association>`相似，many属性和`<collection>`相似
* Results - 多个结果映射列表
* Options - 提供配置选项的附加值，通常在映射语句上作为附加功能配置出现
* One - 复杂类型的单独属性值映射，必须指定select属性，表示已映射的SQL语句的完全限定名
* Many - 复杂类型的集合属性映射，必须制定select属性，表示已映射的SQL语句的完全限定名
* Param - 当映射器方法需要多个参数时，这个注解可以被应用于映射器方法参数来给每个参数取一个名字。否则，多参数将会以它们的顺序位置和SQL语句中的表达式进行映射，这是默认的。使用@Param("id")，SQL中参数应该被命名为#{id}

### Annotation注解的使用
```
public interface UserMapper {
	
	@Insert("INSERT INTO TB_USER(name,sex,age) VALUES(#{name},#{sex},#{age})")
	@Options(useGeneratedKeys=true,keyProperty="id")
//	@SelectKey(before=false,keyProperty="id",resultType=Integer.class,
//	statement="SELECT LAST_INSERT_ID() AS id")
	int saveUser(User user);
	
	@Delete("DELETE FROM TB_USER WHERE id = #{id}")
	int removeUser(@Param("id") Integer id);
	
	@Update("UPDATE TB_USER SET name = #{name},sex = #{sex},age = #{age} WHERE id = #{id}")
	void modifyUser(User user);
	
	@Select("SELECT * FROM TB_USER WHERE id = #{id}")
	@Results({
		@Result(id=true,column="id",property="id"),
		@Result(column="name",property="name"),
		@Result(column="sex",property="sex"),
		@Result(column="age",property="age")
	})
	User selectUserById(@Param("id") Integer id);
	
	@Select("SELECT * FROM TB_USER ")
	List<User> selectAllUser();

}
```

**一对一**

```
public interface PersonMapper {
	
	@Select("SELECT * FROM TB_PERSON WHERE ID = #{id}")
	@Results({
		@Result(id=true,column="id",property="id"),
		@Result(column="name",property="name"),
		@Result(column="sex",property="sex"),
		@Result(column="age",property="age"),
		@Result(column="card_id",property="card",
		one=@One(
				select="org.fkit.mapper.CardMapper.selectCardById",
				fetchType=FetchType.EAGER))
	})
	Person selectPersonById(Integer id);

}
```

**一对多**

```
public interface ClazzMapper {

	// 根据id查询班级信息
	@Select("SELECT * FROM TB_CLAZZ  WHERE ID = #{id}")
	@Results({
		@Result(id=true,column="id",property="id"),
		@Result(column="code",property="code"),
		@Result(column="name",property="name"),
		@Result(column="id",property="students",
		many=@Many(
				select="org.fkit.mapper.StudentMapper.selectByClazzId",
				fetchType=FetchType.LAZY))
	})
	Clazz selectById(Integer id);

		
}
```

**多对多**

```
public interface OrderMapper {

	@Select("SELECT * FROM TB_ORDER WHERE ID = #{id}")
	@Results({
		@Result(id=true,column="id",property="id"),
		@Result(column="code",property="code"),
		@Result(column="total",property="total"),
		@Result(column="user_id",property="user",
			one=@One(select="org.fkit.mapper.UserMapper.selectById",
		fetchType=FetchType.EAGER)),
		@Result(column="id",property="articles",
			many=@Many(select="org.fkit.mapper.ArticleMapper.selectByOrderId",
		fetchType=FetchType.LAZY))
	})
	Order selectById(Integer id);
	
}
```

**动态SQL**

`org.apache.ibatis.jdbc.SQL`工具类，常用方法如下

* T SELECT(String columns)
* T FROM(String table)
* T JOIN(String join)
* T INNER_JOIN(String join)
* T LEFT_OUTER_JOIN(String join)
* T RIGHT_OUTER_JOIN(String join)
* T WHERE(String conditions)
* T OR()
* T AND()
* T GROUP_BY(String columns)
* T HAVING(String conditions)
* T ORDER_BY(String columns)
* T INSERT_INTO(String tableName)
* T VALUES(String columns, String values)
* T DELETE_FROM(String table)
* T UPDATE(String table)
* T SET(String sets)

EmployeeMapper.java

```
public interface EmployeeMapper {
	
	// 动态查询
	@SelectProvider(type=EmployeeDynaSqlProvider.class,method="selectWhitParam")
	List<Employee> selectWhitParam(Map<String, Object> param);
	
	// 动态插入
	@InsertProvider(type=EmployeeDynaSqlProvider.class,method="insertEmployee")
	@Options(useGeneratedKeys = true, keyProperty = "id")  
	int insertEmployee(Employee employee);
	
	// 根据id查询
	@SelectProvider(type=EmployeeDynaSqlProvider.class,method="selectWhitParam")
	Employee selectEmployeeWithId(Map<String, Object> param);
	
	// 动态更新
	@UpdateProvider(type=EmployeeDynaSqlProvider.class,method="updateEmployee")
	void updateEmployee(Employee employee);

	// 动态删除
	@DeleteProvider(type=EmployeeDynaSqlProvider.class,method="deleteEmployee")
	void deleteEmployee(Map<String, Object> param);
	
}
```

EmployeeDynaSqlProvider.java

```
public class EmployeeDynaSqlProvider {
	
	public String selectWhitParam(Map<String, Object> param){
		return new SQL(){
			{
				SELECT("*");
				FROM("tb_employee");
				if(param.get("id") != null){
					WHERE(" id = #{id} ");
				}
				if(param.get("loginname") != null){
					WHERE(" loginname = #{loginname} ");
				}
				if(param.get("password") != null){
					WHERE("password = #{password}");
				}
				if(param.get("name")!= null){
					WHERE("name = #{name}");
				}
				if(param.get("sex")!= null){
					WHERE("sex = #{sex}");
				}
				if(param.get("age")!= null){
					WHERE("age = #{age}");
				}
				if(param.get("phone")!= null){
					WHERE("phone = #{phone}");
				}
				if(param.get("sal")!= null){
					WHERE("sal = #{sal}");
				}
				if(param.get("state")!= null){
					WHERE("state = #{state}");
				}
				
			}
		}.toString();
	}	
	
	public String insertEmployee(Employee employee){
		
		return new SQL(){
			{
				INSERT_INTO("tb_employee");
				if(employee.getLoginname() != null){
					VALUES("loginname", "#{loginname}");
				}
				if(employee.getPassword() != null){
					VALUES("password", "#{password}");
				}
				if(employee.getName()!= null){
					VALUES("name", "#{name}");
				}
				if(employee.getSex()!= null){
					VALUES("sex", "#{sex}");
				}
				if(employee.getAge()!= null){
					VALUES("age", "#{age}");
				}
				if(employee.getPhone()!= null){
					VALUES("phone", "#{phone}");
				}
				if(employee.getSal()!= null){
					VALUES("sal", "#{sal}");
				}
				if(employee.getState()!= null){
					VALUES("state", "#{state}");
				}
			}
		}.toString();
	}
	
	public String updateEmployee(Employee employee){
		
		return new SQL(){
			{
				UPDATE("tb_employee");
				if(employee.getLoginname() != null){
					SET("loginname = #{loginname}");
				}
				if(employee.getPassword() != null){
					SET("password = #{password}");
				}
				if(employee.getName()!= null){
					SET("name = #{name}");
				}
				if(employee.getSex()!= null){
					SET("sex = #{sex}");
				}
				if(employee.getAge()!= null){
					SET("age = #{age}");
				}
				if(employee.getPhone()!= null){
					SET("phone = #{phone}");
				}
				if(employee.getSal()!= null){
					SET("sal = #{sal}");
				}
				if(employee.getState()!= null){
					SET("state = #{state}");
				}
				WHERE(" id = #{id} ");
			}
		}.toString();
	}
	
	public String deleteEmployee(Map<String, Object> param){
		
		return new SQL(){
			{
				DELETE_FROM("tb_employee");
				if(param.get("id") != null){
					WHERE(" id = #{id} ");
				}
				if(param.get("loginname") != null){
					WHERE(" loginname = #{loginname} ");
				}
				if(param.get("password") != null){
					WHERE("password = #{password}");
				}
				if(param.get("name")!= null){
					WHERE("name = #{name}");
				}
				if(param.get("sex")!= null){
					WHERE("sex = #{sex}");
				}
				if(param.get("age")!= null){
					WHERE("age = #{age}");
				}
				if(param.get("phone")!= null){
					WHERE("phone = #{phone}");
				}
				if(param.get("sal")!= null){
					WHERE("sal = #{sal}");
				}
				if(param.get("state")!= null){
					WHERE("state = #{state}");
				}
			}
		}.toString();
	}
}
```

测试代码

```
public class DynamicSQLTest {

	public static void main(String[] args) throws Exception {

		// 创建Session实例
		SqlSession session = FKSqlSessionFactory.getSqlSession();
		
		DynamicSQLTest t = new DynamicSQLTest();
		// 获取EmployeeMapper对象
		EmployeeMapper em = session.getMapper(EmployeeMapper.class);
		
//		t.testSelectWhitParam(em);
		
//		t.testInsertEmployee(em);
		
//		t.testUpdateEmployee(em);
		
		t.testDeleteEmployee(em);
		
		// 提交事务
		session.commit();
		// 关闭Session
		session.close();
	}
	
	// 根据动态参数查询员工数据
	public void testSelectWhitParam(EmployeeMapper em){
		// 使用Map装载参数
		Map<String, Object> param = new HashMap<String, Object>();
		param.put("loginname", "jack");
		param.put("password", "123456");
		// 调用selectWhitParam方法
		List<Employee> list = em.selectWhitParam(param);
		// 查看返回结果
		System.out.println(list);
	}
	
	// 根据设置的属性动态插入数据
	public void testInsertEmployee(EmployeeMapper em){
		
		Employee e = new Employee();
		e.setLoginname("mary");
		e.setPassword("123456");
		e.setName("玛丽");
		e.setSex("女");
		e.setAge(20);
		e.setPhone("13902019999");
		e.setSal(9800.99);
		// 注意：没有设置state属性，则insert语句中不会包含state列
		// e.setState("ACTIVE");
		em.insertEmployee(e);
		
		System.out.println("插入成功，返回id：" + e.getId());

	}
	
	// 根据设置的属性动态更新数据
	public void testUpdateEmployee(EmployeeMapper em){
		// 使用Map装载参数
		Map<String, Object> param = new HashMap<String, Object>();
		param.put("id", 5);
		// 查询id为1的员工
		Employee e = em.selectEmployeeWithId(param);
		// 修改员工对象的三个属性
		e.setLoginname("update");
		e.setPassword("fkjava");
		e.setName("测试");
		// 动态更新
		em.updateEmployee(e);
	}
	
	// 根据设置的属性动态删除数据
	public void testDeleteEmployee(EmployeeMapper em){
		// 使用Map装载参数
		Map<String, Object> param = new HashMap<String, Object>();
		param.put("loginname", "jack");
		param.put("password", "123456");
		// 动态删除
		em.deleteEmployee(param);

	}
}
```

## Spring4整合MyBatis3
**配置文件**

```
// src/db.properties
dataSource.driverClass=com.mysql.jdbc.Driver
dataSource.jdbcUrl=jdbc:mysql://127.0.0.1:3306/mybatis
dataSource.user=root
dataSource.password=root
dataSource.maxPoolSize=20
dataSource.maxIdleTime = 1000
dataSource.minPoolSize=6
dataSource.initialPoolSize=5
```

```
// WEB-INF/applicationContext.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" 
	xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
			            http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
			            http://www.springframework.org/schema/context
			            http://www.springframework.org/schema/context/spring-context-4.2.xsd
			            http://www.springframework.org/schema/mvc
			            http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd
			            http://www.springframework.org/schema/tx
			            http://www.springframework.org/schema/tx/spring-tx-4.1.xsd
			            http://mybatis.org/schema/mybatis-spring 
			            http://mybatis.org/schema/mybatis-spring.xsd ">
			      
	 <!-- mybatis:scan会将org.fkit.mapper包里的所有接口当作mapper配置，之后可以自动引入mapper类-->  
    <mybatis:scan base-package="org.fkit.mapper"/>   
       
	 <!-- 扫描org.fkit包下面的java文件，有Spring的相关注解的类，则把这些类注册为Spring的bean -->
    <context:component-scan base-package="org.fkit"/>
    
	<!-- 使用PropertyOverrideConfigurer后处理器加载数据源参数 -->
	<context:property-override location="classpath:db.properties"/>

	<!-- 配置c3p0数据源 -->
	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"/>
	
	<!-- 配置SqlSessionFactory，org.mybatis.spring.SqlSessionFactoryBean是Mybatis社区开发用于整合Spring的bean -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean"
	    p:dataSource-ref="dataSource"/>
	
	<!-- JDBC事务管理器 -->
	<bean id="transactionManager" 
	class="org.springframework.jdbc.datasource.DataSourceTransactionManager"
		 p:dataSource-ref="dataSource"/>
	
	<!-- 启用支持annotation注解方式事务管理 -->
	<tx:annotation-driven transaction-manager="transactionManager"/>
	
</beans>
```

```
// WEB-INF/springmvc-config.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd     
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-4.2.xsd">
        
    <!-- 自动扫描该包，SpringMVC会将包下用了@controller注解的类注册为Spring的controller -->
    <context:component-scan base-package="org.fkit.controller"/>
    <!-- 设置默认配置方案 -->
    <mvc:annotation-driven/>
    <!-- 使用默认的Servlet来响应静态文件 -->
    <mvc:default-servlet-handler/>
    <!-- 视图解析器  -->
     <bean id="viewResolver"
          class="org.springframework.web.servlet.view.InternalResourceViewResolver"> 
        <!-- 前缀 -->
        <property name="prefix">
            <value>/WEB-INF/content/</value>
        </property>
        <!-- 后缀 -->
        <property name="suffix">
            <value>.jsp</value>
        </property>
    </bean>
    
</beans>
```

```
// WEB-INF/web.xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns="http://xmlns.jcp.org/xml/ns/javaee" 
	xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
	http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" 
	id="WebApp_ID" version="3.1">
	
	<!-- 配置spring核心监听器，默认会以 /WEB-INF/applicationContext.xml作为配置文件 -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	<!-- contextConfigLocation参数用来指定Spring的配置文件 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/applicationContext*.xml</param-value>
	</context-param>
	
	<!-- 定义Spring MVC的前端控制器 -->
  <servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>
        org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>/WEB-INF/springmvc-config.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  
  <!-- 让Spring MVC的前端控制器拦截所有请求 -->
  <servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
  
  <!-- 编码过滤器 -->
  <filter>
		<filter-name>characterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
 </filter>
	<filter-mapping>
		<filter-name>characterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
	
</web-app>
```

POJO、持久层略

服务层示例

```
public interface BookService {
	
	/**
	 * 查找所有图书
	 * @return Book对象集合
	 * */
	List<Book> getAll();

}


@Service("bookService")
public class BookServiceImpl implements BookService {
	
	/**
	 * 自动注入BookMapper
	 * */
	@Autowired
	private BookMapper bookMapper;

	/**
	 * BookService接口getAll方法实现
	 * @see { BookService }
	 * */
	@Override
	public List<Book> getAll() {
		
		return bookMapper.findAll();
	}

}
```

控制层、JSP略


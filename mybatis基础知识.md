## 创建mybatis数据库，创建user表，在里面添加数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/e57dd32b55944e35bbe6d76a1d12df80.png)

## 创建空项目，创建maven模块
![在这里插入图片描述](https://img-blog.csdnimg.cn/758bfc94eaef4a318617f066c791ae4b.png)

## 导入相关的依赖
在pom.xml文件中导入mysql，junit测试，mybatis依赖。
```
<!--导入mysql依赖-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.30</version>
        </dependency>
```
```
<!--导入mybatis依赖-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.10</version>
        </dependency>
```
```
<!--导入junit测试依赖-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
```

## 编写mybatis核心配置文件
resource目录主要是存放一些配置文件，在resource目录下创建mybatis-config.xml配置文件,mybatis-config.xml配置文件中主要是存放数据库的连接信息和加载mapper映射文件，映射文件中存放了一些sql语句。
	
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <!--数据库的连接信息-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false&amp; serverTimezone=Hongkong&amp; characterEncoding=utf-8&amp; autoReconnect=true"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>

    <!--加载mapper映射文件，该文件中有相关的sql语句-->
    <mappers>
        <mapper resource="UserMapper.xml"/>
    </mappers>
</configuration>
```
## 编写UserMapper.xml配置文件
在resource目录下创建一个UserMapper.xml配置文件。

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace叫做命名空间-->
<!--
id是sql语句的唯一标识，
resultType叫做返回的结果类型-->
<mapper namespace="test">
    <select id="selectAll" resultType="com.acwer.pojo.User">
        select *from user;
    </select>
</mapper>

```
同时需要更改mybatis-config.xml文件中mapper标签中的代码，让其加载mapper映射文件
```
<!--加载mapper映射文件，该文件中有相关的sql语句-->
    <mappers>
        <mapper resource="UserMapper.xml"/>
    </mappers>
```

## 编写pojo类
编写结果集返回的对象类型，即为com.acwer.pojo.User类。
```
package com.acwer.pojo;

public class User {
    //变量名要和数据库中的字段名一致
    private int id;
    private String name;
    private String pwd;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPwd() {
        return pwd;
    }

    public void setPwd(String pwd) {
        this.pwd = pwd;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", pwd='" + pwd + '\'' +
                '}';
    }
}
```

## 编写一个测试类
编写一个测试类，即为mybatisDemo类。
```
package com.acwer;

import com.acwer.pojo.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

/**
 * Mybatis 快速入门代码
 */
public class mybatisDemo {

    public static void main(String[] args) throws IOException {

        //1. 加载mybatis的核心配置文件，获取 SqlSessionFactory实例化对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象，用它来执行sql
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //3. 执行sql
        //sqlSession对象可以执行sql语句,
        // 而selectList()括号内的参数test.selectAll表示的是可以追踪到命名为test的
        //命名空间(namespace),又追踪到id为selectAll的标识。
        List<User> users = sqlSession.selectList("test.selectAll");
        for(User user:users){
            System.out.println(user.getId()+" "+user.getName()+" "+user.getPwd());
        }
        //4. 释放资源
        sqlSession.close();

    }
}

```

## 代理mapper
上面的sqlSession.selectList("test.selectAll"); 方法中的参数test.selectAll其实还是硬编码，我们需要通过不同的文件来对其进行调整，这就是mapper代理存在的价值。

1.在java下创建一个com.acwer. mapper.UserMapper接口
```
package com.acwer.mapper;

import com.acwer.pojo.User;

import java.util.List;

public interface UserMapper {
    /*
    *   mybatis面向接口编程的两个一致：
    *   1. 映射文件的namespace要和mapper接口的类名保持一致。
    *   2. 映射文件的sql语句的id要和mapper接口的方法名保持一致。
    *   这两句话可以这样理解，例如userMapper.selectAll(),userMapper是
    *   UserMapper的实例化对象，到调用该方法时，我们通过UserMapper(类名)定位可以寻找到UserMapper
    *   的配置文件的mapper标签，通过selectAll()(方法名)可以找到相应的id，进一步缩小范围，即select
    *   标签中的sql语句。
    * */
    List<User> selectAll();
}

```

2. 在resources文件夹下创建一个mapper文件夹，在mapper目录下创建一个UserMapper.xml目录。
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace叫做命名空间-->
<!--
id是sql语句的唯一标识，
resultType叫做返回的结果类型-->
<mapper namespace="com.acwer.mapper.UserMapper">
    <select id="selectAll" resultType="com.acwer.pojo.User">
        select *from user;
    </select>
</mapper>

```
3. 在mybatis-config核心配置文件中，修改mapper标签的resource的内容，改成相应的映射文件的地址，即为mapper/UserMapper.xml。
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <!--数据库的连接信息-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false&amp; serverTimezone=Hongkong&amp; characterEncoding=utf-8&amp; autoReconnect=true"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>

    <!--加载mapper映射文件，该文件中有相关的sql语句-->
    <mappers>
        <mapper resource="mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

## 测试结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/d4743021be8a45d79f9e941df2da95a9.png)

## junit测试
在test的目录下，在java目录下创建一个com.acwer.mybatis.test.mybatisTest类。
```
package com.acwer.mybatis.test;
import com.acwer.mapper.UserMapper;
import com.acwer.pojo.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.*;
import java.io.*;
import java.util.List;


public class mybatisTest {
    public static void main(String[] args) throws IOException {
        //加载核心配置文件
        InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");

        //获取SqlSessionFactoryBuilder对象
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();

        //用SqlSessionFactoryBuilder对象来获取SqlSessionFactory对象
        SqlSessionFactory sqlSessionFactory =sqlSessionFactoryBuilder.build(inputStream);

        //获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);//true表示自动提交事务。

        //获取mapper接口对象
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

        //执行sql语句，测试功能
        int t = userMapper.insert();
        System.out.println(t);
        List<User> users = userMapper.selectAll();
        for(User user:users){
            System.out.println(user.getId()+" "+user.getName()+" "+user.getPwd());
        }  
    }
}
```

## 删除操作代码
```
1. 在UserMapper接口中添加int delete()方法，在UserMapper.xml映射文件中
添加delete的相关信息，然后在测试代码中添加delete()的相关测试代码。
```

## 更新操作代码
```
和上面的删除，插入操作基本类似。
```

## 查询操作代码
```
和上面的删除，插入操作基本类似。
```



**注意**
测试结果时，可能会出现**java: 错误: 不支持发行版本**错误信息，此时我们需要调高Java编译器的字节码版本，调到12以上就可以了。

## package的使用
在一般的情况下，我们肯定要导入多个映射文件，所以我们要使用package,然后规定的包下所有的映射文件进行加载。

**修改mybatis-config.xml配置文件**
```
<!--加载mapper映射文件，该文件中有相关的sql语句-->
    <mappers>
        <!--<mapper resource="mapper/UserMapper.xml"/>-->
        <!--映射文件目录和映射接口目录需要保持一样的，
            即映射文件目录:com/acwer/mapper/
            映射接口目录:com/acwer/mapper/
        -->
        <package name="com.acwer.mapper"/>
    </mappers>
```
**如果目录没要保持一致，可能会报错！**

## properties的使用
在mybatis-config.xml文件中dataSource标签中还是存在硬编码，我们需要通过一个properties文件来使得更加地灵活。我们可以将driver，url，username
，password的信息用properties文件进行储存。

1. mybatis-config.xml文件
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--导入jdbc.properties配置文件-->
    <properties resource="jdbc.properties"/>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <!--数据库的连接信息-->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>

    <!--加载mapper映射文件，该文件中有相关的sql语句-->
    <mappers>
        <!--<mapper resource="mapper/UserMapper.xml"/>-->
        <!--映射文件目录和映射接口目录需要保持一样的，
            即映射文件目录:com/acwer/mapper/
            映射接口目录:com/acwer/mapper/
        -->
        <package name="com.acwer.mapper"/>
    </mappers>
</configuration>
```


2. jdbc.properties文件
在resource下创建一个jdbc.properties文件。
```
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis?useSSL=false
jdbc.username=root
jdbc.password=123456
```

## mybatis获取参数值的几种方式
**1. 当方法内只有一个参数时**
* UserMapper接口
```
User selectByName(String name);
```
* UserMapper.xml
```
<!--
        单个参数时。
        ${}表示的是占位符,会自动加上单引号。
        #{}表示的是字符串拼接，不会自动加上单引号，我们需要手动地加上单引号。
-->
    <select id="selectByName" resultType="com.acwer.pojo.User">
        select * from user where name=#{name};
    </select>
```

**2. 当方法内有多个参数时**
* UserMapper接口
```
User checkLogin(int id,String name);
```
* UserMapper.xml文件
```
<!--
        多个参数时。
        select * from user where id = #{id} and name = #{name};这样写会报错
        当方法内有多个参数时，这些参数会放到[arg0,arg1,param1,param2]这个集合中，
        ->arg0,arg1为键，他们对应的参数为值。
        ->param1,param2为键，他们对应的参数为值。
        #{}表示的意思是通过键来访问所对应的参数值。
        我们只需这样写:
        select * from user where id = #{arg0} and name = #{arg1};或者
        select * from user where id = #{param1} and name = #{param2};
-->
    <select id="checkLogin" resultType="com.acwer.pojo.User">
        select * from user where id = #{param1} and name = #{param2};
    </select>
```


**3. map存储**
* UserMapper接口
```
<!--
        mapper接口方法的参数有多个时，可以手动将这些参数放到一个map集合中进行存储。
        #{} 通过键可以访问值
-->
    <select id="checkLoginByMap" resultType="com.acwer.pojo.User">
        select * from user where name=#{name} and pwd=#{pwd};
    </select>
```

* UserMapper.xml文件
```
<!--
        mapper接口方法的参数有多个时，可以手动将这些参数放到一个map集合中进行存储。
        #{} 通过键可以访问值
-->
    <select id="checkLoginByMap" resultType="com.acwer.pojo.User">
        select * from user where name=#{name} and pwd=#{pwd};
    </select>
```

**4. param注解(重点)**
* UserMapper接口
```
//Param("name")表示的是键，String name表示的是值。
User checkLoginByParam(@Param("name") String name,@Param("pwd") String password);
```
* UserMappe.xml配置文件
```
<select id="checkLoginByParam" resultType="com.acwer.pojo.User">
        select * from user where name=#{name} and pwd=#{pwd};
</select>
```

## 模糊查询
* UserMapper接口
```
   List<User> selectByLike(@Param("name") String name);
```
* UserMapper.xml
```
<!--这里应该使用${}，如果使用#{}可能识别不出-->
<mapper namespace="com.acwer.mapper.UserMapper">
    <select id="selectByLike" resultType="com.acwer.pojo.User">
        select *from user where name like '%${name}%';
    </select>
</mapper>
```

## 别名配置优化
实体类的地址名字太长，我们可以通过别名将名字进行简化。typeAliases标签需要放到configuration标签里第二个位置，第一个位置是properties标签。

*** mybatis-config.xml配置文件**
```
<!--给实体类取别名-->
    <typeAliases>
        <typeAlias type="com.acwer.pojo.User" alias="User"/>
    </typeAliases>
```
* **UserMapper.xml配置文件**
```
<mapper namespace="com.acwer.mapper.UserMapper">
    <select id="selectByLike" resultType="User">
        select *from user where name like '%${name}%';
    </select>
</mapper>
```

## 字段名和属性名不一样如何解决？
1. 给字段名取和属性名一样的别名。
```
<mapper namespace="com.acwer.mapper.UserMapper">
    <select id="select" resultType="User">
        select id,name, pwd as password from user;
    </select>
</mapper>
```


2. 通过resultMap来解决
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">


<mapper namespace="com.acwer.mapper.UserMapper">
<!--
		创建一对resultMap标签，让select的resultMap和resultMap
		的id产生映射关系，property表示的是实体类的属性名，col
		umn表示的是数据库的字段名。
-->
    <resultMap id="result1" type="com.acwer.pojo.User">
    	<!--id标签表示是主键-->
        <id property="id" column="id"></id>
        <!--result表示的是普通字段名-->
        <result property="name" column="name"></result>
        <result property="password" column="pwd"></result>
    </resultMap>
    <select id="select" resultMap="result1">
        select *from user;
    </select>
</mapper>
```

## 动态mysql
Mybatis框架的动态SQL技术是一种根据特定条件动态拼装SQL语句的功能，它存在的意义是为了解决拼接SQL语句字符串时的痛点问题。
**if标签可通过test属性的表达式进行判断，若表达式的结果为true，则标签中的内容会执行；反之标签中的内容不会执行。**
* if语句
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">


<mapper namespace="com.acwer.mapper.UserMapper">
    <resultMap id="result1" type="com.acwer.pojo.User">
        <id property="id" column="id"></id>
        <result property="name" column="name"></result>
        <result property="password" column="pwd"></result>
    </resultMap>
    <!--resultMap相当于起到了映射的作用-->
    <select id="select" resultMap="result1">
        select * from user where 1=1
        <!--
				name!=null 等号左边name是属性变量
        		name=#{name} 等号左边的是字段名，右边的是属性值。
        -->
        <if test="name!=null and name!=''">
            and name = #{name}
        </if>
        <if test="password!=null and password!=''">
            and pwd = #{password}
        </if>
    </select>
</mapper>
```
* **choose when otherwise**
choose when otherwise 相当于if else-if else
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.acwer.mapper.UserMapper">
    <select id="select" resultType="User">
        select * from user
        <where>
            <choose>
                <when test="id!=null and id!=''">
                    id = #{id}
                </when>
                <when test="name!=null and name!=''">
                    and name = #{name}
                </when>
                <when test="password!=null and password!=''">
                     and pwd = #{password}
                </when>
                <otherwise>
                     and id = 5
                </otherwise>
            </choose>
        </where>
    </select>
</mapper>
```
## 插入操作
```
<insert id="insert">
        insert into user values (#{id},#{name},#{password});
    </insert>
```

## 更新操作
```
<update id="update">
        update user set id=#{id},pwd=#{password} where name='kris';
    </update>
```

## 单个删除
```
<delete id="deleteById">
        delete from user where id=#{id};
    </delete>
```

## 批量删除
* UserMapper接口
```
public interface UserMapper {
    //a[]的键是a
    int delete(@Param("a") int a[]);
}
```
* UserMapper.xml配置文件
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.acwer.mapper.UserMapper">
    <delete id="delete">
        delete from user where id
        in(
        <!--a表示的是数组，t表示的是数组中的每一个元素，逗号表示分隔-->
            <foreach collection="a" item="t" separator=",">
                #{t}
            </foreach>
            )
    </delete>
</mapper>
```

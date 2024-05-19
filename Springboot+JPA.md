# Springboot+JPA

###### --0513--SXJ练习

###### 项目：demo

1. JPA—Java persistence API—java持久层API，是描述对象-关系表的映射关系，并将运行时期的实体对象映射到数据库中

JPA是一个持久层的ORM框架，对jdbc的封装，使用jpa就可以实现操作实体对象，实现对数据库表的CRUD

ORM关系映射：

关系型数据库---对应java

数据表----对应java中的实体类

记录数----对应java中的对象

Field------对应java类中的属性

Java程序员面向对象的角度操作对象，由于我们的表以及表当中的属性已经和关系型数据库中的表和字段进行了一一映射，因此操作对象就可以操作表中的记录

2. 创建springboot项目，引入相关依赖

   ![](D:\godblessing\forwork\操作记录\figure\springboot+jpa-1.png)

```
自动引入了jpa相关的依赖：
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
自动引入了mysql相关的依赖：
<dependency>
	<groupId>com.mysql</groupId>
	<artifactId>mysql-connector-j</artifactId>
	<scope>runtime</scope>
</dependency>
```

3. application.properties配置

```
spring.application.name=demo

#连接数据库的四大参数
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
#spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/jpa?serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=123456

#jpa相关配置
#开发阶段可以设置为ture，开启了逆向工程（实际运营中设为false
#逆向工程：存在数据库的表，然后数据库表可以生成实体类
#正向工程：先存在实体类，然后根据实体类生成底层的表
spring.jpa.generate-ddl=true
#create:每次运行都会将原来的数据表删除，再新建表
#create-drop：每次都创建一个数据表，使用完后删除表
#none:功能不生效
#update：如果表结构和实体类没有一一映射（实体类发生了改变，数据表会更新
#如果数据库中有数据表，就用原来的表；没有数据表，就会创建一个数据表；常用
#validate:实体类和数据表进行校验，如果属性或者个数不一致，就会抛出异常
spring.jpa.hibernate.ddl-auto=update
#操作实体对象的时候，true会生成sql语句；false不生成sql语句
spring.jpa.show-sql=true
#制定了数据库的类型：
spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect
```

4. 创建一个domain包，新建类Pet

```java
package com.example.demo.domain;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
//Pet实体类，对应底层数据库表，一开始mysql数据库中没有t_pet表
//可以使用jpa的正向工程，生成底层对应的关系表
//@Entity注解：表明当前是实体类，和底层名为t_pet的关系表进行映射
@Entity(name="t_pet")
public class Pet {
	@Id //表明是个主键字段（唯一值）
	//指定当前主键的生成策略：自增
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private int id;
	@Column //标识pname是个普通列，其他属性不设置就是默认值
	private String pname;
	@Column 
	private String color;
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
}
```

5. 运行test文件夹下的测试类（加一句输出“创建成功”）

![](D:\godblessing\forwork\操作记录\figure\springboot+jpa-2.png)

日志已经输出成功，且有一句

```
Hibernate: create table t_pet (id integer not null auto_increment, color varchar(255), pname varchar(255), primary key (id)) engine=InnoDB
```

6.建一个dao包，建立接口PetDao，用于执行增删改查

底层用了jpaRepository里面的方法

```
package com.example.demo.dao;
import org.springframework.data.jpa.repository.JpaRepository;
import com.example.demo.domain.Pet;
//PetDao接口，实现pet实体类的CRUD操作
//JpaRepository<T, ID>是jpa提供的接口，接口当中定义了实体的基本操作
//T 指定具体操作的实现类--Pet实体类
//ID 指定主键字段的类型（id注解对应的属性） --Integer
public interface PetDao extends JpaRepository<Pet, Integer>{
}
```

7. 对增删改查进行单元测试

```
/*添加宠物信息*/
	@Autowired
	PetDao petDao;
	//springboot在启动的时候，底层使用动态代理的方式获得接口的实现类
```

上面创建了一个接口的实现类，前面加上注解@Autowired（注入petdao这个bean，然后就可以使用petDao调用底层增删改查的方法

```
@Test
	void addpets() {
		System.out.println("添加pet");
		Pet pet=new Pet();
		//id自增，不需要手动设置
		pet.setPname("猫咪");
		pet.setColor("白色");
		
		petDao.save(pet);//添加操作
		/*save:
		 *如果没指定id，则自增添加
		 *如果指定了id，先去查询有没有这个id的记录
		 *有则更新，没有则添加
		 **/
	}
```

```
//查询--根据id查询
	@Test
	void findpets() {
		//findById是jpa接口中该定义的，查询的内容封装到了Optional对象中
		//再根据get()方法得到了pet对象
		Optional<Pet>optional =petDao.findById(1);
		Pet pet =optional.get();
		System.out.println(pet.getId()+""+pet.getPname()+""+pet.getColor());
	}
```

```
	//查询--所有
	@Test
	void findAllpets() {
		//List<T> findAll();返回一个list
		List<Pet> pets=petDao.findAll();
		for(Pet pet:pets) {
			System.out.println(pet.getId()+""+pet.getPname()+""+pet.getColor());
		}
```

```
//删除
	@Test
	void delePet() {
		/*delete(T)：传递一个对象，删除这个对象
		 *deleteById:按传入的id删除对象
		 */
//		Pet pet=new Pet();
//		pet.setId(1);
//		petDao.delete(pet);
		
		petDao.deleteById(2);
	}
```


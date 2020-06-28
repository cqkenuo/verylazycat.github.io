---
title: veu&spring项目开发
tags: JAVA
---

[toc]

# 参考

- [mybati-plus](https://mp.baomidou.com/guide/#%E7%89%B9%E6%80%A7)

# 数据库配置

```mysql
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `name` varchar(45) NOT NULL,
  `age` int NOT NULL,
  `email` varchar(45) NOT NULL,
  `create_time` datetime DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `id_UNIQUE` (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

> 一般需要在数据库添加`Create_time`和`update_time`字段,用于记录创建时间和修改时间,可以在操作数据库代码中添加相应代码去实现,不过可以在实体类中加上@TableField注解,再写一个handler继承MetaObjectHandler,实现`InsertFill`和`updateFill`方法,从而简化代码

```mysql
DROP TABLE IF EXISTS `product`;
CREATE TABLE `product` (
  `id` bigint NOT NULL,
  `name` varchar(30) DEFAULT NULL,
  `price` int DEFAULT '0',
  `version` int DEFAULT '0' COMMENT '乐观锁',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

> version 字段 在后续解决同时操作数据库发生冲突,需在其实体类对于的数据上添加@version注解

# application.properties配置

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis_plus?serverTimezone=GMT%2B8
spring.datasource.username=name
spring.datasource.password=pass
```

# 实体类

- 创建entity package
- 创建User.class

```java
package com.lazycat.mybatis_plus.entity;

import com.baomidou.mybatisplus.annotation.FieldFill;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import lombok.Data;

import java.util.Date;

@Data
public class User {
    @TableId(type =  IdType.AUTO)
    private  long id;
    private  String name;
    private  Integer age;
    private  String email;

    @TableField(fill = FieldFill.INSERT)
    private Date create_time;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private  Date update_time;
}
```

>- @Data 注解:来源与`lombok`,需要配置插件,主要功能是简化,自动生成getter和setter
>- @TableId:主键注解,具体type参考官网
>- @TableField:字段注解(非主键),后续创建类去继承MetaObjectHandler,实现InsertFill和updateFill等主要方法

- Product

```java
package com.lazycat.mybatis_plus.entity;

import com.baomidou.mybatisplus.annotation.Version;
import lombok.Data;

@Data
public class Product {
    private  Long id;
    private  String name;
    private  Integer price;

    @Version
    private  Integer version;
}
```

>@Version:乐观锁注解,为了在并发使用数据库时不产生冲突,具体参考测试数据库

# Mapper

- 创建mapper package
- 在mapper下创建UserMapper`接口`

```java
package com.lazycat.mybatis_plus.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.lazycat.mybatis_plus.entity.User;
import org.springframework.stereotype.Repository;

@Repository
public interface UserMapper extends BaseMapper<User> {
}
```

> 主要是要继承`BaseMapper`,泛型根据自己定义的实体类

- ProductMapper

```java
package com.lazycat.mybatis_plus.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.lazycat.mybatis_plus.entity.Product;
import org.springframework.stereotype.Repository;

@Repository
public interface ProductMapper extends BaseMapper<Product> {
}
```

# Handler

- 创建Handler package
- 创建MyObjectHandler

```java
package com.lazycat.mybatis_plus.handler;

import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import org.apache.ibatis.reflection.MetaObject;
import org.springframework.stereotype.Component;

import java.util.Date;

@Component
public class MyObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        this.setFieldValByName("create_time",new Date(),metaObject);
        this.setFieldValByName("update_time",new Date(),metaObject);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.setFieldValByName("update_time",new Date(),metaObject);
    }
}
```

> 主要是继承MetaObjectHandler,实现insertFill,updateFill等方法

# 测试数据库

```
package com.lazycat.mybatis_plus;

import com.lazycat.mybatis_plus.entity.Product;
import com.lazycat.mybatis_plus.entity.User;
import com.lazycat.mybatis_plus.mapper.ProductMapper;
import com.lazycat.mybatis_plus.mapper.UserMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.Date;

@SpringBootTest
public class CRUDtest {
    @Autowired
    private UserMapper userMapper;
    @Autowired
    private ProductMapper productMapper;
    @Test
    public void test(){
        User user = new User();
        user.setAge(19);
        user.setEmail("test@qq.com");
//        user.setCreate_time(new Date());
//        user.setCreate_time(new Date());
//        user.setId(5);
        user.setName("hhh");
        //影响的行数
        int result = userMapper.insert(user);
        System.out.println("影响的行数"+result);
        System.out.println("id="+user.getId());
    }
    @Test
    public void updateById(){
        User user = new User();
        user.setId(1);
        user.setName("test1");
        user.setAge(20);
        int result = userMapper.updateById(user);
        System.out.println("影响行数:"+result);
    }
    @Test
    //并发测试
    public void ConcurrentUpdatetest(){
        Product product1 = productMapper.selectById(1);
        Product product2 = productMapper.selectById(1);
        product1.setPrice(product1.getPrice()+50);
        productMapper.updateById(product1);
        product2.setPrice(product2.getPrice()-30);
        int result = productMapper.updateById(product2);
        if (result == 0){
            System.out.println("pro2更新失败");
            System.out.println("pro2重新更新");
            product2 = productMapper.selectById(1);
            product2.setPrice(product2.getPrice() - 30);
            productMapper.updateById(product2);
        }
        Product product3 = productMapper.selectById(1);
        System.out.println(product3.getPrice()
        );
    }
}
```

# 配置package

- 创建config配置package
- 创建MyBatisPlusConfig

```java
package com.lazycat.mybatis_plus.config;

import com.baomidou.mybatisplus.extension.plugins.OptimisticLockerInterceptor;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@EnableTransactionManagement
@Configuration
@MapperScan("com.lazycat.mybatis_plus.mapper")
public class MyBatisPlusConfig {
    /*
        乐观锁
    * */
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor() {
        return new OptimisticLockerInterceptor();
    }
}
```
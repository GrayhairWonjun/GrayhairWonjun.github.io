---
layout: post
title:  "Spring Data JPA"
comments: true
archive: true
date:   2017-02-13 13:44:58 -0600
categories: Java Spring
tags: java spring data jpa hibernate
---

### Spring Data JPA

Spring Data JPA Repository abstraction의 목적은 데이터 접근을 위해 작성되는 반복적인 코드 조각(boilerplate code)를 줄이기 위함.
>The goal of Spring Data repository abstraction is to significantly reduce the amount of boilerplate code required to implement data access layers for various persistence stores.

`Spring Data JPA`는 Repository abstraction을 제공하는 `Spring Data Commons`위에 놓여있다. 그렇기에 Spring Repository interface를 직접 구현하지 않고 Spring Data JPA를 사용할 수 있다. 

첫째, Spring Data Commons 는 다음과 같은 인터페이스를 제공한다.
* `Repository<T, ID extends Serializable>`
* `CrudRepository<T, ID extends Serializable>`
  * Entity에 대한 CRUD operator를 제공
* `PagingAndSortingRepository<T, ID extends Serializable>`
  * Database로 부터 검색된 Entity들을 정렬하고 페이지매김하는 메소드를 선언
* `QueryDslPredicateExecutor<T>`
  * Repository interface가 아님
  * QueryDsl Predicate 객체를 사용하여 Database로 부터 Entity를 검색하는 메소드를 선언

둘째, Spring Data JPA 는 다음과 같은 인터페이스를 제공한다
* `JpaRepository<T, ID extends Serializable>`
  * JPA specific repository interface.
* `JpaSpecificationExecutor<T>`
  * Repository interface 가 아님
  * Specification<T> object를 이용하여 Entity를 조회할 때 사용 

### What components do we need?

* JDBC Driver
  * 특정 Database 연결 구현을 한 드라이버
* datasource
  * connection을 제공해 주는 datasource. (HikariCP datasource 속도가 빠름)
* JPA Driver
  * Java Persistence API를 구현한 구현체. Hibernate를 주로 많이 사용

#### dependency 정의 (pom.xml)
{% highlight xml %}
    <!-- Database (H2) -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <version>1.4.185</version>
    </dependency>
             
    <!-- DataSource (HikariCP) -->
    <dependency>
        <groupId>com.zaxxer</groupId>
        <artifactId>HikariCP</artifactId>
        <version>2.2.5</version>
    </dependency>
     
    <!-- JPA Provider (Hibernate) -->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-entitymanager</artifactId>
        <version>4.3.8.Final</version>
    </dependency>
     
    <!-- Spring Data JPA -->
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-jpa</artifactId>
        <version>1.7.2.RELEASE</version>
    </dependency>
{% endhighlight %}

### Configuration

1. Create the properties file that contains the properties used by our application context configuration class.
2. Configure the datasource bean.
3. Configure the entity manager factory bean.
4. Configure the transaction manager bean.
5. Enable annotation-driven transaction management.
6. Configure Spring Data JPA.


1. Configure the database connection of our application. We need to configure the name of the JDBC driver class, the JDBC url, the username of the database user, and the password of the database user.
2. Configure Hibernate by following these steps:
  * Configure the used database dialect.
  * Ensure that Hibernate creates the database when our application is started and drops it when our application is closed.
  * Configure the naming strategy that is used when Hibernate creates new database objects and schema elements.
  * Configure the Hibernate to NOT write the invoked SQL statements to the console.
  * Ensure that if Hibernate writes the SQL statements to the console, it will use prettyprint.

`java/main/resources/application.properties` 파일에 아래와 같은 형식으로 db connection string과 hibernate 설정해 준다.
{% highlight properties %}
#Database Configuration
db.driver=org.h2.Driver
db.url=jdbc:h2:mem:datajpa
db.username=sa
db.password=
 
#Hibernate Configuration
hibernate.dialect=org.hibernate.dialect.H2Dialect
hibernate.hbm2ddl.auto=create-drop
hibernate.ejb.naming_strategy=org.hibernate.cfg.ImprovedNamingStrategy
hibernate.show_sql=true
hibernate.format_sql=true
{% endhighlight %}

*PersistenceContext.java 파일 생성*
{% highlight java %}
package wjang.spring.data.jpa.example.config;

import java.util.Properties;

import javax.persistence.EntityManagerFactory;
import javax.sql.DataSource;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

@Configuration
@EnableJpaRepositories(basePackages={"wjang.spring.data.jpa.example"})
@EnableTransactionManagement
public class PersistenceContext {
    // configured required beans here
}
{% endhighlight %}

*Configure Datasource Bean*
{% highlight java %}
    @Bean(destroyMethod = "close")
    DataSource datasource(Environment env) {
        HikariConfig dataSourceConfig = new HikariConfig();
        dataSourceConfig.setDataSourceClassName(env.getRequiredProperty("db.driver"));
        dataSourceConfig.setJdbcUrl(env.getRequiredProperty("db.url"));
        dataSourceConfig.setUsername(env.getRequiredProperty("db.username"));
        dataSourceConfig.setPassword(env.getRequiredProperty("db.password"));
        return new HikariDataSource(dataSourceConfig);
    }
{% endhighlight %}

*Configure the Entity Manager Factory Bean*
{% highlight java %}
    @Bean
    LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource, 
            Environment env) {
        LocalContainerEntityManagerFactoryBean entityManagerFactoryBean = new LocalContainerEntityManagerFactoryBean();
        entityManagerFactoryBean.setDataSource(dataSource);
        entityManagerFactoryBean.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
        entityManagerFactoryBean.setPackagesToScan("wjang.spring.data.jpa.example");
        
        Properties jpaProperties = new Properties();
        jpaProperties.put("hibernate.dialect", env.getRequiredProperty("hibernate.dialect"));
        jpaProperties.put("hibernate.hbm2ddl.auto", env.getRequiredProperty("hibernate.hbm2ddl.auto"));
        jpaProperties.put("hibernate.ejb.naming_strategy", env.getRequiredProperty("hibernate.ejb.naming_strategy"));
        jpaProperties.put("hibernate.show_sql", env.getRequiredProperty("hibernate.show_sql"));
        jpaProperties.put("hibernate.format_sql", env.getRequiredProperty("hibernate.format_sql"));
        
        entityManagerFactoryBean.setJpaProperties(jpaProperties);
        
        return entityManagerFactoryBean;
    }
{% endhighlight %}

*Configure Transaction Manager Bean*
{% highlight java %}
    @Bean
    JpaTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {
        JpaTransactionManager transactionManager = new JpaTransactionManager();
        transactionManager.setEntityManagerFactory(entityManagerFactory);
        return transactionManager;
    }
{% endhighlight %}

#### Repository 생성
Repository 생성 시 CrudRepository를 extends할 것인지 아니면 Repository를 extends할 것인지 결정해야 한다. 보통은 Repository를 extends하는 BaseRepository interface를 정의하여 기본 CRUD를 정의한 후 이를 extends 하는 Entity별 Repository를 생성한다.

*BaseRepository*
{% highlight java %}
package wjang.spring.data.jpa.example.repository;

import java.io.Serializable;
import java.util.List;
import java.util.concurrent.Future;

import org.springframework.data.repository.Repository;
import org.springframework.scheduling.annotation.Async;

public interface BaseRepository<T, ID extends Serializable> extends Repository<T, ID>{
    
    void delete(T deleted);
    List<T> findAll();
    T findOne(ID id);
    @Async
    Future<T> save(T persisted);
}
{% endhighlight %}

*query method strategy*
* method 이름은 반드시 아래의 이름으로 시작되어야 한다
  * find...By
  * read...By
  * query...By
  * count...By
  * get...By
* 쿼리 결과의 갯수를 한정하려면 First 또는 Top 키워드를 By 앞에 써준다. First, Top뒤에 숫자는 optional
* unique 한 결과만을 얻고 싶다면 Distinct 키워드를 By 앞에 써준다.

#### Named Query
named query를 정의하는 방법은
* properties 파일에 정의
  * META-INF폴더 하위에 jpa-named-queries.properties 파일에 쿼리를 등록
  {% highlight properties %}
  {% endhighlight %}
* annotation 으로 정의
  * Entity class 상단에 @NamedQuery, @NamedNativeQuery 를 이용하여 정의
  {% highlight java %}
import javax.persistence.Entity;
import javax.persistence.NamedNativeQuery;
import javax.persistence.NamedQuery;
import javax.persistence.Table;
 
@Entity
@NamedNativeQuery(name = "Todo.findBySearchTermNamedNative",
        query="SELECT * FROM todos t WHERE " +
                "LOWER(t.title) LIKE LOWER(CONCAT('%',:searchTerm, '%')) OR " +
                "LOWER(t.description) LIKE LOWER(CONCAT('%',:searchTerm, '%'))",
        resultClass = Todo.class
)
@NamedQuery(name = "Todo.findBySearchTermNamed",
        query = "SELECT t FROM Todo t WHERE " +
                "LOWER(t.title) LIKE LOWER(CONCAT('%', :searchTerm, '%')) OR " +
                "LOWER(t.description) LIKE LOWER(CONCAT('%', :searchTerm, '%'))"
)
@Table(name = "todos")
final class Todo {
 
}
  {% endhighlight %}
* xml 파일에 정의
  * META-INF/orm.xml 파일에 정의
  {% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<entity-mappings
        xmlns="http://java.sun.com/xml/ns/persistence/orm"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/persistence/orm http://java.sun.com/xml/ns/persistence/orm_2_0.xsd"
        version="2.0">
 
    <named-query name="Todo.findBySearchTermNamedOrmXml">
        <query>SELECT t FROM Todo t WHERE LOWER(t.title) LIKE LOWER(CONCAT('%', :searchTerm, '%')) OR LOWER(t.description) LIKE LOWER(CONCAT('%', :searchTerm, '%'))</query>
    </named-query>
 
    <named-native-query name="Todo.findBySearchTermNamedNativeOrmXml"
                        result-class="net.petrikainulainen.springdata.jpa.todo.Todo">
        <query>SELECT * FROM todos t WHERE LOWER(t.title) LIKE LOWER(CONCAT('%',:searchTerm, '%')) OR LOWER(t.description) LIKE LOWER(CONCAT('%',:searchTerm, '%'))</query>
    </named-native-query>
</entity-mappings>
  {% endhighlight %}

#### Creating the Query Methods
{% highlight java %}
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.Repository;
import org.springframework.data.repository.query.Param;
 
import java.util.List;
 
interface TodoRepository extends Repository<Todo, Long> {
 
    List<Todo> findBySearchTermNamed(@Param("searchTerm") String searchTerm);
 
    @Query(nativeQuery = true)
    List<Todo> findBySearchTermNamedNative(@Param("searchTerm") String searchTerm);
}
{% endhighlight %}

#### Sorting query results with the method name of our query methods
**OrderBy 키워드를 이용한 정렬**
* OrderBy를 메소드 이름 뒤에 붙인다
* 정렬하고자 하는 대상 필드 명을 뒤에 붙인다. 첫글자는 대문자
* Asc, Desc 를 명시한다
*Example*
{% highlight java %}
import org.springframework.data.repository.Repository;
import java.util.List;

interface TodoRepository extends Repository<Todo, Long> {
    List<Todo> findByTitleOrderByTitleAsc(String title);
    List<Todo> findByTitleOrderByTitleAscDescriptionDesc(String title);
    List<Todo> findByDescriptionContainsOrTitleContainsAllIgnoreCaseOrderByTitleAsc(String descriptionPart, String titlePart);
}
{% endhighlight %}

## Reference
* [Spring Data JPA Tutorial: Introduction][https://www.petrikainulainen.net/programming/spring-framework/spring-data-jpa-tutorial-introduction/]

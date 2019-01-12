---
title: "How to persist a one to many relationship using Hibernate"
date: 2019-01-12T11:31:48+05:30
---

Many times during the software development process we have to create database and define relationships between tables. This is a really time consuming and error prone task and even if you forget to add a column or constrant it is a painful task to do. So most of times people use Object Relational Mapper(ORM) for this purpose. Hibernate is one of the ORM that is used for mapping the objects to the relational model where entities/classes are mapped to tables, instances are mapped to rows and attributes of instances are mapped to columns of table.

In this tutorial we are going to implement a one-to-many relationship between the entities in Hibernate. We are going to create the following tables in the database with the help of Hibernate.

![Database Schema](/images/tutorials/er_diagram_hibernate.png)

One tip while using ORM is when you create tables you should try to map it to a class(entity). For example, here in database we have a person information table. Hence we should have a corresponding Person Entity in Hibernate. Now lets get started with the project.

For my day to day development I use IntelliJ Community Edition. Its free and you can download it from here. We will use [Gradle](https://gradle.org/) for dependency management of our project. Now create a gradle project. If you want to know about gradle checkout my blog on Gradle. The build.gradle file should have the following dependencies.

## build.gradle
```java
dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
    compile 'org.hibernate:hibernate-core:5.3.7.Final'
    compileOnly 'org.projectlombok:lombok:1.18.+'
    runtime 'org.postgresql:postgresql:42.1.1'
    compile 'javax.xml.bind:jaxb-api:2.3.0'
    compile 'com.sun.xml.bind:jaxb-impl:2.3.0'
    compile 'com.sun.xml.bind:jaxb-core:2.3.0'
}
```

In the main->java folder create a package entity. Here we will define the entities for the tables in the database. There will be three entities for the tables. Create the following classes inside the entity folder.

## Person Entity
```java
import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;
import java.util.Set;

@Getter
@Setter
@Entity
@Table(name = "person_info")
public class Person {
    @Id
    @Column(name = "user_id", nullable = false)
    String userId;
    @Column(name = "first_name")
    String firstName;
    @Column(name = "last_name")
    String lastName;
    String email;
    @Column(name = "phone_number", length = 10)
    String phoneNumber;
    @OneToMany(cascade = CascadeType.ALL,mappedBy = "person")
    Set<Order> orderSet;
}
```

## Order Entity
```java
import lombok.Getter;
import lombok.Setter;
import org.hibernate.annotations.Generated;
import org.hibernate.annotations.GenerationTime;

import javax.persistence.*;
import java.util.List;
import java.util.Set;

@Getter
@Setter
@Entity
@Table(name = "order_details")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    @Column(name = "order_name")
    String orderName;
    @Column(name = "total_amount")
    Double totalAmount = 0.0;
    @Column(name = "currency_code")
    String currencyCode;
    @OneToMany(cascade = CascadeType.ALL,mappedBy = "order")
    List<OrderItem> orderItems;
    @ManyToOne
    @JoinColumn(name = "person_id")
    Person person;
}
```

## OrderItem Entity
```java
import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;

@Getter
@Setter
@Entity
@Table(name = "order_item")

public class OrderItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    @Column(name = "product_name")
    private String productName;
    @Column(name = "product_category")
    String productCategory;
    Double price;
    @Column(name = "currency_code")
    String currencyCode;
    @ManyToOne
    @JoinColumn(name = "order_id")
    private Order order;
}
```
Important annotations in the code are as follows:

* **@Getter** : This is just an annotation of Lombok which will create getters for all the properties of a class
* **@Setter** : This is just an annotation of Lombok which will create setters for all the properties of a class
* **@Entity** : This annotation declares that class is an entity in Hibernate
* **@Table(name = "")** : This annotation is used to declare the name of table mapped by this entity in database
* **@Id** : This annotation is used to declare the property is the primary key of table
* **@GeneratedValue(strategy = GenerationType.IDENTITY)** : This annotation is used to generate the auto-increment feature of databases.
* **@Column(name = "")** : This annotation is used to define the name of the column in table for the given property
* **@OneToMany(cascade = "",mappedBy = "")** : This annotation is used to define one to many relationships between the entities. For example a Person can have one or many orders. Hence the Person Entity we have declared a Set of orders and defined that annotation there. mappedBy is the name of the property of first entity in the other entity of the relationship.
* **@ManyToOne** : This annotation is used to define many to one relationships between entities.
* **@JoinColumn(name = "")** : This annotation is used to define who is the owner in the relationship. For example here in Order Entity, Order is the owner of person-order relationship. This will create a column in order table with the value given in name parameter.


Now you may be guessing if I can define **@JoinColumn** annotation with **@OneToMany**. But there will unnecessary more SQL queries that needs to be executed. You can read [this article](https://vladmihalcea.com/the-best-way-to-map-a-onetomany-association-with-jpa-and-hibernate/) and this [stackoverflow answer](https://stackoverflow.com/questions/11938253/whats-the-difference-between-joincolumn-and-mappedby-when-using-a-jpa-onetoma) for better understanding.

After defining the entities, we will try to create a few records in the table. We will create a helper class which will return a sessionFactory which then can be used to create sessions with database. In the main->resources folder, create hibernate.cfg.xml file. This file is used to provide the connection parameters, the mappings and strategy for table creation to Hibernate.

## hibernate.cfg.xml

```xml
<?xml version = "1.0" encoding = "utf-8"?>
<!DOCTYPE hibernate-configuration SYSTEM
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <property name="hibernate.dialect">org.hibernate.dialect.PostgreSQL9Dialect</property>
        <property name="hibernate.connection.driver_class">org.postgresql.Driver</property>
        <property name="hibernate.connection.url">jdbc:postgresql://localhost:5432/postgres_test</property>
        <property name="hibernate.connection.username">dbuser</property>
        <property name="hibernate.connection.password">dbuser</property>

        <property name="connection_pool_size">1</property>
        <property name="hbm2ddl.auto">create-drop</property>
        <property name="show_sql">true</property>
        <property name="current_session_context_class">thread</property>


        <property name="hibernate.dbcp.initialSize">5</property>
        <property name="hibernate.dbcp.maxTotal">20</property>
        <property name="hibernate.dbcp.maxIdle">10</property>
        <property name="hibernate.dbcp.minIdle">5</property>
        <property name="hibernate.dbcp.maxWaitMillis">-1</property>

        <mapping class="entity.Order"/>
        <mapping class="entity.OrderItem"/>
        <mapping class="entity.Person"/>

    </session-factory>
</hibernate-configuration>
```

Once this is done we will create a helper class HibernateUtil.

## HibernateUtil.java
```java
import org.hibernate.SessionFactory;
import org.hibernate.boot.Metadata;
import org.hibernate.boot.MetadataSources;
import org.hibernate.boot.registry.StandardServiceRegistry;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;

public class    HibernateUtil {
    private static StandardServiceRegistry registry;
    private static SessionFactory sessionFactory;

    public static SessionFactory getSessionFactory(){
        try{
            registry = new StandardServiceRegistryBuilder().configure().build();

            MetadataSources sources = new MetadataSources(registry);
            Metadata metadata = sources.getMetadataBuilder().build();
            sessionFactory = metadata.getSessionFactoryBuilder().build();
        }catch (Exception e){
            e.printStackTrace();
            if(registry != null){
                StandardServiceRegistryBuilder.destroy(registry);
            }
        }
        return sessionFactory;
    }

    public static void shutdown(){
        if(registry != null){
            StandardServiceRegistryBuilder.destroy(registry);
        }
    }
}
```

Once these things are ready, we will run the program. This is the main function of the program.
```java
import entity.Order;
import entity.OrderItem;
import entity.Person;
import helper.HibernateUtil;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

import java.util.ArrayList;
import java.util.HashSet;

public class MainClass {
    public static void main(String[] args){
        Transaction transaction = null;
        Person person = new Person();
        person.setUserId("person_1213");
        person.setFirstName("Karan");
        person.setLastName("Khanchandani");
        person.setEmail("person1@gmail.com");
        person.setOrderSet(new HashSet<>());

        OrderItem orderItem1 = new OrderItem();
        orderItem1.setProductName("Laptop");
        orderItem1.setProductCategory("Electronics");
        orderItem1.setCurrencyCode("INR");
        orderItem1.setPrice(55000.212);

        OrderItem orderItem2 = new OrderItem();
        orderItem2.setProductName("Shirt");
        orderItem2.setProductCategory("Clothing");
        orderItem2.setCurrencyCode("INR");
        orderItem2.setPrice(500.00);

        OrderItem orderItem3 = new OrderItem();
        orderItem3.setProductName("Jeans");
        orderItem3.setProductCategory("Clothing");
        orderItem3.setCurrencyCode("INR");
        orderItem3.setPrice(1022.11);

        OrderItem orderItem4 = new OrderItem();
        orderItem4.setProductName("Keyboard");
        orderItem4.setProductCategory("Electronics");
        orderItem4.setCurrencyCode("INR");
        orderItem4.setPrice(233.11);

        OrderItem orderItem5 = new OrderItem();
        orderItem5.setProductName("Mouse");
        orderItem5.setProductCategory("Electronics");
        orderItem5.setCurrencyCode("INR");
        orderItem5.setPrice(199.99);

        Order order1 = new Order();
        order1.setOrderName("Order 1");
        order1.setOrderItems(new ArrayList<>());
        order1.getOrderItems().add(orderItem1);
        order1.getOrderItems().add(orderItem2);

        Order order2 = new Order();
        order2.setOrderName("Order 1");
        order2.setOrderItems(new ArrayList<>());
        order2.getOrderItems().add(orderItem3);
        order2.getOrderItems().add(orderItem4);
        order2.getOrderItems().add(orderItem5);

        //This step is really important
        person.getOrderSet().add(order1);
        person.getOrderSet().add(order2);
        order1.setPerson(person);
        order2.setPerson(person);

        orderItem1.setOrder(order1);
        orderItem2.setOrder(order1);
        orderItem3.setOrder(order2);
        orderItem4.setOrder(order2);
        orderItem5.setOrder(order2);

        for(OrderItem orderItem: order1.getOrderItems()){
            order1.setTotalAmount(order1.getTotalAmount() + orderItem.getPrice());
            order1.setCurrencyCode(orderItem.getCurrencyCode());
        }

        for(OrderItem orderItem: order2.getOrderItems()){
            order2.setTotalAmount(order2.getTotalAmount() + orderItem.getPrice());
            order2.setCurrencyCode(orderItem.getCurrencyCode());
        }
        Session session = HibernateUtil.getSessionFactory().openSession();
        try {
            System.out.println("Started inserting the data");
            transaction = session.beginTransaction();
            session.saveOrUpdate(person);
            transaction.commit();
            System.out.println("Data inserted");
        }catch (Exception e){
            e.printStackTrace();
            if(transaction != null){
                transaction.rollback();
            }
        }finally {

        }
    }
}
```
Here make sure to map the parent child references properly in the code. Otherwise it won't work properly.

Thats it! Once the program runs you can see that the entries have been created in the database. You can learn more about Hibernate by following the its [documentation](http://docs.jboss.org/hibernate/annotations/3.5/reference/en/html_single/) page. It has tons of examples with great explaination.
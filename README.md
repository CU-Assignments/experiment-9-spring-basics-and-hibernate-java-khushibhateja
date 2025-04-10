[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/9RndFDpw)

Create a simple Spring application that demonstrates Dependency Injection (DI) using Java-based configuration instead of XML. Define a Student class that depends on a Course class. Use Spring's @Configuration and @Bean annotations to inject dependencies. Requirements:

Define a Course class with attributes courseName and duration.
Define a Student class with attributes name and a reference to Course.
Use Java-based configuration (@Configuration and @Bean) to configure the beans.
Load the Spring context in the main method and print student details.

```
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.*;

// Course class class Course { private String courseName; private int duration;

public Course(String courseName, int duration) {
    this.courseName = courseName;
    this.duration = duration;
}

public String getCourseName() {
    return courseName;
}

public int getDuration() {
    return duration;
}
}

// Student class class Student { private String name; private Course course;

public Student(String name, Course course) {
    this.name = name;
    this.course = course;
}

public void displayDetails() {
    System.out.println("Student Name: " + name);
    System.out.println("Course: " + course.getCourseName());
    System.out.println("Duration: " + course.getDuration() + " weeks");
}
}

// Configuration class @Configuration class AppConfig { @Bean public Course course() { return new Course("Spring Framework", 6); }

@Bean
public Student student() {
    return new Student("Alice", course());
}
}

// Main class public class SpringDIExample {
     public static void main(String[] args) {
       ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class); Student student = context.getBean(Student.class);
       student.displayDetails();
 }
}
```

# Develop a Hibernate-based application to perform CRUD (Create, Read, Update, Delete) operations on a Student entity using Hibernate ORM with MySQL.
Requirements:
1. Configure Hibernate using hibernate.cfg.xml.
2. Create an Entity class (Student.java) with attributes: id, name, and age.
3. Implement Hibernate SessionFactory to perform CRUD operations.
4. Test the CRUD functionality with sample data.

```
import org.hibernate.*; import org.hibernate.cfg.Configuration;

import javax.persistence.*;

// 1. Entity Class @Entity @Table(name = "students") class Student { @Id @GeneratedValue(strategy = GenerationType.IDENTITY) private int id;

private String name;
private int age;

public Student() {} // Required by Hibernate

public Student(String name, int age) {
    this.name = name;
    this.age = age;
}

// Getters & setters
public int getId() { return id; }
public String getName() { return name; }
public int getAge() { return age; }

public void setName(String name) { this.name = name; }
public void setAge(int age) { this.age = age; }

public String toString() {
    return "Student [id=" + id + ", name=" + name + ", age=" + age + "]";
}
}

// 2. Main Class public class MainApp { public static void main(String[] args) { SessionFactory factory = new Configuration().configure("hibernate.cfg.xml") .addAnnotatedClass(Student.class) .buildSessionFactory();

    // Create
    try (Session session = factory.getCurrentSession()) {
        Student student = new Student("John Doe", 22);
        session.beginTransaction();
        session.save(student);
        session.getTransaction().commit();
        System.out.println("Created: " + student);
    }

    int studentId = 1; // Use appropriate ID (likely auto-generated)

    // Read
    try (Session session = factory.getCurrentSession()) {
        session.beginTransaction();
        Student student = session.get(Student.class, studentId);
        System.out.println("Read: " + student);
        session.getTransaction().commit();
    }

    // Update
    try (Session session = factory.getCurrentSession()) {
        session.beginTransaction();
        Student student = session.get(Student.class, studentId);
        student.setAge(25);
        session.getTransaction().commit();
        System.out.println("Updated: " + student);
    }

    // Delete
    try (Session session = factory.getCurrentSession()) {
        session.beginTransaction();
        Student student = session.get(Student.class, studentId);
        if (student != null) {
            session.delete(student);
            System.out.println("Deleted: " + student);
        }
        session.getTransaction().commit();
    }

    factory.close();
}
}
```
# Hard Level:

Develop a Spring-based application integrated with Hibernate to manage transactions. Create a banking system where users can transfer money between accounts, ensuring transaction consistency.
Requirements:
1. Use Spring configuration with Hibernate ORM.
2. Implement two entity classes (Account.java and Transaction.java).
3. Use Hibernate Transaction Management to ensure atomic operations
4. If a transaction fails, rollback should occur.
5. Demonstrate successful and failed transactions.
```
import org.hibernate.SessionFactory; import org.hibernate.cfg.Configuration; import org.hibernate.Session; import org.springframework.context.annotation.*; import org.springframework.orm.hibernate5.HibernateTransactionManager; import org.springframework.transaction.annotation.EnableTransactionManagement; import org.springframework.transaction.annotation.Transactional;

import javax.persistence.*;

@Configuration @EnableTransactionManagement @ComponentScan(basePackages = "banking") public class BankingApp {

// ============ Entities ============

@Entity
@Table(name = "accounts")
public static class Account {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String owner;
    private double balance;

    public Account() {}
    public Account(String owner, double balance) {
        this.owner = owner;
        this.balance = balance;
    }

    // Getters and setters
    public int getId() { return id; }
    public String getOwner() { return owner; }
    public double getBalance() { return balance; }
    public void setBalance(double balance) { this.balance = balance; }

    public String toString() {
        return "Account[id=" + id + ", owner=" + owner + ", balance=" + balance + "]";
    }
}

@Entity
@Table(name = "transactions")
public static class TransactionRecord {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private int fromAccount;
    private int toAccount;
    private double amount;

    public TransactionRecord() {}
    public TransactionRecord(int from, int to, double amount) {
        this.fromAccount = from;
        this.toAccount = to;
        this.amount = amount;
    }

    public String toString() {
        return "Transaction[id=" + id + ", from=" + fromAccount + ", to=" + toAccount + ", amount=" + amount + "]";
    }
}

// ============ DAO Service ============

@org.springframework.stereotype.Service
public static class BankService {
    private final SessionFactory sessionFactory;

    public BankService(SessionFactory sessionFactory) {
        this.sessionFactory = sessionFactory;
    }

    @Transactional
    public void transferMoney(int fromId, int toId, double amount, boolean fail) {
        Session session = sessionFactory.getCurrentSession();

        Account from = session.get(Account.class, fromId);
        Account to = session.get(Account.class, toId);

        if (from.getBalance() < amount) {
            throw new RuntimeException("Insufficient funds.");
        }

        from.setBalance(from.getBalance() - amount);
        to.setBalance(to.getBalance() + amount);

        session.save(new TransactionRecord(fromId, toId, amount));

        if (fail) {
            throw new RuntimeException("Forced failure to test rollback.");
        }
    }

    @Transactional
    public void createAccounts() {
        Session session = sessionFactory.getCurrentSession();
        session.save(new Account("Alice", 1000));
        session.save(new Account("Bob", 500));
    }

    @Transactional
    public void showAccounts() {
        Session session = sessionFactory.getCurrentSession();
        session.createQuery("from BankingApp$Account", Account.class).list()
               .forEach(System.out::println);
    }
}

// ============ Spring Beans ============

@Bean
public SessionFactory sessionFactory() {
    return new Configuration()
            .configure("hibernate.cfg.xml")
            .addAnnotatedClass(Account.class)
            .addAnnotatedClass(TransactionRecord.class)
            .buildSessionFactory();
}

@Bean
public HibernateTransactionManager transactionManager(SessionFactory sf) {
    return new HibernateTransactionManager(sf);
}

// ============ Main Method ============

public static void main(String[] args) {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(BankingApp.class);
    BankService bankService = context.getBean(BankService.class);

    System.out.println("Creating accounts...");
    bankService.createAccounts();

    System.out.println("\nAccounts BEFORE transfer:");
    bankService.showAccounts();

    try {
        System.out.println("\nPerforming successful transfer...");
        bankService.transferMoney(1, 2, 200, false);
    } catch (Exception e) {
        System.out.println("Transfer failed: " + e.getMessage());
    }

    try {
        System.out.println("\nPerforming FAILED transfer (should rollback)...");
        bankService.transferMoney(1, 2, 300, true); // This will fail and rollback
    } catch (Exception e) {
        System.out.println("Transfer failed: " + e.getMessage());
    }

    System.out.println("\nAccounts AFTER transfers:");
    bankService.showAccounts();

    context.close();
}
}
```

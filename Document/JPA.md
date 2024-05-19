## Hướng dẫn Sử dụng JPA Hibernate với SQL Server và Persistence

Tài liệu này cung cấp hướng dẫn chi tiết về cách sử dụng JPA Hibernate để thao tác với cơ sở dữ liệu SQL Server trong ứng dụng Java.

**Mục lục:**

1. Chuẩn bị Môi trường
2. Tạo Entity
3. Thao tác Dữ liệu
4. Tạo Bảng
5. Tạo lớp DAO và Implement
6. Các thuộc tính trong Annotation để tạo quan hệ giữa các bảng trong JPA
7. Ví dụ về các thuộc tính trong Annotation để tạo quan hệ giữa các bảng trong JPA

## 1. Chuẩn bị Môi trường

**1.1. Thêm Maven Dependencies:**

Thêm các dependencies sau vào file `pom.xml` của dự án Maven:

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.6.10.Final</version>
</dependency>
<dependency>
    <groupId>com.microsoft.sqlserver</groupId>
    <artifactId>mssql-jdbc</artifactId>
    <version>9.4.0.jre8</version>
</dependency>
```

**1.2. Cấu hình Persistence.xml:**

Tạo file `persistence.xml` trong thư mục `src/main/resources/META-INF/` và cấu hình các thuộc tính kết nối đến SQL Server:

```xml
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd"
             version="2.2">

    <persistence-unit name="MyPersistenceUnit">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <properties>
            <property name="jakarta.persistence.jdbc.driver" value="com.microsoft.sqlserver.jdbc.SQLServerDriver"/>
            <property name="jakarta.persistence.jdbc.url" value="jdbc:sqlserver://localhost:1433;databaseName=your_database_name"/>
            <property name="jakarta.persistence.jdbc.user" value="your_username"/>
            <property name="jakarta.persistence.jdbc.password" value="your_password"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.SQLServerDialect"/>
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
        </properties>
    </persistence-unit>
</persistence>
```

**Lưu ý:** Thay thế `your_database_name`, `your_username`, và `your_password` bằng thông tin đăng nhập SQL Server của bạn.

## 2. Tạo Entity

Entity là các lớp Java đại diện cho các bảng trong cơ sở dữ liệu. JPA sử dụng annotations để ánh xạ các thuộc tính của lớp tới các cột trong bảng.

**2.1. Entity cơ bản:**

```java
import jakarta.persistence.*;

@Entity
@Table(name = "employees")
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "employee_name", length = 100, columnDefinition = "VARCHAR(100)") // Sử dụng VARCHAR
    private String name;

    @Column(name = "email", nullable = false, unique = true, columnDefinition = "NVARCHAR(255)") // Sử dụng NVARCHAR
    private String email;

    // ... Các thuộc tính khác

    // Getters and Setters
}
```

**Giải thích:**

- `@Entity`: Khai báo lớp là một entity JPA.
- `@Table(name = "employees")`: Ánh xạ lớp tới bảng `employees` trong cơ sở dữ liệu.
- `@Id`: Khai báo thuộc tính `id` là khóa chính của bảng.
- `@GeneratedValue(strategy = GenerationType.IDENTITY)`: Chỉ định strategy để tạo giá trị cho khóa chính.
- `@Column(name = "employee_name")`: Ánh xạ thuộc tính `name` tới cột `employee_name` trong bảng.
- `columnDefinition = "VARCHAR(100)"`: Xác định trường `employee_name` trong bảng SQL là kiểu `VARCHAR` có độ dài tối đa là 100 ký tự.
- `columnDefinition = "NVARCHAR(255)"`: Xác định trường `email` trong bảng SQL là kiểu `NVARCHAR` có độ dài tối đa là 255 ký tự.
  **2.2. Entity với Enum:**

```java
@Entity
@Table(name = "products")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @Enumerated(EnumType.STRING)
    private Category category;

    // Getters and Setters

    public enum Category {
        ELECTRONICS, CLOTHING, FOOD
    }
}
```

**Giải thích:**

- `@Enumerated(EnumType.STRING)`: Ánh xạ thuộc tính `category` (kiểu Enum) tới một cột trong bảng dưới dạng chuỗi.

**2.3. Entity với Kế thừa:**

JPA hỗ trợ ba loại chiến lược kế thừa:

- **Table Per Class (TPC):** Mỗi lớp con được ánh xạ tới một bảng riêng biệt.
- **Table Per Hierarchy (TPH):** Tất cả các lớp trong hệ thống phân cấp kế thừa được ánh xạ tới một bảng duy nhất.
- **Table Per Subclass (TPS):** Mỗi lớp con được ánh xạ tới một bảng riêng biệt, nhưng bảng này chia sẻ khóa chính với bảng của lớp cha.

**Ví dụ TPC:**

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Person {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    // Getters and Setters
}

@Entity
@Table(name = "employees")
public class Employee extends Person {
    private String department;

    // Getters and Setters
}

@Entity
@Table(name = "customers")
public class Customer extends Person {
    private String address;

    // Getters and Setters
}
```

**2.4. Entity với Quan hệ:**

JPA hỗ trợ các loại quan hệ sau:

- **One-to-One:** Một thực thể liên kết với chính xác một thực thể khác.
- **One-to-Many:** Một thực thể liên kết với nhiều thực thể khác.
- **Many-to-One:** Nhiều thực thể liên kết với một thực thể khác.
- **Many-to-Many:** Nhiều thực thể liên kết với nhiều thực thể khác.

**Ví dụ One-to-Many:**

```java
@Entity
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL)
    private List<Employee> employees = new ArrayList<>();

    // Getters and Setters
}

@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;

    // Getters and Setters
}
```

## 3. Thao tác Dữ liệu

**3.1. Khởi tạo EntityManager:**

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("MyPersistenceUnit");
EntityManager em = emf.createEntityManager();
```

**3.2. Thêm dữ liệu:**

```java
Employee employee = new Employee();
employee.setName("John Doe");
employee.setEmail("john.doe@example.com");

em.getTransaction().begin();
em.persist(employee);
em.getTransaction().commit();
```

**3.3. Xóa dữ liệu:**

```java
Employee employee = em.find(Employee.class, 1L);

em.getTransaction().begin();
em.remove(employee);
em.getTransaction().commit();
```

**3.4. Sửa dữ liệu:**

```java
Employee employee = em.find(Employee.class, 1L);
employee.setName("Jane Doe");

em.getTransaction().begin();
em.merge(employee);
em.getTransaction().commit();
```

**3.5. Lấy dữ liệu:**

```java
// Lấy employee theo ID
Employee employee = em.find(Employee.class, 1L);

// Lấy tất cả employees
TypedQuery<Employee> query = em.createQuery("SELECT e FROM Employee e", Employee.class);
List<Employee> employees = query.getResultList();
```

## 4. Tạo Bảng

Hibernate có thể tự động tạo bảng dựa trên cấu hình của Entity. Sử dụng thuộc tính `hibernate.hbm2ddl.auto` trong `persistence.xml` để kiểm soát quá trình tạo bảng:

- `create`: Xóa bảng hiện có và tạo bảng mới mỗi khi chạy ứng dụng.
- `create-drop`: Tương tự `create`, nhưng xóa bảng sau khi đóng EntityManagerFactory.
- `update`: Cập nhật schema của bảng nếu có thay đổi trong Entity.
- `validate`: Kiểm tra xem schema của bảng có khớp với Entity hay không, nếu không sẽ báo lỗi.
- `none`: Không thực hiện bất kỳ thao tác nào với schema của bảng.

## 5. Tạo lớp DAO và Implement

DAO (Data Access Object) là các lớp cung cấp một giao diện trừu tượng để thao tác với cơ sở dữ liệu.

**5.1. Generic DAO Interface:**

```java
import java.io.Serializable;
import java.util.List;

public interface GenericDAO<T, ID extends Serializable> {

    T findById(ID id);

    List<T> findAll();

    T save(T entity);

    T update(T entity);

    void delete(T entity);

    void deleteById(ID id);

}
```

**5.2. Generic DAO Implement:**

```java
import jakarta.persistence.EntityManager;
import jakarta.persistence.PersistenceContext;
import java.io.Serializable;
import java.lang.reflect.ParameterizedType;
import java.util.List;

public abstract class GenericDAOImpl<T, ID extends Serializable> implements GenericDAO<T, ID> {

    @PersistenceContext
    protected EntityManager entityManager;

    private Class<T> persistentClass;

    @SuppressWarnings("unchecked")
    public GenericDAOImpl() {
        this.persistentClass = (Class<T>) ((ParameterizedType) this.getClass().getGenericSuperclass())
                .getActualTypeArguments()[0];
    }

    @Override
    public T findById(ID id) {
        return entityManager.find(persistentClass, id);
    }

    @Override
    public List<T> findAll() {
        return entityManager.createQuery("SELECT e FROM " + persistentClass.getSimpleName() + " e", persistentClass)
                .getResultList();
    }

    @Override
    public T save(T entity) {
        entityManager.persist(entity);
        return entity;
    }

    @Override
    public T update(T entity) {
        return entityManager.merge(entity);
    }

    @Override
    public void delete(T entity) {
        entityManager.remove(entity);
    }

    @Override
    public void deleteById(ID id) {
        T entity = findById(id);
        if (entity != null) {
            delete(entity);
        }
    }
}
```

**5.3. Employee DAO Interface và Implement:**

```java
import java.util.List;

public interface EmployeeDAO extends GenericDAO<Employee, Long> {

    List<Employee> findByDepartment(String departmentName);

    List<Employee> findByProjectName(String projectName);

    List<Employee> findByName(String name);

}
```

```java
import jakarta.persistence.TypedQuery;
import java.util.List;

public class EmployeeDAOImpl extends GenericDAOImpl<Employee, Long> implements EmployeeDAO {

    @Override
    public List<Employee> findByDepartment(String departmentName) {
        TypedQuery<Employee> query = entityManager.createQuery("SELECT e FROM Employee e WHERE e.department.name = :departmentName", Employee.class);
        query.setParameter("departmentName", departmentName);
        return query.getResultList();
    }

    @Override
    public List<Employee> findByProjectName(String projectName) {
        TypedQuery<Employee> query = entityManager.createQuery("SELECT e FROM Employee e JOIN e.projects p WHERE p.name = :projectName", Employee.class);
        query.setParameter("projectName", projectName);
        return query.getResultList();
    }

    @Override
    public List<Employee> findByName(String name) {
        TypedQuery<Employee> query = entityManager.createQuery("SELECT e FROM Employee e WHERE e.name = :name", Employee.class);
        query.setParameter("name", name);
        return query.getResultList();
    }
}
```

## 6. Các thuộc tính trong Annotation để tạo quan hệ giữa các bảng trong JPA

JPA cung cấp các annotations để tạo các quan hệ giữa các bảng trong cơ sở dữ liệu.

**6.1. @OneToOne:**

- **cascade:** Xác định cách thức cascaded operations (persist, merge, remove, refresh, detach) được áp dụng cho thực thể liên quan.
  - `CascadeType.ALL`: Áp dụng tất cả cascaded operations.
  - `CascadeType.PERSIST`: Áp dụng cascaded persist operation.
  - `CascadeType.MERGE`: Áp dụng cascaded merge operation.
  - `CascadeType.REMOVE`: Áp dụng cascaded remove operation.
  - `CascadeType.REFRESH`: Áp dụng cascaded refresh operation.
  - `CascadeType.DETACH`: Áp dụng cascaded detach operation.
- **fetch:** Xác định kiểu fetch (EAGER hoặc LAZY) cho quan hệ.
  - `FetchType.EAGER`: Dữ liệu liên quan được nạp cùng lúc với thực thể chính.
  - `FetchType.LAZY`: Dữ liệu liên quan chỉ được nạp khi được truy cập.
- **optional:** Xác định xem quan hệ có phải là bắt buộc hay không. Mặc định là `true`.
- **orphanRemoval:** Xác định xem thực thể con có bị xóa khi bị tách khỏi thực thể cha hay không. Mặc định là `false`.
- **mappedBy:** Được sử dụng ở phía non-owning của quan hệ để chỉ định thuộc tính trong thực thể sở hữu quan hệ.

**6.2. @OneToMany:**

- **cascade:** Giống như `@OneToOne`.
- **fetch:** Giống như `@OneToOne`.
- **orphanRemoval:** Giống như `@OneToOne`.
- **mappedBy:** Được sử dụng ở phía owning của quan hệ để chỉ định thuộc tính trong thực thể sở hữu quan hệ.

**6.3. @ManyToOne:**

- **cascade:** Giống như `@OneToOne`.
- **fetch:** Giống như `@OneToOne`.
- **optional:** Giống như `@OneToOne`.

**6.4. @ManyToMany:**

- **cascade:** Giống như `@OneToOne`.
- **fetch:** Giống như `@OneToOne`.
- **mappedBy:** Được sử dụng ở phía owning của quan hệ để chỉ định thuộc tính trong thực thể sở hữu quan hệ.
- **targetEntity:** Xác định kiểu của thực thể liên quan.

**6.5. Các thuộc tính chung:**

- **@JoinColumn:** Được sử dụng để chỉ định tên cột trong bảng hiện tại liên kết với bảng khác.
- **@JoinTable:** Được sử dụng trong quan hệ `@ManyToMany` để chỉ định bảng trung gian kết nối hai bảng.

## 7. Ví dụ về các thuộc tính trong Annotation để tạo quan hệ giữa các bảng trong JPA

**7.1. @OneToOne:**

```java
@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "address_id")
    private Address address;

    // Getters and Setters
}

@Entity
public class Address {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String street;
    private String city;

    // Getters and Setters
}
```

**7.2. @OneToMany:**

```java
@Entity
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL)
    private List<Employee> employees = new ArrayList<>();

    // Getters and Setters
}

@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;

    // Getters and Setters
}
```

**7.3. @ManyToMany:**

```java
@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(name = "employee_project",
            joinColumns = @JoinColumn(name = "employee_id"),
            inverseJoinColumns = @JoinColumn(name = "project_id"))
    private List<Project> projects = new ArrayList<>();

    // Getters and Setters
}

@Entity
public class Project {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToMany(mappedBy = "projects")
    private List<Employee> employees = new ArrayList<>();

    // Getters and Setters
}
```

## Hướng dẫn sử dụng JUnit 5

JUnit là một framework kiểm thử đơn vị phổ biến cho Java. JUnit 5 là phiên bản mới nhất của framework này với nhiều cải tiến so với các phiên bản trước.

### 1. Cài đặt

Bạn cần thêm dependencies sau vào file `pom.xml` (nếu bạn đang sử dụng Maven):

```xml
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter-api</artifactId>
  <version>5.8.1</version>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter-engine</artifactId>
  <version>5.8.1</version>
  <scope>test</scope>
</dependency>
```

### 2. Viết Test Case

- Mỗi test case là một method được chú thích bằng `@Test`.
- Tên method nên mô tả rõ ràng mục đích của test case.
- Sử dụng các Assertion methods để kiểm tra kết quả mong muốn.

**Ví dụ:**

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class CalculatorTest {

    @Test
    void testAdd() {
        Calculator calculator = new Calculator();
        int result = calculator.add(2, 3);
        assertEquals(5, result);
    }

    @Test
    void testSubtract() {
        Calculator calculator = new Calculator();
        int result = calculator.subtract(5, 2);
        assertEquals(3, result);
    }
}
```

### 3. Assertions

JUnit 5 cung cấp nhiều Assertion methods để kiểm tra các điều kiện khác nhau.

| Method                                        | Mô tả                                                              |
| --------------------------------------------- | ------------------------------------------------------------------ |
| `assertEquals(expected, actual)`              | Kiểm tra xem hai giá trị có bằng nhau hay không.                   |
| `assertNotEquals(unexpected, actual)`         | Kiểm tra xem hai giá trị có khác nhau hay không.                   |
| `assertTrue(condition)`                       | Kiểm tra xem điều kiện có đúng hay không.                          |
| `assertFalse(condition)`                      | Kiểm tra xem điều kiện có sai hay không.                           |
| `assertNull(object)`                          | Kiểm tra xem object có phải là `null` hay không.                   |
| `assertNotNull(object)`                       | Kiểm tra xem object có khác `null` hay không.                      |
| `assertSame(expected, actual)`                | Kiểm tra xem hai object có cùng tham chiếu hay không.              |
| `assertNotSame(unexpected, actual)`           | Kiểm tra xem hai object có khác tham chiếu hay không.              |
| `assertThrows(expectedException, executable)` | Kiểm tra xem một đoạn code có throw exception mong muốn hay không. |
| `assertAll(executables)`                      | Kiểm tra nhiều assertions cùng lúc.                                |

### 4. Life Cycle Annotations

JUnit 5 cung cấp các Annotations để quản lý vòng đời của test:

| Annotation    | Mô tả                                                    |
| ------------- | -------------------------------------------------------- |
| `@BeforeEach` | Thực thi trước mỗi test case.                            |
| `@AfterEach`  | Thực thi sau mỗi test case.                              |
| `@BeforeAll`  | Thực thi một lần trước tất cả các test case trong class. |
| `@AfterAll`   | Thực thi một lần sau tất cả các test case trong class.   |

**Ví dụ:**

```java
import org.junit.jupiter.api.*;

public class CalculatorTest {

    Calculator calculator;

    @BeforeAll
    static void setup() {
        System.out.println("Setting up...");
    }

    @BeforeEach
    void init() {
        calculator = new Calculator();
    }

    @Test
    void testAdd() {
        int result = calculator.add(2, 3);
        assertEquals(5, result);
    }

    @AfterEach
    void tearDown() {
        System.out.println("Tearing down...");
    }

    @AfterAll
    static void done() {
        System.out.println("Finished!");
    }
}
```

### 5. Anotation

Trong JUnit 5, để các test case chạy lần lượt theo thứ tự mà chúng được định nghĩa trong code, bạn có thể sử dụng annotation `@TestMethodOrder` cùng với một `MethodOrderer` cụ thể.

Dưới đây là ví dụ về cách sử dụng `@TestMethodOrder` với `MethodOrderer.OrderAnnotation`:

```java
import org.junit.jupiter.api.*;

@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
public class OrderedTest {

    @Test
    @Order(1)
    void testA() {
        System.out.println("Test A");
    }

    @Test
    @Order(2)
    void testB() {
        System.out.println("Test B");
    }

    @Test
    @Order(3)
    void testC() {
        System.out.println("Test C");
    }
}
```

**Giải thích:**

1. **`@TestMethodOrder(MethodOrderer.OrderAnnotation.class)`**: Annotation này được đặt ở cấp độ lớp để chỉ định rằng các phương thức kiểm thử trong lớp này sẽ được thực thi theo thứ tự được chỉ định bởi annotation `@Order`.

2. **`@Order(n)`**: Annotation này được đặt trên mỗi phương thức kiểm thử để chỉ định thứ tự thực thi của phương thức đó. Giá trị `n` là một số nguyên, với các phương thức có giá trị `n` thấp hơn sẽ được thực thi trước.

### Tài liệu tham khảo

- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [JUnit 5 API Documentation](https://junit.org/junit5/docs/current/api/)

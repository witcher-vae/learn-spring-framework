![](./img/spring-boot.png)

[Java cơ bản](./DOC/java-basic.md)

[Java OOP](./DOC/java-oop.md)

------------------
table of contents

Spring boot

1. [Các khái niệm cần nắm](#spring_other)

2. [Cách cài đặt](#spring_install)

3. [Spring boot @Component và @Autowired](#springboot_1)

   3.1. [@Component](#spring_component)
   
   3.3.2. [@Autowired](#spring_autowired)

4. [Spring Bean Life Cycle, @PostConstruct và @PreDestroy](#springboot_2)

   4.1. [@PostConstruct](#spring_postconstruct)

   4.2. [@PreDestroy](#spring_predestroy)

   4.3. [Bean Life Cycle](#spring_beanlifecycle)

5. [Spring boot @Component, @Service và @Repository](#springboot_3)

   5.1. [Kiến trúc trong spring boot](#springboot_architecture)

   5.2. [@Controller vs @Service vs @Repository](#springboot_csr)

6. [Component scan](#springboot_componentscan)


------------------


# Learn Spring Boot Framwork

## Các khái niệm cần nắm <a name="java_other"></a>

### Khái niệm tight-coupling (liên kết ràng buộc) và cách loosely coupled (liên kết lỏng lẻo)

`tight-coupling` hay "liên kết ràng buộc" là một khái niệm trong Java ám chỉ việc mối quan hệ giữa các Class quá chặt chẽ. Khi yêu cầu thay đổi logic hay một class bị lỗi sẽ dẫn tới ảnh hưởng tới toàn bộ các Class khác.

`loosely-coupled` là cách ám chỉ việc làm giảm bớt sự phụ thuộc giữa các Class với nhau.

#### Ví dụ

1. Các code level 1

```java
public class BubbleSortAlgorithm{
   public void sort(int[] array) {
      // TODO: Add your logic here
      System.out.println("Đã sắp xếp bằng thuật toán sx nổi bọt");
   }
}

public class VeryComplexService {
   private BubbleSortAlgorithm bubbleSortAlgorithm = new BubbleSortAlgorithm();

   public VeryComplexService(){
   }

   public void complexBusiness(int array[]){
      bubbleSortAlgorithm.sort(array);
      // TODO: more logic here
   }
}
```

Với cách làm ở trên, `VeryComplexService` đã hoàn thiện được nhiệm vụ, tuy nhiên, khi có yêu cầu thay đổi thuật toán sắp xếp sang QuickSort thì nghe vẻ chúng ta sẽ phải sửa lại hoàn toàn cả 2 Class trên.

Ngoài ra `BubbleSortAlgorithm` sẽ chỉ tồn tại nếu `VeryComplexService` tồn tại, vì `VeryComplexService` tạo đối tượng `BubbleSortAlgorithm` bên trong nó (hay nói cách khác là sự sống chết của `BubbleSortAlgorithm` sẽ do `VeryComplexService` quyết định), theo như cách implement này, nó là liên kết rất chặt với nhau.

2. Cách code level 2

```java
public interface SortAlgorithm {
    /**
     * Sắp xếp mảng đầu vào
     * @param array
     */
    public void sort(int array[]);
}

public class BubbleSortAlgorithm implements SortAlgorithm{

    @Override
    public void sort(int[] array) {
        // TODO: Add your logic here
        System.out.println("Đã sắp xếp bằng thuật toán sx nổi bọt");
    }
}


public class VeryComplexService {
    private SortAlgorithm sortAlgorithm;
    public VeryComplexService(){
        sortAlgorithm = new BubbleSortAlgorithm();
    }

    public void complexBusiness(int array[]){
        sortAlgorithm.sort(array);
        // TODO: more logic here
    }
}
```

Với cách làm này, `VeryComplexService` sẽ chỉ quan hệ với một interface `SortAlgorithm`. Với cách này thì mỗi quan hệ giảm bớt sự liên kết, nhưng nó không thay đổi được việc thuật toán vẫn đang là `BubbleSortAlgorithm`.

3. Cách code level 3

```java
public interface SortAlgorithm {
   /**
   * Sắp xếp mảng đầu vào
   * @param array
   */
   public void sort(int array[]);
}

public class BubbleSortAlgorithm implements SortAlgorithm{

   @Override
   public void sort(int[] array) {
      // TODO: Add your logic here
      System.out.println("Đã sắp xếp bằng thuật toán sx nổi bọt");
   }
}

public class QuicksortAlgorithm implements SortAlgorithm {
   @Override
   public void sort(int[] array) {
      // TODO: Add your logic here
      System.out.println("Đã sắp xếp bằng thuật sx nhanh");
   }
}

public class VeryComplexService {
   private SortAlgorithm sortAlgorithm;
   public VeryComplexService(SortAlgorithm sortAlgorithm){
      this.sortAlgorithm = sortAlgorithm;
   }

   public void complexBusiness(int array[]){
      sortAlgorithm.sort(array);
      // TODO: more logic here
   }
}

public static void main(String[] args) {
   SortAlgorithm bubbleSortAlgorithm = new BubbleSortAlgorithm();
   SortAlgorithm quickSortAlgorithm = new QuicksortAlgorithm();
   VeryComplexService business1 = new VeryComplexService(bubbleSortAlgorithm);
   VeryComplexService business2 = new VeryComplexService(quickSortAlgorithm);
}
```

Cách thứ ba này cũng là cách làm phổ biển nhất. Mối liên hệ giữa 2 Class đã "lỏng lẻo" hơn trước rất nhiều. `VeryComplexService` sẽ không quan tâm tới việc thuật toán sắp xép là gì nữa, mà chỉ cần tập trung vào nghiệp vụ. Còn `SortAlgorithm` sẽ được đưa vào từ bên ngoài tùy theo nhu cầu sử dụng.

### Tiêm phụ thuộc (dependency injection) và IoC (Inversion of Control) 

1. DI

> Các class không nên phụ thuộc vào các kế thừa cấp thấp mà nên phụ thuộc vào class abstraction.

`Dependency Injection` là việc các Object nên phụ thuộc vào các Abstract Class và thể hiện chi tiết của nó sẽ được Inject vào đối tượng lúc runtime.

Các cách để Inject dependency vào một đối tượng có thể kể đến như sau:

- Constructor Injection: Cái này chính là ví dụ của mình, tiêm dependency ngay vào Contructor cho tiện.

- Setter Injection: Ồ, sao không chứ 😗 chúng ta học về Setter từ những bài học vỡ lòng rồi, quá hợp lý. Xài girl.setOutfit(new Naked()) 😈

- Interface Injection: Mỗi Class muốn inject cái gì, thì phải implement một Interface có chứa một hàm inject(xx) (Gần như thay thế cho setter ý bạn). Rồi bạn muốn inject gì đó thì gọi cái hàm inject(xx) ra. Cách này hơi dài và khó cho người mới.

2. IoC

> Inversion of Control is a programming principle. flow of control within the application is not controlled by the application itself, but rather by the underlying framework.

Ta định nghĩa trước toàn bộ các `dependency` có trong Project, mô tả nó và tống nó vào 1 cái kho và giao cho một thằng tên là `framework` quản lý. Bất kỳ các `Class` nào khi khởi tạo, nó cần `dependency` gì, thì cái `framework` này sẽ tự tìm trong kho rồi `inject` vào đối tượng thay chúng ta.

## Cách cài đặt <a name="spring_install"></a>

1. Tạo một Spring boot project 

Vào https://start.spring.io/ tạo 1 project . Ở đây sử dụng `maven` và package là `com.example.blog`

Nếu trong `Java` truyền thống, khi chạy cả một project, chúng ta sẽ phải định nghĩa một hàm `main()` và để nó khởi chạy đầu tiên.

Thì **Spring Boot** cũng vậy, chúng ta sẽ phải chỉ cho **Spring Boot** biết nơi nó khởi chạy lần đầu, để nó cài đặt mọi thứ.

Cách thực hiện là thêm annotation `@SpringBootApplication` trên class chính và gọi `SpringApplication.run(BlogApplication.class, args);` để chạy project.

```java
package com.example.blog;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class BlogApplication {

	public static void main(String[] args) {
		SpringApplication.run(BlogApplication.class, args);
	}
}
```

Một trong những nhiệm vụ chính của Spring là tạo ra một cái Container chứa các Dependency cho chúng ta.

`SpringApplication.run(BlogApplication.class, args);` chính là câu lệnh *để tạo ra* **container**. Sau đó nó *tìm toàn bộ* các **dependency** trong project của bạn và đưa vào đó.

Spring đặt tên cho **container** là ***ApplicationContext***

và đặt tên cho các **dependency** là ***Bean***


## Hướng dẫn @Component và @Autowired <a name="springboot_1"></a>

`@Component` và `@Autowire` là 2 annotation (chú thích) cơ bản trong Spring boot.

### @Component là một Annotation đánh dấu trên các `class` để giúp **Spring** biết đó là một `bean`. <a name="spring_component"></a>

Ví dụ:

Ta có một interface `outfit`

```java
public interface Outfit {
   public void wear();
}
```

implement nó là Class `Bikini`

```java
/*
 Đánh dấu class bằng @Component
 Class này sẽ được Spring Boot hiểu là một Bean (hoặc dependency)
 Và sẽ được Spring Boot quản lý
*/
@Component
public class Bikini implements Outfit {
   @Override
   public void wear() {
      System.out.println("Mặc bikini");
   }
}
```

Và chạy chương trình
```java
@SpringBootApplication
public class App {
   public static void main(String[] args) {
      // ApplicationContext chính là container, chứa toàn bộ các Bean
      ApplicationContext context = SpringApplication.run(App.class, args);

      // Khi chạy xong, lúc này context sẽ chứa các Bean có đánh
      // dấu @Component.

      // Lấy Bean ra bằng cách
      Outfit outfit = context.getBean(Outfit.class);

      // In ra để xem thử nó là gì
      System.out.println("Instance: " + outfit);
      // xài hàm wear()
      outfit.wear();
   }
}
```

Bạn sẽ thấy `Outfit` lúc này chính là `Bikini`. Class đã được đánh dấu là `@Component`.

**Spring Boot** khi chạy sẽ dò tìm toàn bộ các *Class* cùng cấp hoặc ở trong các *package* thấp hơn so với class `BlogApplication` mà bạn cung cấp cho Spring (Chúng ta có thể cấu hình việc tìm kiếm này, sẽ đề cập sau). 
Trong quá trình dò tìm này, khi gặp một *class* được đánh dấu `@Component` thì nó sẽ tạo ra một *instance* và đưa vào `ApplicationContext` để quản lý.

### @Autowired <a name="spring_autowired"></a>

Bây giờ mình tạo ra một Class `Girl` và có một thuộc tính là `Outfit`.

Mình cũng đánh dấu `Girl` là một `@Component`. Tức **Spring Boot** cần tạo ra một *instance* của `Girl` để quản lý.

```java
@Component
public class Girl {

   @Autowired
   Outfit outfit;

   public Girl(Outfit outfit) {
      this.outfit = outfit;
   }
   
   // GET 
   // SET
}
```

Tôi đánh dấu thuộc tính `Outfit` của `Girl` bởi Annotation `@Autowired`. Điều này nói với **Spring Boot** hãy tự *inject (tiêm)* một instance của `Outfit` vào thuộc tính này khi khởi tạo `Girl`.

Và chạy chương trình

```java
@SpringBootApplication
public class App {
   public static void main(String[] args) {
      // ApplicationContext chính là container, chứa toàn bộ các Bean
      ApplicationContext context = SpringApplication.run(App.class, args);

      // Khi chạy xong, lúc này context sẽ chứa các Bean có đánh
      // dấu @Component.

      // Lấy Bean ra bằng cách
      Outfit outfit = context.getBean(Outfit.class);

      // In ra để xem thử nó là gì
      System.out.println("Output Instance: " + outfit);
      // xài hàm wear()
      outfit.wear();

      Girl girl = context.getBean(Girl.class);

      System.out.println("Girl Instance: " + girl);

      System.out.println("Girl Outfit: " + girl.outfit);

      girl.outfit.wear();
   }
}
```

**Spring Boot** đã tự tạo ra một `Girl` và trong quá trình tạo ra đó, nó truyền `Outfit` vào làm thuộc tính.

### Singleton

Điều đặc biệt là các `Bean` được quản lý bên trong `ApplicationContext` đều là ***singleton***. Bạn chắc đã để ý điều này từ các *Output* ở phía trên.

```
Instance: com.example.blog.Bikini@54336c81

Girl Outfit: com.example.blog.Bikini@54336c81
```

`Outfit` ở 2 đối tượng trên là một.

Tất cả những `Bean` được quản lý trong `ApplicationContext` đều chỉ được tạo ra **một lần duy nhất** và khi có `Class` yêu cầu `@Autowired` thì nó sẽ lấy đối tượng có sẵn trong `ApplicationContext` để inject vào.

Trong trường hợp bạn muốn mỗi lần sử dụng là một instance hoàn toàn mới. Thì hãy đánh dấu `@Component` đó bằng `@Scope("prototype")`

```java
@Component
@Scope("prototype")
public class Bikini implements Outfit {
   @Override
   public void wear() {
      System.out.println("Mặc bikini");
   }
}
```

## Spring Bean Life Cycle, @PostConstruct và @PreDestroy <a name="springboot_2"></a>

### @PostConstruct <a name="spring_postconstruct"></a>

`@PostConstruct` được đánh dấu trên một method duy nhất bên trong `Bean`. `IoC Container` hoặc `ApplicationContext` sẽ gọi hàm này **sau khi** một `Bean` được tạo ra và quản lý.

```java
@Component
public class Girl {

   @PostConstruct
   public void postConstruct(){
      System.out.println("\t>> Đối tượng Girl sau khi khởi tạo xong sẽ chạy hàm này");
   }
}
```

### @PreDestroy <a name="spring_predestroy"></a>

`@PreDestroy` được đánh dấu trên một method duy nhất bên trong `Bean`. `IoC Container` hoặc `ApplicationContext` sẽ gọi hàm này **trước khi** một `Bean` bị xóa hoặc không được quản lý nữa.

```java
@Component
public class Girl {

   @PreDestroy
   public void preDestroy(){
      System.out.println("\t>> Đối tượng Girl trước khi bị destroy thì chạy hàm này");
   }
}
```

### Bean Life Cycle <a name="spring_beanlifecycle"></a>

**Spring Boot** từ thời điểm chạy lần đầu tới khi *shutdown* thì các `Bean` nó quản lý sẽ có một vòng đời được biểu diễn như ảnh dưới đây:

![](./img/spring-bean-life-cycle.jpg)

Cần hiểu như sau:

1. Khi `IoC Container (ApplicationContext)` tìm thấy 1 `Bean` cần quản lý, nó sẽ khởi tạo bằng `constructor`

2. *Inject dependencies* vào `Bean` bằng Setter, và thực hiện các quá trình cài đặt khác vào `Bean` như `setBeanName`, `setBeanClassLoader`, v.v..

3. Gọi hàm `PostConstruct`

4. Tiền xử lý sau khi `PostConstruct` được gọi

5. `Bean` sẵn sàng để hoạt động

6. Nếu `IoC Container` không quản lý `Bean` nữa hoặc bị shutdown nó sẽ gọi hàm `@PreDestroy` trong `Bean`

7. Xóa `Bean`

### Ví dụ

Thêm `@PostConstruct` và `@PreDestroy` vào class `Girl` trong chương trước.

```java
import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

@Component
public class Girl {

   @PostConstruct
   public void postConstruct(){
      System.out.println("\t>> Đối tượng Girl sau khi khởi tạo xong sẽ chạy hàm này");
   }

   @PreDestroy
   public void preDestroy(){
      System.out.println("\t>> Đối tượng Girl trước khi bị destroy thì chạy hàm này");
   }
}
```

và kết quả 

```
> Trước khi IoC Container được khởi tạo
> Trước khi IoC Container được khởi tạo
	>> Đối tượng Girl sau khi khởi tạo xong sẽ chạy hàm này
> Sau khi IoC Container được khởi tạo
> Trước khi IoC Container destroy Girl
	>> Đối tượng Girl trước khi bị destroy thì chạy hàm này
> Sau khi IoC Container destroy Girl
```

Bạn sẽ thấy dòng *"Trước khi IoC Container được khởi tạo"* được chạy 2 lần.

> Điều này xảy ra bởi vì hàm `App.main(args)` được chạy 2 lần!

Lần đầu là do chúng ta chạy.

Lần thứ hai là do **Spring Boot** chạy sau khi nó được gọi `SpringApplication.run(App.class, args)`. Đây là lúc mà **IoC Container** (`ApplicationContext`) được tạo ra và đi tìm `Bean`.

## @Component vs @Service vs @Repository <a name="springboot_3"></a>

### Kiến trúc trong spring boot <a name="springboot_architecture"></a>

Kiến trúc MVC trong Spring Boot được xây dựng dựa trên tư tưởng "độc lập" kết hợp với các nguyên lý thiết kế hướng đối tượng (một đại diện tiêu biểu là Dependency Inversion). 

> Độc lập ở đây ám chỉ việc các layer phục vụ các mục đích nhất định, khi muốn thực hiện một công việc ngoài phạm vi thì sẽ đưa công việc xuống các layer thấp hơn.

Kiến trúc Controller-Service - Repository chia project thành 3 lớp:

![](./img/controller-service-repository.png)

- **Consumer Layer hay Controller**: là tầng giao tiếp với bên ngoài và handler các request từ bên ngoài tới hệ thống.

- **Service Layer**: Thực hiện các nghiệp vụ và xử lý logic

- **Repository Layer**: Chịu trách nhiệm giao tiếp với các DB, thiết bị lưu trữ, xử lý query và trả về các kiểu dữ liệu mà tầng Service yêu cầu.

### @Controller vs @Service vs @Repository <a name="springboot_csr"></a>

Để phục vụ cho kiến trúc ở trên, **Spring Boot** tạo ra 3 Annotation là `@Controller`,  `@Service` và `@Repository` để chúng ta có thể đánh dấu các tầng với nhau.

- `@Service` Đánh dấu một Class là tầng `Service`, phục vụ các logic nghiệp vụ.

- `@Repository` Đánh dấu một Class Là tầng `Repository`, phục vụ truy xuất dữ liệu.

### Ví dụ

Đầu tiên ta làm 1 class `Girl`

```java
public class Girl {
   private String name;

   public Girl(String name) {
      this.name = name;
   }

   public String getName() {
      return name;
   }

   public void setName(String name) {
      this.name = name;
   }

   @Override
   public String toString() {
      return "Girl(" + this.name + ")";
   }
}
```

Tạo ra một interface `GirlRepository` để giao tiếp với DB.

```java
public interface GirlRepository {
   /**
   * Tìm kiếm một cô gái trong database theo tên
   * @param name
   * @return
   */
   Girl getGirlByName(String name);
}
```

Kế thừa `GirlRepository` và đánh dấu nó là `@Repository`

```java
@Repository
public class GirlRepositoryImpl implements GirlRepository {

   @Override
   public Girl getGirlByName(String name) {
      // Ở đây tôi ví dụ là database đã trả về
      // một cô gái với tên đúng như tham số

      // Còn thực tế phải query trong csđl nhé.
      return new Girl(name);
   }
}
```

Tạo ra một class `GỉrlService` để giải quyết các logic nghiệp vụ. Lớp `GirlService` sẽ giao tiếp với DB thông qua `GirlRepository`.

```java
import org.apache.commons.lang3.RandomStringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class GirlService {
   @Autowired
   private GirlRepository girlRepository;

   public Girl getRandomGirl(){
      // Random 1 cái tên độ dài 10
      String name = randomGirlName(10);

      // Gọi xuông tầng repository để query lấy một cô gái tên là "name" trong database
      return girlRepository.getGirlByName(name);
   }

   public String randomGirlName(int length) {
      // Random một string có độ dài quy định
      // Sử dụng thư viện Apache Common Lang
      return RandomStringUtils.randomAlphanumeric(length).toLowerCase();
   }
}
```

và 

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;

@SpringBootApplication
public class App {

   public static void main(String[] args) {
      ApplicationContext context = SpringApplication.run(App.class, args);

      // Lấy ra bean GirlService
      GirlService girlService = context.getBean(GirlService.class);
      // Lấu ra random một cô gái từ tầng service
      Girl girl = girlService.getRandomGirl();
      // In ra màn hình
      System.out.println(girl);

   }
}
```

Về bản chất `@Service` và `@Repository` cũng chính là `@Component`. Nhưng đặt tên khác nhau để giúp chúng ta phân biệt các tầng với nhau.

Trong các bài đầu tiên chúng ta đã biết `@Component` đánh dấu cho **Spring Boot** biết Class đó là `Bean`. Và hiển nhiên `@Service` và `@Repository` cũng vậy. Vì thế ở ví dụ trên chúng ta có thể lấy `GirlService` từ `ApplicationContext`.

Về bản chất thì bạn có thể sử dụng thay thế 3 Annotation `@Component`, `@Service` và `@Repository` cho nhau mà không ảnh hưởng gì tới code của bạn cả. Nó vẫn sẽ hoạt động.

Tuy nhiên từ góc độ thiết kế thì chúng ta cần *phân rõ* 3 Annotation này cho các Class đảm nhiệm đúng nhiệm vụ của nó.

- `@Service` gắn cho các Bean đảm nhiệm *xử lý logic*
- `@Repository` gắn cho các Bean đảm nhiệm *giao tiếp với DB*
- `@Component` gắn cho các Bean khác.

## Component Scan <a name="springboot_componentscan></a>

Là cách **Spring Boot** tìm kiếm `Bean` trong project.

**Spring Boot** khi chạy sẽ dò tìm toàn bộ các `Class` cùng cấp hoặc ở trong các `package` thấp hơn và tạo ra `Bean` từ các `Class` tìm thấy.

Có 2 cách để tùy chỉnh cấu hình **Spring Boot** chỉ tìm kiếm các `Bean` trong một package nhất định:

1. `@ComponentScan`.

2. Sử dụng `scanBasePackages` tromg `@SpringBootApplication`.

### Sử dụng `@ComponentScan`

```java
@ComponentScan("com.example.blog.other")
@SpringBootApplication
public class App {
   public static void main(String[] args) {
      ApplicationContext context = SpringApplication.run(App.class, args);
      try {
         Girl girl = context.getBean(Girl.class);
         System.out.println("Bean: " + girl.toString());
      } catch (Exception e) {
         System.out.println("Bean Girl không tồn tại");
      }

      try {
         OtherGirl otherGirl = context.getBean(OtherGirl.class);
         if (otherGirl != null) {
               System.out.println("Bean: " + otherGirl.toString());
         }
      } catch (Exception e) {
         System.out.println("Bean Girl không tồn tại");
      }
   }
}
```

### Sử dụng `scanBasePackages`

```java
@SpringBootApplication(scanBasePackages = "com.example.blog.other")
public class App {
   public static void main(String[] args) {
      ApplicationContext context = SpringApplication.run(App.class, args);
      try {
         Girl girl = context.getBean(Girl.class);
         System.out.println("Bean: " + girl.toString());
      } catch (Exception e) {
         System.out.println("Bean Girl không tồn tại");
      }

      try {
         OtherGirl otherGirl = context.getBean(OtherGirl.class);
         if (otherGirl != null) {
               System.out.println("Bean: " + otherGirl.toString());
         }
      } catch (Exception e) {
         System.out.println("Bean Girl không tồn tại");
      }
   }
}
```

Theo như 2 ví dụ này, ta chỉ tìm các `Bean` trong `com.example.blog.other` nên kết quả sẽ như thế này:

```
Bean Girl không tồn tại
Bean: OtherGirl.java
```

> Có thể cấu hình để tìm `Bean` ở nhiều package khác nhau 

```java
@ComponentScan({"com.example.blog","com.example.blog.other"})
// hoặc
@SpringBootApplication(scanBasePackages = {"com.example.blog", "com.example.blog.other"})
```
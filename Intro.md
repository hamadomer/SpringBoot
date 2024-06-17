## How we can configure a bean :

we can use `@Configuration` declare class as "full" configuration class
>Class must be non-final and public.

`@Bean` declares a bean configuration inside the class that has `@Configuration` annotation.
>Method must be non-final and non private.


Here is an example :
```java
@Configuration
public class AppConfig {
    @Bean
      public PaymentServic paymentService ( 
      AccountRepository accountRepository
      ) {
        retun new PaymentServiceImpl(accountRepository);
      }
}
```

So as we can see `@Bean` go inside a `@Configuration` class, that is public and non-final.

This is a full example :
```java
@Configuration
public class AppConfig {
    @Bean
      public PaymentServic paymentService ( 
      AccountRepository accountRepository
      ) {
        retun new PaymentServiceImpl(accountRepository);
      }
    @Bean
      public AccountRespository accountRespository() {
	    return new JdbcAccountRepository(dataSource());
      }
    @Bean
     public DataSource dataSource() {
         return (...);
     }
}
```

as we can see we can declare multiple `@Beans` inside one `Configuration` class.
We have some explaining here :
- `dataSource()` beans which can be used to connect to DB like PostgreSQL, it will return an object of type `DataSource`.
- `accountRespository()` beans which return a new instance of `JdbcAccountRepository()` who takes the return of the `dataSoruce()` as it parameter so it becomes `JdbcAccountRepository(dataSource())`
- `paymentService()`beans that will return a new instance of `PaymentServiceImpl(accountRepository)`;

So for now what i understand is :

- `@Beans` are just like methods, they are under a class that has the annotation `@Configuration`
- Seem like we declare ``@Beans`` as SR, and the depends one each others, Spring will manage the injection of each part


## Spring Component sample :

- Spring Component contains class-level annotation that marks class as a Spring Component `@Component`.
-  Constructor-dependency injection is automatically done using `@Autowird` by injection the constructor parameter(s).
- `Autowired` on Constructor is optional if there is only one constructor.

Example :

```java
@Component
public class PaymentServiceImpl {
  private final AccountRepository accountRepository;

  @Autowired
   public PaymentServiceImpl (
     AccountRepository accountRepository
   ) {
     this.accountRepository = accountRepository;
   }
}
```

Spring Components :
- Spring provides component stereotype to classify classes as Spring Components
    - Sub-types are available as a refinement for the standard components.
- @Component as a general component annotation indicating that the class should be initialized, configured and managed by the core container
- @Repository, @Service and @Controller as meta-annotation for @Component that allows to further re-fine components.
- Own stereotype annotation can (and should) be defined to support general architecture principles.


let's take look at this: 

```java
@Configuration
public class AppConfig {
    @Bean
      public PaymentServic paymentService ( 
      AccountRepository accountRepository
      ) {
        retun new PaymentServiceImpl(accountRepository);
      }
    @Bean
      public AccountRespository accountRespository() {
	    return new JdbcAccountRepository(dataSource());
      }
    @Bean("ds")
     public DataSource dataSource() {
         return (...);
     }
}
```

==If we don't provide a bean name like "ds" spring will give it the method name by default==.

## Dependency injection "Beans injection" :
1. **Constructor Injection** 
2. **Field Injection
3. **Configuration Methods**
4. **Setter Methods injection**

## Constructor Injection :

```java
@Service
public class DefaultPaymentService {

  private final AccountRepository accountRepository;

  public DefaultPaymentService (
    AccountRepository accountRepository
  ) {
     this.accountRepository = accountRepository;
  }
}
```

- Using @Qualifier :

Example :

```java
@Configuration
public class ApplicationConfig {

   @Bean
   @Qualifier("primary")
   public AccountRepository primary () {
      retun new JdbcAccountRepository(...);
   }
   @Bean
   @Qualififer("secondary")
   public AccountRepository secondary () {
      return new JdbcAccountRepository(...);
   }
}
```

```java
@Service
public class DefaultPaymentService {
   @Autowired
   public DefaultPaymentService (@Qualifier("primary") AccountRepository accountRepository) {
   this.accountRepository = accountRepository;
   }
}
```

Or we can use `@Primary` directly like this :

```java
@Configuration
public class ApplicationConfig {

   @Bean
   @Primary
   public AccountRepository primary () {
      retun new JdbcAccountRepository(...);
   }
   @Bean
   public AccountRepository secondary () {
      return new JdbcAccountRepository(...);
   }
}
```

```java
@Service
public class DefaultPaymentService {
   @Autowired
   public DefaultPaymentService (AccountRepository accountRepository) {
   this.accountRepository = accountRepository;
   }
}
```


> [!faq]- What if we need to choose another bean?
>  we can we just need to specifay the Quialifier in this case, if not it will take primary as default bean.

## Field Injection :

```java
@Service
public class DefaultPaymentService {

   @Autowired
   private AccountRepository accountRepository;
}
```

> [!tip] Discouraged :
> Makes testing of components in isolation more complex, therefore should only be used in test classes.

## Method Injection : 

- Method injection allows setting one or many dependencies by one method
- Allows for initialization work if needed while receiving dependencies 

```java
public class DefaultPaymentService {
  @Autowired
  public void configureClass (
       AccountRepository accountRepository,
       FeeCaclulator feeCaclulator
  ) {
     // ...
  }
}
```

> [!attention]
> This code assume that accountRepository and feeCaclulator are `@Beans`.


## Setter Injection :

Setter injection follows Java bean naming convention to inject dependency

```java
@Service
public class DefaultPaymentService {
  @Autowired
  public void setAccountRepository (
    AccountRepository accountRepository
  ) {
     // ...
  }
}
```



### Notes from the project :

- if we want to use a bean we need to look for the class using context : 
```java 
     var ctx = SpringApplication.run(Application.class, args);
     
     MyFirstClass myFirstClass = ctx.getBean(MyFirstClass.class);

```
We can see the the variable `ctx` hold run method that comes from SpringApplication object , after that we use this variable with now has it own methods `getBean` which can be used to search for a bean, in this case my first class bean.

> [!note]
> We need to declare the class as `@Component` or one it's children `@Service` and `@Repository` annotation so the context can recognize it as a ``@Bean`.


![](/Attachements/1.png)
in this code we get `NullPointerException` throw because the `myFirstService` is declared as `MyFirstService` type but not initialized so the variable is pointing to null .

![](/Attachements/2.png)

To solve this problem we need to inject a constructer, and add the annotation `@Autowired` so Spring boot know that it should look for a beans that has the same type as a return value. 

![](/Attachements/3.png)

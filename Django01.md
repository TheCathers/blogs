# Django 01 -- 初识Django

## Django简介

相比于许多Javaer第一次接触Django, 因为此前有过一段另一个Python后端框架（odoo）的实习经历，Django并没有给我带来很多
不可思议的开发体验，相反同为Active Record模式的框架非常熟悉。 

## Django ORM

所谓Active Record模式，书面的说法是指数据库中的每个表都对应着一个Python类，而表中的每一行数据则对应着该类的一个实例对象。
并且对象实例包含了对该行数据进行 CRUD 操作的方法。
```python
# Django ORM 是 Active Record 的典型代表
from django.db import models

class User(models.Model):
    name = models.CharField(max_length=100)
    email = models.CharField(max_length=100)
    
    def get_display_name(self):
        return f"{self.name} ({self.email})"

# 使用方式：对象包含数据和行为
user = User(name="张三", email="zhangsan@example.com")
user.save() 

user = User.objects.get(id=1)
user.delete()
```
每一个初次接触Active Record的Javaer应该都能感受到它的优势，这样太简洁了！  
相较于Java主流的Data Mapper模式，实现上述同样逻辑的代码大概需要这样写：
首先是需要pojo类：
```java
@Entity
@Table(name = "users") // 对应数据库表名
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "name", length = 100)
    private String name;
    
    @Column(name = "email", length = 100)
    private String email;
    
    // 默认构造函数 (JPA 要求)
    public User() {}
    
    // 带参构造函数
    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }
```
然后是Mapper和Service：
```java
@Repository
public interface UserMapper extends BaseMapper<User> {
    // 自定义查询方法可以直接在接口中定义
    @Select("SELECT * FROM users WHERE name = #{name}")
    User findByName(String name);
}
```
```java
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;
    
    public User getUserByName(String name) {
        return userMapper.findByName(name);
    }
}
```
如果不使用Lombok或者mybatis-plus等工具，那么还需要写大量的get/set方法和sql语句。
量码量简介、用箱即用、约定大于配置、渐进式复杂性就是Django的设计哲学。   
Django鼓励开发者使用ORM来进行数据库操作，而不是直接编写SQL。ORM提供了高级的抽象，使得代码更易于编写、阅读和维护，并且具有较好的可移植性（更换数据库后端时，ORM可以生成相应的SQL）。

那么代价是什么？
- 首先要主要的肯定是性能问题，ORM的性能通常低于直接编写SQL的性能。
- 同步阻塞，当然这也不能算是Active Record的问题，只是放在这里一起提一下。
Django最初是为同步请求设计的。虽然现在支持异步，但整个异步生态还不像同步那样成熟，而且很多第三方库可能仍然是同步的。
- 灵活性受限：Django的“约定优于配置”意味着如果你按照它的方式做事，会非常顺利。
但当你需要做一些不符合Django约定的事情时，可能会遇到困难。例如，使用非关系型数据库（如MongoDB）作为主要数据库时，Django的ORM支持不够好，需要额外的努力。
- 其他：可读性我个人感知不算明显，因为我认为大段的sql实际上可读性也很差；还有内存开销，这个和Java谁也别说谁
模板系统确实也不够强大，但现在大多也都是前后端分离的。  

尽管有这些代价，但Django的哲学和设计选择在大多数**适合的Web应用开发场景**下带来的好处远远超过这些代价。  
而且，Django社区非常活跃，提供了大量的第三方库和解决方案来缓解这些问题。  
在极特定场景（极致性能、特殊架构需求）下，这些代价才可能成为问题。
title: 使用springboot+jpa快速搭建无xml配置web应用
date: 2016-06-12 15:46:16
tags: ["springboot","jpa"]
categories: ["springboot"]
---
最近一个小工具使用了beego，发现原来java项目手工配置真的是太累了。在接触了Grails后，觉得Grails虽然好用，但是并不完全顺手。看了spring4和springboot后，才发现使用springboot可以非常整洁了。同时结合注解可以完全脱离xml的配置（spring的配置和web.xml的配置）。现在从头开始搭建无xml配置web应用。[代码地址](https://github.com/dengzhengfeng/springboot)
<h2 id="context">准备环境</h2>
>项目管理使用gradle 开发工具使用sts 数据库使用postgresql
<h2 id="gradle">初始化项目</h2>
使用gradle创建一个java项目
`gradle init --type java-library`
<h2 id="ide">导入IDE</h2>
>导入IDE，不同的IDE可能会需要不同的插件，比如sts需要gradle的插件
<h2 id="dependency">配置依赖</h2>
>配置build.gradle
``` +grovvy
buildscript {
    repositories {
        jcenter()
        maven { url "http://repo.spring.io/snapshot" }
        maven { url "http://repo.spring.io/milestone" }
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.3.5.RELEASE")
    }
}
apply plugin: 'java'
apply plugin: 'spring-boot'

jar {
    baseName = 'boot'
    version =  '0.0.1-SNAPSHOT'
}

repositories {
    jcenter()
    maven { url "http://repo.spring.io/snapshot" }
    maven { url "http://repo.spring.io/milestone" }
}

dependencies {
	compile("org.springframework.boot:spring-boot-starter-web")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
//入口类，需要编写对应类
springBoot {
    mainClass = "org.dzf.Application"
}
```
<h2 id="hello">Hello World！</h2>
>官方包路径为domain service web

我们按照官方建议建立包路径
-org.dzf
————————domain
————————service
————————web
>编写Controller类

在web包下新建类
``` +java
@Controller
@EnableAutoConfiguration
public class HomeController {
	
	@RequestMapping("/")
	@ResponseBody
	String home(){
		return "Hello World!";
	}
}
```
@Controller 注解标识这个类是一个Controller类
@EnableAutoConfiguration 是让spring自动处理与拼装bean
@RequestMapping("/") 注解是springMVC的标识，处理映射路径
@ResponseBody 标识方法直接返回响应体如json数据，而不是一个view

>编写入口类

在build.gradle中配置的入口类，也是启动项目的入口
``` +java
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```
@SpringBootApplication 标识这个一个程序入口（配置类）相当于@Configuration,@EnableAutoConfiguration和 @ComponentScan

因为spring boot内置了容器，我们直接运行Application，就可以启动默认的8080端口访问到Hello World程序了

<h2 id="config">配置文件与资源文件</h2>
>spring boot配置文件与资源文件都默认从src/main/resoureces获取

静态文件路径src/main/resoureces/static
模板文件路径src/main/resoureces/templates

>配置文件可以使用application.properties或application.yml

我选择的是application.yml的配置文件，同理application.properties也是类似的
新建编辑src/main/resoureces/application.yml文件
如我不想使用默认的8080端口，就在配置文件中新增配置
``` +yml
server: 
  port: 9090
```
重启应用后，端口就可以发现启动端口是9090了
>官方并不建议使用jsp，这里我使用velocity
首先配置依赖
``` +json
compile("org.springframework.boot:spring-boot-starter-velocity")
```
在templates目录下新建文件home.vm
``` +html
Hello Velocity!
```
修改HomeController，删除@ResponseBody注解
重编译项目后，启动后就可以看到首页使用了Velocity对应的模版了

<h2 id="jpa">使用jpa完成数据持久化</h2>
>配置build.gradle 新增jpa的依赖（使用postgresql并建立对应的用户与库）

``` +json
compile("org.springframework.boot:spring-boot-starter-data-jpa")
compile("postgresql:postgresql:9.1-901-1.jdbc4")
```
在配置文件配置（修改对应的配置）
``` +yml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/dzf
    username: dzf
    password: 123456
    # Keep the connection alive if idle for a long time (needed in production)
    test-while-idle: true
    validation-query: SELECT 1
  jpa:
    # Show or not log for each sql query
    show-sql: true
    hibernate:
      # Hibernate ddl auto (create, create-drop, update)
      ddl-auto: create
      # Naming strategy
      naming-strategy: org.hibernate.cfg.ImprovedNamingStrategy
    # Use spring.jpa.properties.* for Hibernate native properties (the prefix is
    # stripped before adding them to the entity manager)
    # The SQL dialect makes Hibernate generate better SQL for the chosen database
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
```
我们只使用了一个数据源，所以只需要覆盖默认的配置就可以了
我们假设一个实体 书（Book）对应表为bookdoc 创建domain类（在domain包下）
``` +java
@Entity
@Table(name = "bookdoc")
public class Book {
	@Id
	@GeneratedValue(generator = "uuid2")
    @GenericGenerator(name = "uuid2", strategy = "uuid2")
	@Type(type="pg-uuid")
	private UUID id;
	@Column(length=64)
	private String name;//书名
	@Column(length=128)
	private String publisher;//出版社
	@Temporal(TemporalType.DATE)
	private Date publisher_year;//出版时间
	@Temporal(TemporalType.DATE)
	private Date createtime;//创建时间
	
	@PrePersist
	protected void onCreate() {
		createtime = new Date();
	}
	
	//这里省略了get set方法
}
```
@Entity 标识这是一个实体类
@Table(name = "bookdoc") 标识对应的表
@Id 标识主键字段
@Type(type="pg-uuid") 标识字段类型，这里我使用的是postgresql的uuid类型
@Column 标识字段
@GeneratedValue(generator = "uuid2") @GenericGenerator(name = "uuid2", strategy = "uuid2") 标识自动创建uuid
@PrePersist 标识在创建时自动创建动作，这里会在插入行时自动插入创建时间

创建Repository接口
``` +java
@Repository(value="bookRepository")
public interface BookRepository extends JpaRepository<Book,UUID>{
    @Query("select a from Book a where name like %:name%")
	public List<Book> queryBook(@Param("name")String name,Pageable pageable);
}
```
BookRepository可以空申名，因为它继承了JpaRepository，在这之后，我们可以通过spring注入的BookRepository对bookdoc表做简单的操作（如save、findAll），当然这应该是在service里做的，接下来我们完善service，创建BookService接口
``` +java
public interface BookService {
	public void saveBook(Book book);
	public List<Book> findBook(String bookName);
}
```
接口申明了两个方法，保存书和通过书名查找书，接下来我们编写实现类
``` +java
@Service(value = "BookService")
public class BookServiceImpl implements BookService{
	@Resource(name = "bookRepository")
	private BookRepository bookRepository;

	@Override
	public void saveBook(Book book) {
		bookRepository.save(book);
	}

	@Override
	public Book findBook(String bookName) {
		Pageable pageable = new PageRequest(0,5);
		return bookRepository.queryBook(bookName,pageable);
	}
	
}
```
@Service 标识实现类，spring自动注入BookService时会注入这个实现类
好了，我们在HomeController中调用对应的Service
``` +java
@Autowired
	private BookService bookService;

	@RequestMapping("/")
	String home() {
		try {
			Book book = new Book();
			book.setName("测试书");
			book.setPublisher("测试出版社");
			book.setPublisher_year(new SimpleDateFormat("yyyy-MM-dd").parse("2012-12-12"));
			bookService.saveBook(book);
			
			System.err.println(bookService.findBook("测试书"));
		} catch (ParseException e) {
			e.printStackTrace();
			return "redirect:error";
		}
		return "home";
	}
```
启动项目后，访问主页就可以测试了（查询使用了JPQL，分页只要传入Pageable参数）

<h2 id="mapping">关系映射</h2>
>在这里，比如每一本具体的书只能被一个学生拥有，每个学生可以拥有多本书

在Book类中新增一个属性
``` +java
@ManyToOne
private Student student;
```
>@ManyToOne 标识多对一的多这一端，同理@OneToMany、@OneToOne、@ManyToMany也是同字面上的意思
编写Student类（同时补全StudentRepository、StudentService、StudentServiceImpl类）
``` +java
@Entity
public class Student {
	@Id
	@GeneratedValue(generator = "uuid2")
    @GenericGenerator(name = "uuid2", strategy = "uuid2")
	@Type(type="pg-uuid")
	private UUID id;
	@Column(length=64)
	private String name;
	@OneToMany
	private List<Book> books;
}
```
完成后，我们测试一下，修改HomeController
``` +java
@Controller
@EnableAutoConfiguration
public class HomeController {
	@Autowired
	private BookService bookService;
	
	@Autowired
	private StudentService studentService;

	@RequestMapping("/")
	String home() {
		try {
			Student student = new Student();
			student.setName("小明");
			studentService.saveStudent(student);
			List<Book> books = new ArrayList<Book>();
			
			for (int i = 0; i < 10; i++) {
				Book book = new Book();
				book.setName("测试书" + i);
				book.setPublisher("测试出版社" + i);
				book.setPublisher_year(new SimpleDateFormat("yyyy-MM-dd").parse("2012-12-12"));
				book.setStudent(student);
				bookService.saveBook(book);
				books.add(book);
			}
			
			student.setBooks(books);
			studentService.saveStudent(student);
		} catch (ParseException e) {
			e.printStackTrace();
			return "redirect:error";
		}
		return "home";
	}
}
```
之后启动并访问首页
![关系映射](/img/springboot/onetomany.png)
我们看到了由于有多映射端，生成了中间表，同时在one端生成了外键

<h2 id="transaction">多数据源与事务</h2>
>在这里，我们先假设两个数据源。一个是school，包括了Book和Student。一个是game，包括了Game

首先我们注释掉原来spring:datasource的默认数据源，配置两个新的数据源
``` +json
datasource:
  school:
    url: jdbc:postgresql://localhost:5432/school
    username: dzf
    password: 123456
  game:
    url: jdbc:postgresql://localhost:5432/game
    username: dzf
    password: 123456
```
同时需要实例化两个数据源，新建DataConfiguration类
``` +java
@Configuration
public class DataConfiguration {
	@Bean(name="schoolDataSource")  
	@Primary
    @ConfigurationProperties(prefix="datasource.school")
    public DataSource primaryDataSource() {  
        return DataSourceBuilder.create().build();  
    }
	
	@Bean(name="gameDataSource")
    @ConfigurationProperties(prefix="datasource.game")
    public DataSource gameDataSource() {  
        return DataSourceBuilder.create().build();  
    }
}
```
接下来我们在domain和service中分别新建school与game包（同时补全Game、GameRepository、GameService、GameServiceImpl）
>配置GameConfig和SchoolConfig，以GameConfig为例
``` +java
@Configuration
@EnableJpaRepositories(entityManagerFactoryRef = "gameEntityManagerFactory", transactionManagerRef = "gameTransactionManager")
public class GameConfig {
	@Autowired
	@Qualifier("gameDataSource")
	private DataSource gameDataSource;

	@Bean
	PlatformTransactionManager gameTransactionManager() {
		return new JpaTransactionManager(gameEntityManagerFactory().getObject());
	}

	@Bean
	LocalContainerEntityManagerFactoryBean gameEntityManagerFactory() {

		HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
		vendorAdapter.setGenerateDdl(true);

		LocalContainerEntityManagerFactoryBean factoryBean = new LocalContainerEntityManagerFactoryBean();

		factoryBean.setDataSource(gameDataSource);
		factoryBean.setJpaVendorAdapter(vendorAdapter);
		factoryBean.setPackagesToScan("org.dzf.domain.game");

		return factoryBean;
	}
}
```
@EnableJpaRepositories 注解用于Repositorie的配置，事务由transactionManagerRef属性注入

>事务回滚，假设我们可以保存多个游戏，但是超过三个就认为是不正常的，不可以保存

在GameService增加方法 `saveGameList(List<Game> games);`
在GameServiceImpl实现这个方法
``` +java
@Service(value = "gameService")
public class GameServiceImpl implements GameService{
	@Resource(name = "gameRepository")
	private GameRepository GameRepository;
	
	@Transactional(transactionManager="gameTransactionManager",rollbackFor=RuntimeException.class)
	public void saveGameList(List<Game> games) {
		int i = 0;
		for (Game game : games) {
			GameRepository.save(game);
			i++;
		}
		if(i > 3)
			throw new RuntimeException(); 
	}

}
```
@Transactional 注解声明了方法是被事务管理的，transactionManager需要注入对应的事务管理类，rollbackFor为回滚条件

好了，现在我们测试一下，修改HomeController
``` +java
@RequestMapping("/")
String home() {
	try {
		List<Game> games = new ArrayList<Game>();
		for(int i =0;i<3;i++){
			Game game = new Game();
			game.setName("游戏名" + i);
			games.add(game);
		}
		gameService.saveGameList(games);
		
		games.removeAll(games);
		for(int i =3;i<13;i++){
			Game game = new Game();
			game.setName("游戏名" + i);
			games.add(game);
		}
		gameService.saveGameList(games);
	} catch (Exception e) {
		e.printStackTrace();
		return "redirect:error";
	}
	return "home";
}
```
启动后访问主页，发现主页跳转到出错页，同时game表有插入三行数据
![事务](/img/springboot/transaction.png)

>好了，到目前为止已经能通过spring boot完成多数工作了，web.xml的配置也可以通过@WebServlet、@WebFilter、@WebListener 配置完成。多数情况下，已经可以通过注解脱离xml的配置了

## 21/02/14
### 프로젝트 기본환경 셋팅
* 인텔리제이에서 git remote/commit
* alt+insert 새파일 / ctrl+k 커밋 / ctrl+shift+k 푸시

## 21/02/20
### 단위테스트
* @SpringBootApplication : 스프링부트의 자동 설정, 스프링 Bean읽기와 생성을 모두 자동으로 설정되게 해준다.<br>
  @SpringBootApplication이 있는 위치부터 설정을 읽어가기때문에 __이 클래스는 항상 프로젝트의 최 상단에 위치__해야한다<br>
  SpringApplication.run으로 내장 was를 실행한다
* @RestController : 컨트롤러를 JSON을 반환하는 컨트롤러로 만들어준다
  예전에 @ResponseBody를 각 메소드마다 선언했던것을 한번에 사용할 수 있게 해준다
* @GetMapping : HTTP Method인 Get 요청을 받을 수 있는 API를 만들어준다
* Test클래스에서 `MockMvcRequestBuilders.get` 과 `MockMvcResultMatchers.content/status` 가 계속 import가 되지않았다.. 찾아보니 도구들의 버전 변경으로 수정해야 할 코드들이 많았다. 
  저자 블로그에 나온대로 build.gradle을 고쳤지만 계속 안돼서 아래처럼 고쳤더니 import 됐다!
  
  
__as-is__
```
testImplementation('org.springframework.boot:spring-boot-starter-test')
```
__to-be__ <br>
```
    testImplementation('org.springframework.boot:spring-boot-starter-test'){
        exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    }
    testImplementation('org.springframework.restdocs:spring-restdocs-mockmvc')
    testImplementation('org.junit.jupiter:junit-jupiter-api')
```
   
  * @Runwith => @ExtendWith (SpringRunner => SpringExtension) : 
    - 테스트를 진행할 때 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킨다
    - 여기서는 SpringExtension 이라는 스프링 실행자를 사용한다
    - 즉, 스프링 부트 테스트와 JUnit 사이에 연결자 역할을 한다
  * @WebMvcTest
    - 선언시 @Controller, @CoontrollerAdvice등을 사용할 수 있다
    - 단 @Service, @Conponetnt, @Repository는 사용불가
    - 여기서는 컨트롤러만 사용하기때문에 선언한다
  * @privateMockMvc mvc 
    - 웹 API를 테스트할 때 사용한다
    - 이 클래스를 통해 HTTP GET, POST등에 대한 API테스트를 할 수 있다
  * mvc.perform(get("/hello")) : Mockmvc를 통해 /hello 주소로 HTTP GET 요청을 함, 체이닝이 지원되어 여러 검증 기능을 이어서 선언 할 수 있다
    - .andExpect(status().isOk()) : 
      + mvc.perform의 결과를 검정한다
      + HTTP Header의 Status를 검증한다 (흔히 알고있는 200, 404, 500등의 상태 검증)
      + 여기서는 .isOk 즉 200인지 아닌지를 검증한다
    - .andExpect(content().string(hello))
      + 응답 본문의 내용을 검증한다
      + Controller에서 "hello"를 리턴하기 때문에 이 값이 맞는지 검증한다
      
   ## 21/02/24
   ### Lombok
   * 롬복: Getter, Setter, 기본생성자, toString등을 어노테이션으로 자동생성해준다
   * @Getter : 선언된 모든 필드의 get메소드를 생성해준다
   * @RequiredArgsContructor : 선언된 모든 final필드가 포함된 생성자를 생성해준다(final이 없느 필드생성자는 포함x)<br>
   __Junit5에서 assertThat() 사용하기__
     ```
        import org.assertj.core.api.Assertions; <-- 이걸 import 해야함
        Assertion.assertThat()           
     ```
     안돼서 찾아보면 `hamcrest`를 import하라는데.. import자체가 안됐다..ㅜ
   * @assertThat : 테스트 검증 라이브러리. 검증하고 싶은 대상을 메소드 인자로 받는다
   * @isEqualTo : assertThat에 있는 값과 isEqualTo의 값을 비교해서 같을때만 성공
   * @RequestParam : 외부에서 API로 넘긴 파라미터를 가져오는 어노테이션

        ```
        @RequestParam("name") String name 
       => @RequestParam("name") 이름으로 넘긴 데이터를 String name에 저장한다
       ```
       ```
        mvc.perform(get("/hello/dto")
                    .param("name", name)
                    .param("amount", String.valueOf(amount)))
                    .andExpect(status().isOk())
                    .andExpect( jsonPath("name", is(name)))
                    .andExpect(jsonPath("amount", is(amount)));
       ```
   * param : API테스트 할 때 사용 될 요청 파라미터를 설정한다(값은 String만 가능)
   * JSON 응답값을 필드별로 검증할 수 있는 메소드를 기준으로 필드명을 명시한다
   * is : `import static org.hamcrest.Matchers.is`
   
   ## 21/02/25
   ### JPA
   JPA = 자바 표준 ORM(Object Realational Mapping) : 객체 매핑
   MyBits, iBatis = SQL Mapper : 쿼리 매핑
   RDBMS(관계형 데이터베이스) : 어떻게 데이터를 저장할지 에 초점
   객체지향 프로그래밍언어: 기능과 속성을 한 곳에서 관리
   ==> 패러다임 불일치
   ```
    JAP <- Hibernate <- Spring Data JPA
   ```
   Hibernate를 쓰지 않고 Spring Data JAP를 쓰는 이유
   * 구현체 교체의 용이성 : Hibernate 외에 다른 구현체로 쉽게 교체하기 위함 (Sprign Data JPA에서 구현체 매핑 지원)
   * 저장소 교체의 용이성 : 관계형 데이터메이스 외에 다른 저장소로 쉽게 교체하기 위함 (기본 CRUD 인터페이스가 같기때문에 저장소 교체가 용이하다)
   
   * 이 프로젝트에서 domain : 게시글, 댓글, 회원, 정산, 결제 등 소프트웨어에 대한 요구사항 혹은 문제영역
   ```
    @Getter
    @NoArgsConstructor
    @Entity
    class Posts {
    
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
    
        @Column(length = 500, nullable = false)
        private String title;
    
        @Column(columnDefinition = "TEXT", nullable = false)
        private String content;
    
    
        private String author;
    
        @Builder
        public Posts (String title, String content, String author){
            this.title = title;
            this.content = content;
            this.author = author;
    }
}
   ```
   * Posts 클래스는 실제 DB와 매칭될 클래스이며 Entity클래스라고도 한다. JPA를 사용하면 보통 실제 쿼리를 날리기보다 Entity클래스 수정을 통해 작업한다
   * @Entity : 테이블과 링크 될 클래스임을 나타냄
   * @Id : 해당 테이블의 PK필드를 나타냄
   * @GeneratedValue : PK생성규칙. 스프링부트 2.0에서는 GenerationType.IDENTITY를 추가해야 auto_imcrement가 된다
   * @Column : 테이블의 칼럼을 나타내며 굳이 선언하지않아도 해당 클래스의 필드는 모두 칼럼이 된다.<br>
               기본값 외에 추가로 변경이 필요한 옵션이 있으면 사용함.<br>
               문자열의 경우 VARCHAR(255)가 기본값인데 <br>
                 - 사이즈를 500으로 늘리고싶거나 <br>
                 - 타입을 TEXT로 변경하고싶거나 할 때 사용된다<br>
   * @NoArgsConstructor : 기본생성자 자동 추가
   * @Buider : 해당 클래스의 빌더 패턴 클래스를 생성. 생성자에 포함된 필드만 빌더에 포함
   * Entity클래스에는 __setter 메소드를 만들지 않는다.__ 값 변경이 필요할 경우 그 목적과 의도를 명확히하는 메소드를 추가한다.<br>
     ==> DB에 값을 채우는것은 기본적으로 생성자를 통하여 / 값 변경이 필요한경우는 public 메소드를 호출하여
   * Builder 클래스를 사용하면 어느 필드에 어떤값을 채워야하는지 명확하게 인지할 수 있다.  
   * repository : Dao 역할. 인터페이스로 생성 후 JpaRepository<Entity클래스, PK 타입>을 상속하면 기본적인 CRUD메소드가 생성된다.
    <br> Entity클래스와 기본 Entity Repository는 함께 위치해야한다!
   
   * repositoryTest
        - postRepository.save : 테이블 post에 insert/update 쿼리를 실행한다
        - id값이 있으면 update, 있으면 insert
   
   * 어떻게 쿼리 날리는지 보는 방법: application.properties에 `spring.jpa.show_sql=true` 추가하기

   ## 21/02/26
   ### 등록/수정/조회 API만들기
   API를 만들기 위해서는 총 세개의 클래스가 필요하다
```
    * Request 데이터를 받을 Dto
    * API 요청을 받을 Controller
    * 트랜잭션, 도메인 기능 간의 순서를 보장하는 Service
```
   * service : 비즈니스로직을 처리하는것이 아니다. 트랜잭션, 도메인간 순서 보장의 역할만 한다
   * Dto(Data Transfer Object) : 계층 간에 데이터 교환을 위한 객체
   * Domain Model : 도메인이라 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해할 수 있고 공유할 수 있도록 단순화 시킨 것
     <br>무조건 DB 테이블과 관계있어야하는 것은 아니고 VO처럼 값 객체들도 이영역에 해당한다
     <br>비즈니스 로직 처리
   
   * 생성자를 직접 만들지 않고 @RequiredArgsConstructor(롬복 어노테이션) 사용이유: 클래스의 의존성 관계가 변경될 때마다 생성자 코드를 계속 수정하는 번거로움을 해결할 수 있다
   
   * Entity 클래스와 거의 유사한 형태임에도 Dto클래스를 추가로 생성했다. 절대로 __Entity 클래스를 Request/Response 클래스로 사용해선 안된다!!__
     <br> Entity클래스는 DB와 맞닿은 핵심클래스이다. Entity를 기준으로 테이블이 생성되고, 스키마가 변경되기때문이다!
     <br> View Layer와 DB Layer의 역할 분리를 철저하게 하는 게 좋다!
     
   * JPA테스트토 함께 하는경우네는 @WebMvcTest를 사용하지않고 @SpringBootTest와 TestRestTemplate를 사용한다
   * [TestRestTemplate](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/web/client/TestRestTemplate.html)
     <br>[Static import](https://docs.oracle.com/javase/8/docs/technotes/guides/language/static-import.html) 
     <br>[ResponseEntity](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/ResponseEntity.html)
   
   * update 처리할 때 : 쿼리를 날리는 부분이 없는데 처리된다. 이것은 __JPA의 영속성 컨텍스트__ 때문이다. 
     <br>__영속성 컨텍스트__ : 엔티티를 영구 저장하는 환경.(JPA의 핵심내용은 엔티티가 영속성 컨텍스트에 포함되어 있냐 아니냐로 갈린다)
     <br> JPA의 엔티티 매니저(Spring Data JPA의 기본옵션)가 활성화된 상태로 __트랜잭션 안에서 DB데이터를 가져오면__ 데이터는 영속성 컨텍스트가 유지된 상태이다
     <br> 이 상태에서 해당 데이터의 값을 변경하면 __트랜잭션이 끝나는 시점에 해당 테이블에 변경분을 반영__한다.
     <br> 즉 Entity 객체의 값만 변경하면 별도로 UPDATE 쿼리를 날릴 필요가 없다는 것!!! <-- 더티 체킹(dirty checking)이라고 한다
     
   ### H2 DB에 연결
     책에서 나온대로 접속하면 H2 보안이슈때문에 연결이 안된다. h2버전을 낮춰줌
     <br>build.gradle에서 `com.h2database:h2:1.4.197` 로 설정변경. 근데 접속해도 POSTS 테이블이 안보임!
     <br>그래서 applicaton.properties 에서 아래 설정을 추가해줌
   
   ```
        spring.h2.console.enabled=true
        spring.datasource.hikari.jdbc-url=jdbc:h2:mem:testdb;MODE=MYSQL
        spring.datasource.driverClassName=org.h2.Driver
        spring.datasource.username=sa
        spring.datasource.password=
        spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
   ```
   * 왜인지는 모르겠는데 
    `spring.datasource.url=jdbc:h2:mem:testdb` 이렇게 하면 POSTS 테이블이 안붙고
    <br>`spring.datasource.hikari.jdbc-url=jdbc:h2:mem:testdb;MODE=MYSQL` 이렇게 MYSQL까지 같이 설정해주니까 붙었다
   * 테스트에서 create table을 못하는 이슈발생...
    `spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect`
    <br> 로 변경해줌
    <br> https://github.com/jojoldu/freelec-springboot2-webservice/issues/67 
   ### JPA Auditing으로 생성시간/수정시간/ 자동화하기
   * __Locla Date__ , __LoclaDate Time__ 을 사용
   * 보통 entity에는 해당 데이터의 생성시간과 수정시간을 포함한다. 반복적인 코드가 여러군데 들어가는데 이것을 JPA Auditing를 사용하여 해결!
   * BaseTimeEntity 클래스는 모든 Entity클래스의 상위 클래스가 되어 Entity들의 createDate, modifiedDate를 __자동으로 관리하는 역할을 한다__
   * @MappedSuperclass : JPA Entity 클래스들이 BaseTimeEntity를 상속할 경우 필드들(createDate, modifiedDate)도 칼럼으로 인신하도록한다
   * @EntityListeners(AuditingEntityListener.class) :  BaseTimeEntity클래스에 Auditing기능을 포함시킨다
   * @CreateDate : Entity가 생성되어 저장될 때 시간이 자동 저장된다
   * @LastModifiedDate : 조회한 Entity값을 변경할 때 시간이 자동저장된다
   * JPA Auditing 어노테이션들을 모두 활성화 할 수 있도록 Application클래스에 `@EnableJpaAuditing` 을 추가해준다

   ## 2021/03/09
   ### 머스테치로 화면 구성하기
   * point : 서버 템플릿 엔진과 클라인언트 템플릿 엔진의 차이. JSP가 아니라 머스테치를 이용한 화면개발
    1. 템플릿엔진 : 지정된 템플릿 양식과 데이터가 합쳐져 HTML문서를 출력하는 소프트웨어
    <br>2. 서버 템플릿엔진을 이용한 화면생성은 서버에서 JAVA코드를 문자열로 만든 뒤 이 문자열을 HTML로 변환하여 브라우저로 전달한다.
    <br>3. 자바스크립트는 브라우저 위에서 작동한다. 즉 브라우저에서 작동될때는 서버 템플릿의 영역이 아니어서 제어할 수 없다
    <br>4. Vue.js나 React.js를 이용한 SPA(Single Page Application)는 브라우저에서 화면을 생성한다. 즉 __서버에서 이미 코드가 벗어난 경우!__
     따라서 서버에서는 Json 혹은 XML형식의 데이터만 절달하고 클라이언트에서 조립한다
     -  리액트나 뷰와 같은 자바스크립트 프레임워크에서 서버 사이드 렌더링(Server Side Rending)을 지원하기도한다
     <br> 5.자바스크립트 프레임워크의 화면 생성 방식을 서버에서 실행.__ V8 엔진라이브러리를 지원하기 때문. 스프링프트에서는 Nashorn, J2V8이 있다
    
 
   * 머스테치 : 수많은 언어를 지원하는 가장 심플한 템플릿엔진 
   * 머스테치의 기본위치 :`src/main/resources/templates` 이다. 이 위치에 머스테치 파일을 두면 스프링부트에서 알아서 자동로딩한다
   * 머스테치 스타터가 있기때문에 controller에서 맵핑할 때 src/main/resources/templates 경로와 머스테치 확장자는 생략해도 된다.
   * css는 header에 두고 js는 body 하단에 두어 화면을 빠르게 불러올 수 있게한다
   * index.js를 만들 때 var main = {...} 코드를 선언함. index.js에도 sava function이 있고 다른 js에도 save function이 있다고 가정할경우!
    <br> 브라우저의 스코프는 __공용공간__ 으로 쓰이기때문에 나중에 로딩된 js의 svae가 먼저 로딩된 js의 function을 덮어쓴다
    <br> 여러 사람이 참여하는 프로젝트에서는 중복된 함수 이름이 자주 발생할 수 있다. 모든 function의 이름을 확인하여 만들 수 없기때문에 index.js
     만의 유효범위를 만들어서 사용한다. 
    <br>var main이란 객체를 만들어 해당 객체에서 필요한 모든 function을 선언한다. 이렇게하면 main객체 안에서만 function이 유효하기때문에 다른 JS와
     겹칠 위헙이 사라진다
   * `<script src="/js/app/index.js"></script>`
    <br> footer에 추가할 때 index.js 의 경로를 /js/app/index.js 로 지정해준다
    <br> 스프링부트는 기본적으로 src/main/resources/static에 위치한 자바스크립트, CSS, 이미지 등 정적 파일들은 URL에서 / 로 설정된다
     

   ## 21/03/13
   ### Spring Security login
   * 스프링부트에서는 properties의 이름을 application-xxx.properties로 만들면 xxx라는 이름의 profile이 생성되어 이를 통해 관리할 수 있다.
     즉 __profile=xxx__ 라는 식으로 호출하면 __해당 properties__ 의 설정을 가져올 수 있다.
     <br> 해당 프로젝트에서는 스프링부트 기본 설정파일인 application.properties에서 application-auth.properties를 포함하도록 구성한다.
   * 구글 로그인을 위한 클라이언트 id와 클라이언트 비밀번호는 공유되면 안되기때문에 .gitignore에 추가해준다.  
   * @Enumerated(EnumType.STRING) : 
    JPA로 데이터베이스로 저장할 때 Enum 값을 어떤 형태로 저장할지 결정한다.
     <br> 기본적인 형태는 int형 숫자가 저장된다. 숫자로 저장되면 데이터베이스로 확인할 때 그 값이 무슨 코드를 의미하는지 알 수 없다
     <br> 그래서 문자열(EnumType.STRING)로 저장될 수 있도록 선언한다
   *스프링 시큐리티에서는 권한 코드에 항상 `ROEL_`이 있어야한다
   * findByEmail : 소셜 로그인으로 반환되는 값 중 email을 통해 이미 생성된 사용자인지 처음 가입하는 사용자인지 판단한다

```
    @RequiredArgsConstructor
    @EnableWebSecurity
    public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
        private final CustomOAuth2UserService customOAuth2UserService;
    
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http
                    .csrf().disable()
                    .headers().frameOptions().disable()
                    .and()
                        .authorizeRequests()
                        .antMatchers("/", "/css/**","/images/**","/js/**","/h2-console/**").permitAll()
                        .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                        .anyRequest().authenticated()
                    .and()
                        .logout()
                            .logoutSuccessUrl("/")
                    .and()
                        .oauth2Login()
                            .userInfoEndpoint()
                                .userService(customOAuth2UserService);
        }
    }
```
   * @EnableWebSecurity : Spring Security 설정들을 활성화시켜줌
   * csrf().disable().headers().frameOptions().disable() : h2를 사용하기위해 해당 옵션을 disable처리
   * authorizeRequests()
    <br> URL별 권한관리를 하는 시작점
    <br> authorizeRequests가 선언되어야 andMatchers옵션을 사용할 수 있다
   * andMatchers()
    <br> 권한 관리 대상을 지정하는 옵션
    <br> URL, HTTP 메소드별로 관리가 가능하다
    <br> `("/", "/css/**","/images/**","/js/**","/h2-console/**")` 은 .permitAll()을 통해 전체 열람권한을 주었다
    <br> `("/api/v1/**")`주소를 가진 API는 USER권한을 가진 사람만 가능하도록 지정
   * anyRequest() 
    <br> 설정된 값 이외의 URL
    <br> authenticated()를 추가하여 나머지 URL들은 모두 인증된 사용자들에게만 허용
    <br> 인증된 사용자 = 로그인한 사용자
   * userInfoEndpoint() : OAuth2 로그인 성공 이후 사용자 정보를 가져올 때 설정 담당
   * userService() 
    <br> 소셜 로그인 성공시 후속조치를 진행할 UserService 인터페이스의 구현체를 등록한다
    <br> 리소스 서버(즉, 소셜 서비스들)에서 사용자 정보를 가져온 상태에서 추가로 진행하고자하는 기능을 명시할 수 있다
```
     public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {

    private final UserRepository userRepository;
    private final HttpSession httpSession;


    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {

        OAuth2UserService<OAuth2UserRequest, OAuth2User > delegate = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = delegate.loadUser(userRequest);

        String registrationId = userRequest.getClientRegistration().getRegistrationId();

        String userNameAttributeName = userRequest.getClientRegistration().getProviderDetails()
                .getUserInfoEndpoint().getUserNameAttributeName();

        OAuthAttributes attributes = OAuthAttributes.of(registrationId, userNameAttributeName, oAuth2User.getAttributes());

        User user = saveOrUpdate(attributes);

        httpSession.setAttribute("user", new SessionUser(user));

        return new DefaultOAuth2User(
                Collections.singleton(new SimpleGrantedAuthority(user.getRoleKey())),
                attributes.getAttributes(),
                attributes.getNameAttributeKey()
        );
    }

    private User saveOrUpdate(OAuthAttributes attributes) {

        User user = userRepository.findByEmail(attributes.getEmail())
                .map(entity -> entity.update(attributes.getName(), attributes.getPicture()))
                .orElse(attributes.toEntity());

        return userRepository.save(user);

        }
    } 

``` 

   * registrationId : 현재 로그인 진행중인 서비스 구분 값
   * userNameAttributeName
    <br> OAuth2 로그인 진행시 키가 되는 필드값. primary key와 같은 의미. 
    <br> 구글의 경우 기본적으로 코드를 지원하지만 네이버 카카오등은 지원x. 구글의 기본코드는 "sub"이다
    <br> 구글과 네이버 로그인 동시지원시 사용된다
   * OAuthAttributes : OAuth2UserService를 통해 가져온 OAuth2User의 attribute를 담을 클래스
   * SessionUser : 세션에 사용자 정보를 저장하기위한 DTO 클래스
   * saveOrUpdate : 구글 사용자 정보가 업데이트되었을 경우를 대비하여 update 기능도 함께 구현.
    <br> 사용자의 이름이나 프로필사진이 변경되면 User 엔티티에도 반영된다
   *  OAuthAttributes.of : OAuth2User에서 반환하는 사용자 정보는 Map이기때문에 값 하나하나를 변환해야한다
   * toEntity 
     <br> User 엔티티를 생성한다
     <br> OAuthAttribute에서 엔티티를 생성하는 시점은 처음 가입할 때 이다. 가입할 때 기본권한을 GUEST로 주기 위해서 role 빌더값에는 Role.GUEST를 사용
   * 로그인 후 세션을 처리하는 SessionUser를 만든 이유
    <br> : User 클래스에 직렬화 코드를 넣으면 직렬화 대상에 자식들까지 포함되기때문에 성능이슈, 부수효과가 발생할 확률이 높다.
    <br>   따라서 직렬화 기능을 가진 세션 Dto를 하나 추가로 만드는것이 이후 유지보수 때 많은 도움이 된다.

    
```
    {{#userName}}
    {{/userName}}
    
    {{^userName}}    
    {{/userName}}
```

   * {{#userName}} : 머스테치는 if문을 제공하지않음. true/false 만 판단
    <br> 따라서 머스테치에서는 항상 최종값을 넘겨줘야함. index.mustache에서도 userName이 있다면 userName을 노출시키도록 구성
   * {{^userName}} : 해당값이 존재하지않는 경우 ^ 를 사용
   * "/logout" : 스프링 시큐리티에서 기본적으로 제공하는 로그아웃 URL. SecurityConfig에서 수정 가능. 별도의 컨트롤러 생성할 필요x
   * "/oauth2/authorization/google" : 스피링시큐리티에서 기본적으로 제공하는 로그인 URL. 별도의 컨트롤러 생성할 필요x

   * 권한이 필요한 경우 session에서 계속 사용자의 정보를 받아와야한다
    ```
     httpSession.getAttribute("user"); 
    ```
   <br> 같은 코드가 반복되다보면 추 후 수정할 때 하나하나 찾아가며 수정해야하기때문에 유지보수하기 힘들다.
   <br> 메소드 인자로 세션값을 바로 받을 수 있게 변경해보자
     
   __1. LoginUser 어노테이션 클래스 생성__  

   * @Target(ElementType.PARAMETER)
     + 이 어노테이션이 생성될 수 있는 위치를 지정한다
     + PARAMETER로 지정했으니 메소드의 파라마터로 선언된 객체에서만 사용할 수 있다
     + 이 외에도 클래스 선언문에 쓸 수 있는 TYPE등이 있다
   * @interface
     + 이 파일을 어노테이션 클래스로 지정한다
     + LonginUser라는 이름을 가진 어노테이션이 생성되었다고 보면 된다
   
   __2. LoginUserArgumentResolver 클래스 생성__

   * HandlerMethodArgumentResolver 인터페이스를 구현한 클래스
     + 조건에 맞는 경우 메소드가 있다면 HandlerMethodArgumentResolver의 구현체가 지정한 값으로 해당 메소드의 파라미터로 넘길 수 있다
     
   * suportParameter()
     + 컨트롤러 메서드의 특정 파라미터를 지원하는지 판단한다
     + 여기서는 파라미터에 @LoginUser 어노테이션이 붙어있고, 파라미터 클래스타입이 SessionUser.class인 경우 true를 반환한다
    
   * resolveArgument()
      + 파라미터에 전달할 객체 생성
      + 여기에서는 세션객체를 가져온다
    
   __3. WebConfig 생성__

   * 생성된 LoginUserArgumentResolver가 스프링에서 인식될 수 있게하기위함
   * WebMvcConfugurer 인터페이스 구현
   * HandlerMethodArgumentResolver 는 항상 WebMvcConfugurer의 addArgumentResolvers()를 통해 추가해야한다

   __4. IndexController 수정__

__as-is__
```aidl
SessionUser user = (SessionUser) httpSession.getAttribute("user");
```
__to-be__
```aidl
@LoginUser SessionUser user
```
   * 기존에(User)httpSession.getAttribute("user")로 가져오던 세션 정보값이 개선되었다.
   * 이제는 어느 컨트롤러든지 @LoginUser만 사용하면 세션정보를 가져올 수 있게 되었다

### 세션 저장소로 데이터베이스 사용하기
*  지금은 서비스를 내렸다 올리면 로그인이 풀린다 -> 세션이 내장 톰캣 메모리에 저장되기때문.
<br> 기본적으로 세션은 실행되는 WAS의 메모리에서 저장되고 호출된다. 메모리에 저장되다보니 내장 톰캣처럼 애플리케이션 실행 시 실행되는 구조에서는 항상 초기화된다.
<br> 즉 __배포할 때마다 톰캣이 재시작 되는 것이다__
<br> 또한 2대 이상의 서버에서 서비스하고있다면 __톰캣마다 세션 동기화__ 설정을 해야한다. 그래도 실제 현업에서는 세션 저장소에대해 다음 세가지 중 한가지를 선택한다
   - (1) 톰캣세션을 사용한다
     + 일반적으로 별다른 설정을 하지 않을 때 기본적으로 선택되는 방식이다
     + 톰캣(WAS)에 세션이 저장되기 때문에 2대 이상의 WAS가 구동되는 환경에서는 톰캣들 간의 세션 공유를 위한 추가 설정이 필요하다
   - (2) MySQL 과 같은 데이터베이스를 세션 저장소로 사용한다
     + 여러 WAS간의 공용 세션을 사용할 수 있는 가장 쉬운 방법이다
     + 많은 설정이 필요없지만, 결국 로그인 요청마다 DB IO가 발생하여 성능상 이슈가 발생 할 수 있다
     + 보통 로그인 요청이 많이 없는 백오피스, 사내 시스템 용도에서 많이 사용한다
   - (3) Redis, Memcached와 같은 메모리 DB를 세션저장소로 사용한다
     + B2C 서빕스에서 가장 많이 사용하는 방식이다
     + 실제 서비스로 사용하기 위해서는 Embedded Redis와 같은 방식이 아닌 외부 메모리 서버가 필요하다
* build.gradle 에 라이브러리 추가 `implementation('org.springframework.session:spring-session-jdbc')`
* application.properties에 추가 `spring.session.store-type = jdbc`   
  서비스 올리고 로그인 후 h2-console에 접속해보면 세션을 위한 테이블 2개가 생성되어있다(SPRING_SESSION, SPRING_SESSION_ATTRIBUTE)
  <br> JPA로 인해 세션테이블이 자동생성되었음!
* 현재는 H2기반이기때문에 서비스를 내리면 세션이 풀리지만 후에는 RDS로 변경 예정

### 네이버로그인
* 네이버는 스프링시큐리티를 공식적으로 지원하지않기때문에 common-OAuth2Provider에서 해주던 값들도 전부 수동입력해야한다
* 스프링 시큐리티에서는 하위필드를 명시할 수 없기때문에 최상위필드들만 user_name으로 지정할 수 있다
<br> 네이버의 응닶값 최상위 필드는 resultCode, message, response이다

### 테스트에 시큐리티 적용하기
* 기존에는 API를 호출할 수 있어 테스트코드역시 바로 API를 호출하도록 구성하였다
<br> 하지만 시큐리티 옵션이 활성화되면서 인증된 사용자만 API를 호출할 수 있다. 따라서 테스트 코드마다 인증한 사용자가 호출한 것 처럼 작동하도록 수정해야한다

* 전체테스트 하는 방법 
<br> gradle탭 -> Tasks -> verification -> test 수행
* 책에서는 전체테스트 돌리면 롬복테스트 제외 다른 테스트는 다 실패로 뜨는데 나는 4개만 실패로 뜬다..뭐지-_-

![뭐지](https://user-images.githubusercontent.com/58330668/111737103-553c9a00-88c2-11eb-8910-6b251bd4cdbc.PNG)

(1) 첫번째 실패 테스트인 "hello_가리턴된다"의 메세지를 보면 " No qualifying bean of type 'com.huchuchu.book.config.auth.CustomOAuth2UserService'"
라는 메세지가 등장한다.
이것은 CustomOAuth2UserService를 생성하는데 필요한 소셜 로그인 관련 설정값들이 없기때문에 발생한다
<br> application-oauth.properties에 설정값들을 추가했지만 test에서 인식하지 못하는 이유는 src/main환경과 src/test 환경의 차이 때문이다.
<br> 다만, src/main/resources/application.properties가 테스트코드를 수행할 때도 적용되는 이유는  test에 application.proterties가 없으면
그대로 가져오기 때문! 하지만 자동으로 가져오는 범위는 application.properties까지 이다. 
<br> 따라서 application-oauth.properites는 test파일에 없다고 가져오는 파일이 아니다
<br> 이 문제를 해결하기위해 테스트 환경을 위한 application.properties를 만들어야한다.
<br> 실제 구글 연동까지 진행할 것은 아니기때문에 임의의 설정값을 등록한다
<br> application.properties 작성 전 후 4개 테스트 통과 4개 테스트 실패로 결과가 동일했다...
<br><br>

![에러2](https://user-images.githubusercontent.com/58330668/111738148-2fb09000-88c4-11eb-9a76-a98e2bd61af0.PNG)
<br> (2)Posts_등록된다() 테스트 로그를 보면 응답 결과로 200(정상응답) Status Code를 원했는데 결과는 302(리다이렉션응답)Status Code가 와서 실패했다
<br> 이것은 스프링 시큐리티 설정 때문에 인증되지 않은 사용자의 요청은 이동 시키기 때문이다. 임의로 인증된 사용자를 추가하여 APL테스트만 해볼 수 있다
<br> spring-security-test를 build.gradle에 추가한다
*  @WithMockUser(roles"USER")
    - 인증된 모의(가짜) 사용자를 만들어서 사용한다
    - roles에 권한을 추가할 수 있다
    - 즉, 이 어노테이션으로 인해 ROLE_USER권한을 가진 사용자가 API를 요청하는 것과 동일한 효과를 가지게 된다
    - @WithMockUser 어노테이션은 MockMvc에서만 작동한다
* @BeforeEach : 매번 테스트가 시작되기 전에 MockMvc 인스턴스를 생성한다
* mvc.perform
    - 생성된 MockMvc를 통해 API를 테스트한다
    - 본문(Body) 영역은 문자열로 표현하기 위해 ObjectMapper를 통해 문자열 JSON으로 변환한다

![에러3](https://user-images.githubusercontent.com/58330668/111856360-7d350780-896d-11eb-8bb9-948a1e1ddd3f.PNG)
(3) Hello가_리턴된다 테스트를 확인해보면 첫번째와 동일한 메시지인 " No qualifying bean of type '''com.huchuchu.book.config.auth.CustomOAuth2UserService'''" 이다
<br> HelloControllerTest는 @WebMvcTest를 사용한다. application.properties로 스프링 시큐리티 설정은 작동했지만 @WebMvcTest는 CustomOauth2UserService를 스캔하지않는다
<br> @WebMvcTest는 WebSecurityConfigAdapter, WebMvcConfigurer를 비롯한 @ControllerAdvice, @Controller를 읽는다
<br> 즉 @Repository, @Service, @Component는 스캔대상이 아니다. 따라서 SecurityConfig는 읽었지만, SecurityConfig를 생성하기위해 필요한
CustomOauth2UserService는 읽을 수 없기때문에 에러가 발생했다.
<br> 이 문제를 해결하기 위해선 스캔대상에서 SecurityConfig를 제거한다.
<br> @WebMvcTest에서 excludeFilter 설정해주고 @WithMockUSer(roles="USER") 설정 후 테스트 돌리면

![재밌네정말](https://user-images.githubusercontent.com/58330668/111856785-82478600-8970-11eb-8c19-0a91b105847c.PNG)
(4) 새로운 에러 발생^^! "java.lang.IllegalArgumentException: JPA metamodel must not be empty!"
<br> 이 에러는 @EnableJpaAuditing로 인해 발생한다. @EnableJpaAuditing를 사용하기위해선 최소 하나의 @Entity클래스가 필요하지만 
@WebMvcTest에서는 당연히 없다!
<br> @EnableJpaAuditing와 @SpringBootAllication이 같이 있다보니 @WebMvcTest에서도 스캔함 -> 둘을 분리해주자!
<br> config 패키지에 JapConfig 클래스를 생성해준다

## 21/03/20
### AWS 서버 환경을 만들어보자 - AWS EC2

외부에서 본인이 만든 서비스에 접근하려면 24시간 작동하는 서버가 필요하다. 
<br> (1) 집에 PC를 24시간 구동시킨다
<br> (2) 호스팅서비스(Cafe 24, 코리아호스팅)을 이용한다
<br> (3) 클라우드서비스(AWS, AZURE, GCP)을 이용한다

* 클라우드 서비스 : 인터넷(클라우드)을 통해 서버, 스토리지, 데이터베이스, 네트워크, 소프트웨어, 모니터링등의 컴퓨팅 서비스를 제공
* 클라우드 종류
  - (1) Infrastructure as a Service(IaaS, 아이아스, 이에스)
    + 기존 물리 장비를 미들웨어와 함께 묶어둔 추상화 서비스
    + 가상머신, 스토리지, 네트워크, 운영체제 등의 IT 인프라를 대여해주는 서비스
    + AWS의 EC2, S3 등
  - (2) Platform as a Service (PaaS, 파스) 
    + IaaS에서 한번 더 추상화 한 서비스 -> 한번 더 추상화했기 때문에 많은 기능이 자동화 되어있다
    + AWS의 Beanstalk(빈스톡), Heroku(헤로쿠) 등
  - (3) Software as a Service(SaaS, 사스) 
    + 소프트웨어 서비스
    + 구글 드라이브, 드립박스 등

### [AWS 서버 환경](https://github.com/huchuchu/springBoot/blob/master/EC2.md) 
### [AWS RDS](https://github.com/huchuchu/springBoot/blob/master/RDS.md)

## 21/04/01
### EC2에서 RDS에서 접근확인

1) putty 접속
2) MySQL 접근 테스트를 위해 MySQL CLI를 설치
(실제 EC2의 MySQL을 설치하는 것이 아닌 명령어 라인만 쓰기위한 설치)
```
    sudo yum install mywql
```
3) 설치 후 RDS에 접속
```
    mysql -u 계정 -p -h HOST주소
```
패스워드까지 입력하면 EC2에서 RDS로 접속되는것을 확인 할 수 있다

### EC2 서버에 프로젝트를 배포
1) 깃허브에서 코드를 받아올 수 있도록 EC2에 깃 설치 `sudo yum install git`
2) 설치 확인 `git --version`
3) git clone으로 프로젝트를 저장할 디렉토리 생성 `mkdir ~/app && mkdir ~/app/step1`
    - ~ : 현재 계정의 홈 디렉토리
4) 생성된 디렉토리로 이동 `cd ~/app/step1`
5) 깃 클론 진행 `git clone 레포지토리 web주소` 
6) 코드들이 잘 수행되는지 테스트로 검증 `./gradlew test`
    - 권한이 없을 경우 수정 `chmod +x ./gradlew`
7) 성공!
```
BUILD SUCCESSFUL in 3m 7s
5 actionable tasks: 5 executed
``` 
8) 현재 EC2에는 그레이들을 설치하지않았다. 하지만 gradle Task를 수행할 수 있는 이유는 프로젝트 내부에 포함된 gradlew파일 때문이다.
그레이들이 설치되지 않은 환경, 혹은 다른 상황에서도 해당 프로젝트에 한해서 그레이들을 쓸 수 있도록 지원하는 Wrapper파일이다. 
해당 파일을 직접 이용하기때문에 별도로 설치 할 필요가 없다

## 21/04/03
### 배포스크립트 만들기
작성한 코드를 실제 서버에 반영하는 것을 배포라고한다.
<br> 자동 배포가능한 쉘스크립트 작성 (쉘스크립트 : .sh 파일확장자를 가진 파일. 리눅스에서 기본적으로 사용할 수 있는 스크립트파일)
1) 디렉토리에 deploy.sh를 작성한다 
<br> `vim ~/app/step1/deploy.sh`

```
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step1
PROJECT_NAME=springBooPrj
    - 프로젝트 디렉토리 주소는 스크립트 내에서 자주 사용하는 값이기 때문이 이를 변수로 저장한다
    - PROJECT_NAME도 동일하게 변수로 저장
    - 쉘에서는 타입없이 선언하여 저장한다
    - 쉘에서는 $변수명으로 변수를 사용할 수 있다

cd $REPOSITORY/$PROJECT_NAME/
    - 제일 처음 git clone 받았던 디렉토리로 이동

echo "> Git pull"

git pull
    - 디렉토리 이동 후, master 브랜치의 최신 내용을 받는다

echo "> 프로젝트 Build시작"

./gradle build
    - 프로젝트 내부의 gradlew로 build를 수행한다

echo "> step1 디렉토리로 이동"

cd $REPOSITORY

echo "> Build 파일 복사"

cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/
    - build의 결과물인 jar파일을 복사해 jar파일을 모아둔 위치로 복사한다

echo "> 현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=${pgrep -f ${PROJECT_NAME}.*.jar)
    - 기존에 수행중이던 부트 애플리케이션을 종료한다
    - pgrep은 process id만 추출하는 명령어이다
    - -f 옵션은 프로세스 이름으로 찾는다

echo "> 현재 구동중인 애플리케이션pid : $CURRENT_PID"

if [ -z "CURRENT_pid"]; then
        echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
else
        echo "> kill -15 $CURRENT_pid"
        kill -15 $CURRENT_PID
        sleep 5
fi
    - 현재 구동중인 프로세스가 있는지 없는지를 판단해서 기능을 수행한다
    - process id 값을 보고 프로세스가 있으면 해당 프로세스를 종료한다

echo "> 새애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/ | grep jar | tail -n 1)
    - 새로 실행할 jar 파일명을 찾는다
    - 여러 jar 파일이 생기기때문에 tail-n 으로 가장 나중의 jar파일(최신파일)을 변수에 저장한다

echo "> JAR NAEM : $JAR_NAME"

nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &
    - 찾은 jar파일명으로 해당 jar파일은 nohub으로 실행한다
    - 스프링 부트의 장점으로 특별히 외장 톰캣을 설치할 필요가 없다
    - 내장 톰캣을 사용하여 jar 파일만 있으면 바로 웹 애플리케이션 서버를 실행할 수 있다
    - 일반적으로 자바를 실행할때는 java -jar라는 명령어를 사용하지만, 
      이렇게하면 사용자가 터미널 접속을 끊을 때 애플리케이션도 같이 종료된다
    - 애플리케이션 실행자가 터미널을 종료해도 애플리케이션은 계속 구동될 수 있도록 nohub명령어를 사용한다
```
2) 실행권한을 추가 `chmod +x ./deploy.sh`
```
    -rw-rw-r-- 1 ec2-user ec2-user 872 Apr  2 12:09 deploy.sh
    xr-rwwxr-x 1 ec2-user ec2-user 872 Apr  2 12:09 deploy.sh
```

3) delploy.sh 실행 `./deploy.sh`
 새 애플리케이션 배포 후 nohub.out 파일이 생성되었다
    - 에러
      <br> 오타를 고친후에도 nohub.out에서 application start 메세지를 보지 못했다...
      <br> 프로젝트 jdk버전은 1.8로 설정했는데도 컴파일 제대로 못하는것같아서 일단 11버전 지우고 클래스패스도 다시잡음..
      <br> class파일 변경된거 확인하고 push함
      <br> ec2에서 pull 받으려고하니까 
      `error: Your local changes to the following files would be overwritten by merge:` 에러뜸
      <br> `git stash` 로 해결

### 외부 security 파일 등록하기
ClientRegistrationRepository를 생성하려면 clientId와 clientSecret가 필수이다. 
<br>로컬에서는 application-oauth.properties가 있어서 문제없지만 git에는 없음(.gitignore 대상)
<br>공개된곳에 clientId와 ClientSecret를 올릴 수 없으니 서버에서 직접 이 설정들을 가지고 있게 해주아야한다

1) ~/app/application-oauth.properties 작성
2) deploy.sh 수정
as-is
```
nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &

```
to-be
```
nohub java -jar \
        -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties \
         $REPOSITORY\$JAR_NAME 2>&1 &
            - 스프링 설정 파일 위치를 지정한다
            - 기본 옵션들을 담고 있는 application.properties와 OAuth 설정을 담고있는 application-oauth.properties의 위치를 지정한다
            - calsspath가 붙으면 jar안에있는 resources 디렉토리를 기준으로 경로가 생성된다
            - application-oauth.properties은 절대경로를 사용한다. 외부에 파일이 있기때문이다

```

+ 
    중간에 잘못지워서 putty창 그냥 껐더니 E325: ATTENTION 에러가 발생했다
    <br> 발생원인 : .swp 파일이 발견되었기때문
    <br> 편집도중 비정상적인 종료등에 대비하기위해 파일을 임시저장하는것. 정상적인 방법으로 vi편집기를 종료하면 스왑파일은 자동 삭제되지만
    네트워크 끊김이나 강제종료등의 이유로 끊기면 스왑파일은 삭제되지않고 남아있다
    <br> 해결방법 : `ls -all` 로 스왑파일이 있는지 확인한다
    <br> `ps -ef | grep 파일이름` 으로 다른 사용자가 사용하고있는지 확인한다
    <br> 사용자가 없다면 `vi -r 파일이름` 으로 스왑 파일로 복구한다
    <br> 작업을완료하고 완료하고 저장 후 스왑파일을 삭제한다

### 스프링 부트 프로젝트로 RDS 접근하기
1) 테이블 생성 
<br> H2에서 자동 생성해주던 테이블들을 MariaDB에선 직접 쿼리를 이용해 생성한다
    - JPA가 사용할 엔티티 테이블
    - 스프링 세션이 사용될 테이블
<br> 두가지를 생성한다
<br> 엔티티 테이블은 테스트코드 돌려서 쿼리 가져오고, 스프링 세션 테이블은 schema-mysql.sql검색해서 가져오기(ctrl+Shift+N)    
2) 프로젝트 설정
<br> 자바 프로젝트가 MariaDB에 접근하려면 데이터베이스 드라이버가 필요하다. MariaDB에서 사용 가능한 드라이버를 프로젝트에 추가한다
    - MariaDB 드라이버를 build.gradle에 등록한다
    - src/main/resources/application-real.properties 작성
        + application-real.properties로 파일을 만들면 profile=real인 환경이 구성된다. 
        실제 운영될 환경이기때문에 보안/로그상 이슈가 될 만한 설정들을 모두 제거하며 RDS환경 profile설정이 추가된다.
3) EC2(리눅스서버)설정     
<br> 데이터베이스의 접속 정보는 중요하게 보호해야 할 정보이기때문에 서버내부에서 관리해야한다
    - ~/app/application-real-db.properties를 작성한다
    ```        
        spring.jpa.hibernate.ddl-auto=none
            - JPA로 테이블이 자동생성도는 옵션을 none으로 지정
            - RDS에는 실제 운영으로 사용될 테이블이니 절대 스프리우트에서 새로 만들지않도록 해야한다
            - 이 옵션을 하지 않으면 자칫 테이블이 모두 새로 생성될 수 있다. 
            - 주의해야하는 옵션이다
   
        spring.jpa.show_sql=false
        
        spring.datasource.hikari.jdbc-url=jdbc:mariadb://RDS주소:포트(기본은 3306)/database이름
        spring.datasource.hikari.username=
        spring.datasource.hikari.password=
        spring.datasource.hikari.driver-class-name=org.mariadb.jdbc.Driver

    ```
    - deploy.sh가 real profile을 쓸 수 있도록 개선한다
    ```
        nohup java -jar \
        -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties,
        /home/ec2-user/app/application-real-db.properties,classpath:/application-real.properties \
        -Dspring.profile.active=real \
            - application-real.properties를 활성화시킨다
            - application-real.properties의 spring.profile.include=oauth,real-db옵션때문에 
              real-db역시 함께 활성화 대상에 포함된다 
   
        $REPOSITORY/$JAR_NAME 2>&1 &

    ```   
    - deploy.sh 실행하기
    - curl 명령어로 html코드가 정상적으로 보이면 성공 `curl localhost:8088`
        + curl : 리눅스에서 curl이라는 http 메세지를 쉘상에서 요청하여 결과를 확인하는 명령어
        <br> curl [옵션] [URL]
    
4) EC2에서 소셜로그인하기
    1) AWS 보안그룹 변경
        - EC2에 스프링부트 프로젝트가 8088포트로 배포되었으니 8088포트가 보안그룹에 열려있는지 확인한다
    2) AWS EC2 도메인으로 접속
        - 인스턴스에서 퍼블릭DNS를 확인한다. DNS:8088을 브라우저에 입력하면 접속할 수 있다!
    3) 구글에 EC2 주소 등록
        - Oauth동의화면/승인된 도메인에 DNS등록
        - 리다이렉트URL 등록
    4) 네이버에 EC2 주소 등록
        - API설정 서비스URL에 DNS등록
        - callbackURL등록
    5) 로그인되는지 확인하기!
    
## 21/04/08
### Travis CI 배포 자동화

1) CI와 CD란? 
    <br> 코드 버전관리를 하는 VCS시스템(Git, SVN등)에 PUS되면 자동으로 테스트와 빌드가 수행되어 안정적인 뱁포 파일을 만드는 과정을 CI
    (Continuous Intergration : 지속적 통합), 이 빌드 결과를 자동으로 운영 서버에 무중단 배포까지 진행되는 과정을 
    CD(Continuous Deployment : 지속적 배포)라고 한다.

2) Travis CI 연동하기
    <br> Travis CI : 깃허브에서 제공하는 CI도구
    1. Travis CI 웹서비스 설정
        - https://travis-ci.com/에서 깃헙 계정으로 로그인 후 setting->저장소를 활성화시킨다
    2. 프로젝트 설정
        - .travis.yml 작성
    3. github에 푸시
    ```
        ./gradlew: Permission denied
    ```
    권한 에러떠서 빌드실패함. 
    ```
       before_install:
         - chmod +x gradlew
    ``` 
    .travis.yml에 추가해줌

3) travis CI와 AWS S3 연동하기
    <br> S3 : AWS에서 제공하는 일종의 파일서버. 이미지 파일을 비롯한 정적 파일들을 관리하거나, 배포 파일들을 관리하는 등의 기능을 한다
    1. Travis CI와 S3 연동
        - 실제 배포는 AWS CodeDeploy라는 서비스를 이용한다. 하지만 S3연동이 먼저 필요한 이유는 Jar 파일을 전달하기 위해서
        - CodeDeploy는 저장 기능이 없다. 그래서 Travis CI가 빌드한 결과물을 받아서 CodeDeploy가 
          가져갈 수 있도록 보관할 수 있는 공간이 필요하다. 이때 AWS S3를 사용한다
        1. AWS Key 발급
            - AWS서비스에 외부서비스가 접근할 수 없기때문에, 접근 가능한 권한을 가진 Key를 생성해서 사용해야한다.
              <br> 인증관련 기능을 제공하는 서비스로 IAM(Identity and Access Management)이 있다
            - AWS에 IAM을 검색 / 사용자 탭 / 사용자를 추가하여 Key 발급
        2. Tracis CI에 Key등록
            - 프로젝트 setting / AWS_ACCESS_KEY, AWS_SECRET_KEY를 변수로 Key를 등록
            - 이제 .travis.yml에서 $AWS_ACCESS_KEY, $AWS_SECRET_KEY 이름으로 사용할 수 있다
        3. S3(Simple Storage Service) 버킷생성
            - AWS의 S3는 일종의 파일서버이다. 순수하게 파일들을 저장하고 접근 권한을 관리, 검색 등을 지원하는 파일서버역할을 한다
            - Travis CI에서 생성된 Build파일을 저장하도록 구성할것이다.
            - S3에 저장된 Build파일은 이후 AWS의 CodeDeploy에서 배포할 파일로 가져가도록 구성할것이다.
            - S3를 검색하여 버킷을 생성한다
            - 버킷 이름을 지정할 때는 배포할 Zip파일들이 모여있는 장소임을 의미하도록 짓는편이 좋다
            - 퍼블릭 액세스 관리부분은 모든차단을 해야한다. 
        4. ./travis.yml에 코드추가
        ```
           before_deploy:
                - deploy 명령어가 실행되기 전에 수행된다
                - CodeDeploy는 Jar 파일은 인식하지못하므로 Jar+ 기타 설정파일들을 모아 압축(zip)한다
             - zip -r springBootPrj *
                - 현재 위치의 모든 파일을 springBootPrj 으름으로 압축한다                
             - mkdir -p deploy
                - delploy라는 디렉토리를 TravisCI가 실행중인 위치에서 생성한다
             - mv springBootPrj.zip deploy/springBootPrj.zip
                - zip 파일이동
           
           deploy:
                - S3로 파일 업로드 혹은 CodeDeploy로 배포 등의 외부 서비스와 연동될 행위를 선언한다
             provider: s3
             access_key_id: $AWS_ACCESS_KEY
             secret_access_key: $AWS_SECRET_KEY
             bucket: springbootprj-build 
                - S3 버킷
             region: ap-northeast-2
             skip_cleanup: true
             acl: private
                - 파일접근을 private로
             local_dir: deploy
                - 앞에서 생성하나 deploy디렉토리를 지정한다
                - 해당 위치의 파일들만 S3로 전송한다
             wait-until-deployed: true
        ```
        5. S3 버킷에 가보면 업로드가 성공한것을 확인할 수 있다
        
4) Travis CO와 AWS S3, CodeDeploy 연동하기
<br> AWS의 배포시스템인 CodeDeploy를 이용하기전에 배포 대상인 EC2가 CodeDeploy룰 연동받을 수 있게 IAM 역할을 하나 생성한다
    
    1. EC2에 IAM 역할 추가
        - IAM의 사용자 역할과 차이점
            - 역할 : AWS 서비스에만 할당할 수 있는 권한 / EC2, CodeDeploy, SQS등
            - 사용자 : AWS 서비스 외에 사용할 수 있는 권한 / 로컬PC, IDC서버 등
        - ec2인스턴스 / 우클릭 / 보안 / IAM수정에서 IAM역할 부분을 수정한 후 재부팅한다
    
    2. [CodeDeploy 에이전트 설치](https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/codedeploy-agent-operations-install-linux.html)
    3. CodeDeploy를 위한 권한 생성
    <br>CodeDeploy에서 EC2에 접근하려면 마찬가지로 권한이 필요하다. AWS서비스이니 IAM역할을 생성한다
        - 역할을 생성해준다
    4. CodeDeploy 생성
        - code commit
            - 깃허브와 같은 코드저장소 역할을 한다
            - 프라이빗이 지원되는 강점이있지만, 현재 깃허브에서 무료로 프라이빗 지원을 하고있기때문에 거의 사용안됨
        - Code Build
            - Travis CI와 마찬가지로 빌드용 서비스
            - 멀티 모듈을 배포해야 하는 경우 사용해볼만하지만, 규모가 있는 서비스에서는 대부분 젠킨스/팀시티등을 이용한다
        - CodeDeploy
            - AWS의 배포서비스
            - CodeDeploy는 대체제가 없다
            - 오토 스케일링 그룹 배포, 블루 그린 배포, 롤링 배포, EC2 단독 배포등 많은 기능을 지원한다
        1. Codeploy에 들어가서 애플리케이션 생성을한다
        2. 배포그룹을 생성한다
        3. codeDeploy설정 끝
    5. Travis CI, S3, CodeDeploy 연동
        1. S3에서 넘겨줄 zip파일을 저장할 디렉토리생성
            - Travis CI의 build가 끝나면 S3에 zip파일이 전송, 이 zip파일은 home/ec2-usr/app/step2/zip으로 복사되어 압축을 푼다
            - Travis CI의 설정은 .travis.yml
            - AWS CodeDeploy의 설정은 appspec.yml                       
    
5) 배포 자동화 구성
    
    1. step2 환경에서 실행될 deploy.sh 생성
        - script 디렉토리를 생성하여 여기에 스크립트를 생성한다
    2. .travis.yml 파일수정
        - 현재 프로젝트는 모든 파일을 zip파일로 만드는데 실제로 필요한 파일들은 Jar, appspec.yml, 배포를 위한 스크립트들이다
          나머지는 배포에 필요하지않기때문에 포함x
    3. appspec.yml 수정
    
    4. 이제 Master 브랜치에 푸시하면 자동으로 EC2에 배포된다. 하지만 배포하는동안 스프링부트 프로젝트는 종료상태가 되어 서비스를 이용할 수 없다
    `서비스 중단없는 배포` 방식이 필요하다
    
### 24시간 중단없는 서비스를 만들자

새로운 Jar가 실행되기 전까지 기존 Jar를 종료시켜놓기 때문에 서비스가 중단된다.</br>
1) 무중단 배포 방식
    - AWS에서 블루 그린(Blue-Green) 무중단 배포
    - 도커를 이용한 웹서비스 무중단 배포
    - L4스위치를 이용한 무중단 배포
    - 엔진엑스(Nginx)를 이용한 무중단 배포
        - 리버스 프록시 : 엔진엑스가 외부의 요청을 받아 백앤드 서버로 요청을 전달하는 행위. 리버스 프로시 서버(엔진엑스)는 요청을 전달하고, 
        실제 요청에 대한 처리는 뒷단의 웹 애플리케이션 서버들이 처리한다. 
        - 구조 : 하나의 EC2 혹은 리눅스 서버에 엔진엑스 1대와 스프링 부트 Jar를 2대 사용하는 것 이다.
            - 엔진엑스는 80(http), 443(https)포트를 할당한다
            - 스프링 부트1은 8081포트로 실행한다
            - 스프링 부트2는 8082포트로 실행한다
        - 운영과정
            - 사용자는 서비스 주소로 접속한다(80 혹은 443포트)     
            - 엔진엑스는 사용자의 요청을 받아 현재 연결된 스프링 부트로 요청을 전달한다
            - 스프링 부트2는 엔진엑스와 연결된 상태가 아니니 요청받지 못한다
            - 1.1 버전으로 신규 배포가 필요하면, 엔진엑스와 연결되지않은 스프링 부트2(8082포트)로 배포한다
            - 배포하는 동안에도 서비스는 중단되지않는다
                - 엔진엑스는 스프링 부트1을 바라보기때문
            - 배포가 끝나고 정상적으로 스프링 부트2가 구동중인지 확인한다
            - 스프링 부트2가 정상 구동중이면 nginx reload 명령어를 통해 8081대신에 8082를 바라보도록한다
            - nginx reload는 0.1초 이내에 완료된다

2) 엔진엑스 설치와 스프링 부트 연동하기
    1. EC2에 엔진엑스 설치
    2. 엔진엑스의 포트번호를 보안 그룹에 추가
    3. 리다이렉션 주소 추가
        - 8088이 아닌 80포트로 주소가 변경되니 구글과 네이버 로그인에도 변경된 주소를 등록해야한다
        <br>기존에 등록된 리다이렉션 주소에서 8088부분을 제거하여 추가 등록한다
    4. 기존 주소에서 8080포트를 제거하고 접근. 즉 도메인만 입력해서 접속하면 엔진엑스 웹페이지가 나온다
    5. 엔진엑스와 스프링 부트 연동
        - /ect/nginx/nginx.conf파일 수정
        - 엔진엑스 재시작 `sudo service nginx restart`    

3)  무중단 배포 스크립트 만들기
<br> 무중단 배포스크립트 작업 전에 API를 하나 추가한다. 이 API는 이후 배포 시에 8081을 쓸지, 8082를 쓸지 판단하는 기준이 된다
                            
           
            
            
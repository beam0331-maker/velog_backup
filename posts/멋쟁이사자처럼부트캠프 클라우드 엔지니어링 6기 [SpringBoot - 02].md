# 멋쟁이사자처럼부트캠프 클라우드 엔지니어링 6기 [SpringBoot - 02]

작성일: Tue, 23 Jun 2026 00:31:49 GMT
링크: https://velog.io/@beam0331/%EB%A9%8B%EC%9F%81%EC%9D%B4%EC%82%AC%EC%9E%90%EC%B2%98%EB%9F%BC%EB%B6%80%ED%8A%B8%EC%BA%A0%ED%94%84-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C-%EC%97%94%EC%A7%80%EB%8B%88%EC%96%B4%EB%A7%81-6%EA%B8%B0-SpringBoot-02

---

<hr />
<h1 id="spring-boot-프로파일profile-환경-분리-jdbctemplate-및-mybatis-연동">Spring Boot 프로파일(Profile) 환경 분리, JdbcTemplate 및 MyBatis 연동</h1>
<hr />
<h3 id="📌-오늘의-핵심-키워드">📌 오늘의 핵심 키워드</h3>
<ul>
<li><strong>Spring Boot</strong></li>
<li><strong>Profile (환경 분리)</strong></li>
<li><strong>application.yml</strong></li>
<li><strong>JdbcTemplate &amp; RowMapper</strong></li>
<li><strong>MyBatis 연동</strong></li>
<li><strong>@Mapper</strong></li>
</ul>
<hr />
<h3 id="1-프로파일profile의-개념과-설정-분리">1. 프로파일(Profile)의 개념과 설정 분리</h3>
<h4 id="1-1-프로파일profile-개념">1-1. 프로파일(Profile) 개념</h4>
<ul>
<li><strong>개념 :</strong> 로컬 개발(dev), 상용 운영(prod), 테스트(test) 등 다양한 실행 환경에 맞춰 포트 번호나 데이터베이스 연결 정보 등의 설정값을 분리하여 관리할 수 있도록 지원하는 기능이다.</li>
</ul>
<h4 id="1-2-설정-파일-분리-흐름-및-문법">1-2. 설정 파일 분리 흐름 및 문법</h4>
<ul>
<li><strong>개념 :</strong> 공통으로 적용되는 기본 설정은 application.yml에 작성하고, 환경별 구체적인 설정은 application-{profile}.yml 형식의 파일로 분리하여 작성한다.</li>
</ul>
<table>
<thead>
<tr>
<th>설정 파일</th>
<th>역할 및 상세 설명</th>
</tr>
</thead>
<tbody><tr>
<td><strong>application.yml</strong></td>
<td>공통 설정 및 애플리케이션 구동 시 활성화할 기본 프로파일을 지정한다. (spring.profiles.active)</td>
</tr>
<tr>
<td><strong>application-dev.yml</strong></td>
<td>로컬 및 개발 환경에서 사용하는 설정을 정의한다. (예: H2 In-memory DB 사용, 8082 포트)</td>
</tr>
<tr>
<td><strong>application-prod.yml</strong></td>
<td>실제 서비스 운영(Production) 시 사용하는 설정을 정의한다. (예: MySQL 연동, 8083 포트)</td>
</tr>
</tbody></table>
<hr />
<h3 id="2-프로파일-기반-빈bean-관리-및-활성화">2. 프로파일 기반 빈(Bean) 관리 및 활성화</h3>
<h4 id="2-1-profile을-활용한-빈-등록-제어">2-1. @Profile을 활용한 빈 등록 제어</h4>
<ul>
<li><strong>개념 :</strong> 특정 환경에서만 동작해야 하는 클래스나 설정이 존재할 경우, @Profile 어노테이션을 사용하여 빈의 생성 여부를 제어한다. 클래스 레벨(@Component)이나 메서드 레벨(@Bean) 모두에 적용할 수 있다.</li>
</ul>
<pre><code class="language-java">// 개발 환경(dev) 프로파일이 활성화될 때만 생성되는 빈
@Component
@Profile(&quot;dev&quot;)
public class DevClass {
    public DevClass() {
        System.out.println(&quot;DevClass&quot;);
    }
}

// 설정 클래스 내에서 운영 환경(prod) 프로파일 조건으로 빈 등록
@Configuration
public class ProfileConfiguration {
    @Bean
    @Profile(&quot;prod&quot;)
    public String createProd() {
        System.out.println(&quot;createProd&quot;);
        return &quot;prod 관련 객체 생성&quot;;
    }
}</code></pre>
<h4 id="2-2-프로파일-활성화-방법">2-2. 프로파일 활성화 방법</h4>
<ul>
<li><strong>개념 :</strong> 분리된 프로파일을 애플리케이션 실행 시점에 활성화하기 위해 설정 파일을 수정하거나 커맨드 라인 인자를 전달한다.</li>
</ul>
<table>
<thead>
<tr>
<th>활성화 방식</th>
<th>상세 설명 및 문법</th>
</tr>
</thead>
<tbody><tr>
<td><strong>설정 파일 내부 지정</strong></td>
<td>application.yml 파일 내에 spring.profiles.active: dev 형태로 작성하여 기본값을 설정한다.</td>
</tr>
<tr>
<td><strong>Command Line 인자 전달</strong></td>
<td>빌드된 JAR 파일을 실행할 때 인자를 전달하여 런타임에 동적으로 변경한다. (java -jar myapp.jar --spring.profiles.active=prod)</td>
</tr>
</tbody></table>
<hr />
<h3 id="3-spring-boot-jdbc와-jdbctemplate">3. Spring Boot JDBC와 JdbcTemplate</h3>
<h4 id="3-1-jdbctemplate의-개요-및-특징">3-1. JdbcTemplate의 개요 및 특징</h4>
<ul>
<li><strong>개념 :</strong> Spring에서 순수 JDBC API 사용 시 발생하는 반복적인 코드(Connection 획득, Statement 셋팅, 예외 처리, 자원 반납 등)를 줄이기 위해 제공되는 템플릿 기반의 유틸리티 클래스이다.</li>
<li><strong>특징 :</strong> spring-boot-starter-jdbc 의존성을 추가하면 컨테이너에 의해 자동으로 빈으로 등록되므로, 데이터 접근 계층(DAO/Repository)에서 즉시 의존성 주입을 받아 사용할 수 있다.</li>
</ul>
<h4 id="3-2-핵심-메서드와-rowmapper">3-2. 핵심 메서드와 RowMapper</h4>
<ul>
<li><strong>개념 :</strong> DML(추가, 수정, 삭제) 및 DQL(조회) 작업의 목적에 따라 특화된 메서드를 제공한다. 조회된 ResultSet을 객체로 변환하기 위해 RowMapper 인터페이스를 사용한다.</li>
</ul>
<table>
<thead>
<tr>
<th>핵심 메서드 / 인터페이스</th>
<th>실무 활용 상세 설명</th>
</tr>
</thead>
<tbody><tr>
<td><strong>update()</strong></td>
<td>INSERT, UPDATE, DELETE 등 데이터베이스 상태 변경 쿼리를 실행하며 영향받은 행의 개수를 반환한다.</td>
</tr>
<tr>
<td><strong>queryForObject()</strong></td>
<td>PK를 조건으로 단일 행 데이터를 조회하거나, COUNT(*)와 같은 단일 스칼라 값을 조회할 때 사용한다.</td>
</tr>
<tr>
<td><strong>query()</strong></td>
<td>다중 행 데이터를 조회하여 List 형태로 반환할 때 사용한다.</td>
</tr>
<tr>
<td><strong>RowMapper</strong></td>
<td>ResultSet의 각 행 데이터를 자바 객체(DTO)로 변환해 주는 콜백 인터페이스이다.</td>
</tr>
</tbody></table>
<h4 id="3-3-rowmapper와-람다lambda-표현식-적용-dao-구현-예시">3-3. RowMapper와 람다(Lambda) 표현식 적용 (DAO 구현 예시)</h4>
<ul>
<li><strong>개념 :</strong> RowMapper는 단일 추상 메서드를 가진 함수형 인터페이스이므로, Java 8 이상에서는 람다(Lambda) 표현식을 사용하여 코드를 간결하게 작성할 수 있다.</li>
</ul>
<pre><code class="language-Java">// DAO 레이어: TodoRepository.java
@Repository
public class TodoRepository {

    @Autowired
    JdbcTemplate template;

    public TodoRepository(JdbcTemplate template) {
        this.template = template;
    }

    // 람다 표현식을 활용한 전체 목록 조회
    public List&lt;TodoDTO&gt; findAll() {
        String sql = &quot;select id, name, job from todo&quot;;
        return template.query(sql, (rs, rowNum) -&gt; {
            TodoDTO todoDTO = new TodoDTO();
            todoDTO.setId(rs.getInt(&quot;id&quot;));
            todoDTO.setName(rs.getString(&quot;name&quot;));
            todoDTO.setJob(rs.getString(&quot;job&quot;));
            return todoDTO;
        });
    }

    // 데이터 삽입
    public int save(TodoDTO todoDTO) {
        String url = &quot;insert into todo (id, name, job) values (?, ?, ?)&quot;;
        return template.update(url, todoDTO.getId(), todoDTO.getName(), todoDTO.getJob());
    }
}</code></pre>
<h4 id="3-4-service-코드-구현-예시">3-4. Service 코드 구현 예시</h4>
<pre><code class="language-Java">// Service 레이어: TodoServiceImpl.java
@Service(&quot;todoService&quot;)
public class TodoServiceImpl implements TodoService{

    TodoRepository todoRepository;

    public TodoServiceImpl(TodoRepository todoRepository) {
        this.todoRepository = todoRepository;
    }

    @Override
    public List&lt;TodoDTO&gt; findAll() {
        return todoRepository.findAll();
    }

    @Override
    @Transactional
    public int save(TodoDTO todoDTO) {
        return todoRepository.save(todoDTO);
    }
}</code></pre>
<hr />
<h3 id="4-spring-boot와-mybatis-연동-흐름">4. Spring Boot와 MyBatis 연동 흐름</h3>
<h4 id="4-1-applicationyml을-통한-mybatis-환경-설정">4-1. application.yml을 통한 MyBatis 환경 설정</h4>
<ul>
<li><strong>개념 :</strong> 과거 Spring 프레임워크에서 별도로 관리하던 configuration.xml의 역할을 Spring Boot에서는 application.yml이 대체한다.</li>
</ul>
<pre><code class="language-yaml"># application.yml 설정 예시
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/testdb
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 1234

  sql:
    init:
      mode: always # 구동 시 테이블 생성 및 데이터 자동 주입

mybatis:
  mapper-locations: classpath:com/exam/config/*Mapper.xml
  type-aliases-package: com.exam.dto</code></pre>
<h4 id="4-2-아키텍처-흐름-및-mapper-규칙">4-2. 아키텍처 흐름 및 Mapper 규칙</h4>
<ul>
<li><strong>개념 :</strong> MyBatis 연동은 인터페이스와 XML의 1:1 매핑을 기반으로 작동한다. Service 계층에서 @Mapper 인터페이스를 호출하면, 프레임워크가 매핑된 Mapper.xml을 찾아 쿼리를 실행한다.</li>
<li><strong>주의 사항 :</strong> Mapper.xml은 반드시 src/main/resources 하위 경로에 위치해야 한다.</li>
</ul>
<h4 id="4-3-mybatis-연동-핵심-코드">4-3. MyBatis 연동 핵심 코드</h4>
<pre><code class="language-Java">// Mapper 인터페이스: TodoMapper.java
@Mapper
public interface TodoMapper {
    List&lt;TodoDTO&gt; findAll();
    int save(TodoDTO todoDTO);
}</code></pre>
<pre><code class="language-Java">// Service 레이어: TodoServiceImpl.java
@Service(&quot;todoService&quot;)
public class TodoServiceImpl implements TodoService {

    @Autowired
    TodoMapper todoMapper; // MyBatis 매퍼 인터페이스 주입

    @Override
    public List&lt;TodoDTO&gt; findAll() {
        return todoMapper.findAll();
    }

    @Override
    @Transactional
    public int save(TodoDTO todoDTO) {
        return todoMapper.save(todoDTO);
    }
}</code></pre>
<pre><code>&lt;mapper namespace=&quot;com.exam.mapper.TodoMapper&quot;&gt;
    &lt;select id=&quot;findAll&quot; resultType=&quot;TodoDTO&quot;&gt;
        SELECT id, name, job FROM todo
    &lt;/select&gt;
    &lt;insert id=&quot;save&quot; parameterType=&quot;TodoDTO&quot;&gt;
        INSERT INTO todo (id, name, job) VALUES (#{id}, #{name}, #{job})
    &lt;/insert&gt;
&lt;/mapper&gt;</code></pre><hr />
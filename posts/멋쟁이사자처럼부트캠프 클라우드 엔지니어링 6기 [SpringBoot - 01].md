# 멋쟁이사자처럼부트캠프 클라우드 엔지니어링 6기 [SpringBoot - 01]

작성일: Sun, 21 Jun 2026 05:50:48 GMT
링크: https://velog.io/@beam0331/%EB%A9%8B%EC%9F%81%EC%9D%B4%EC%82%AC%EC%9E%90%EC%B2%98%EB%9F%BC%EB%B6%80%ED%8A%B8%EC%BA%A0%ED%94%84-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C-%EC%97%94%EC%A7%80%EB%8B%88%EC%96%B4%EB%A7%81-6%EA%B8%B0-SpringBoot-01

---

<h1 id="spring-framework와-spring-boot-핵심-개념-및-빈bean-관리">Spring Framework와 Spring Boot 핵심 개념 및 빈(Bean) 관리</h1>
<hr />
<h3 id="📌-오늘의-핵심-키워드">📌 오늘의 핵심 키워드</h3>
<ul>
<li><strong>Spring Framework</strong></li>
<li><strong>Spring Boot</strong></li>
<li><strong>IoC (Inversion of Control)</strong></li>
<li><strong>DI (Dependency Injection)</strong></li>
<li><strong>Spring Bean &amp; Scope</strong></li>
<li><strong>AOP (Aspect Oriented Programming)</strong></li>
</ul>
<hr />
<h3 id="1-spring-framework-개요-및-특징">1. Spring Framework 개요 및 특징</h3>
<h4 id="1-1-주요-특징">1-1. 주요 특징</h4>
<ul>
<li><strong>개념 :</strong> 2003년 로드 존슨이 개발한 오픈소스 프레임워크, 무겁고 플랫폼에 종속적이던 EJB를 대체하기 위해 등장했다.</li>
<li><strong>IoC 컨테이너 기반 :</strong> 객체의 생성, 소멸, 의존성 주입을 Container가 대신 관리한다. 대표적인 IoC 컨테이너는 ApplicationContext이다.</li>
<li><strong>선언적 트랜잭션 처리 :</strong>  JDBC나 MyBatis 환경에서는 DML 작업 시 <code>commit()</code>이나 <code>rollback()</code>을 수동으로 호출해야 했으나, Spring은 <code>@Transactional</code> 어노테이션으로 정상 수행 시 Commit, 런타임 에러 발생 시 Rollback을 자동 처리한다.</li>
<li><strong>REST API 지원 :</strong> 기존 SOAP 방식의 HTML 응답 구조에서 벗어나, 다양한 클라이언트(모바일, 테블릿, 가전 등)가 처리할 수 있는 JSON 포맷 기반의 RESTful 아키텍처를 지원한다.</li>
<li><strong>Spring 생태계 :</strong> 코어 역할을 하는 Spring Framework뿐만 아니라, Spring Boot, Spring Security, Spring Batch, Spring AI 등 목적에 맞는 다양한 하위 프로젝트가 거대한 생태계를 이룬다.</li>
</ul>
<h4 id="1-2-핵심-용어-정리">1-2. 핵심 용어 정리</h4>
<ul>
<li><strong>POJO (Plain Old Java Object) :</strong> 특정 프레임워크나 플랫폼에 종속되지 않은 순수한 자바 객체를 의미한다. 상속이나 인터페이스 구현에 제약이 없어 재사용성이 높다.</li>
<li><strong>Spring Bean :</strong> Spring IoC 컨테이너가 생성하고 관리하는 POJO 기반의 객체이다.</li>
<li><strong>의존성 주입 (DI: Dependency Injection) :</strong> 객체가 다른 객체를 필요로 할 때 내부에서 직접 생성하지 않고, 컨테이너에서 생성된 객체를 외부로부터 주입받는 방식이다.</li>
<li><strong>AOP (Aspect Oriented Programming) :</strong> 관점 지향 프로그래밍. 애플리케이션을 핵심 비즈니스 로직, 시스템 전반에서 공통으로 발생하는 부가 기능(로깅, 보안, 트랜잭션 처리 등)으로 분리하는 기법이다. 코드의 중복을 크게 줄이고 유지보수성을 향상시킨다.</li>
</ul>
<hr />
<h3 id="2-spring-boot의-등장-배경과-주요-기능">2. Spring Boot의 등장 배경과 주요 기능</h3>
<h4 id="2-1-spring-boot의-특징">2-1. Spring Boot의 특징</h4>
<ul>
<li><strong>개념 :</strong> 2016년에 등장하여 Spring Framework의 복잡한 환경 설정 문제를 해결하고 비즈니스 로직 개발에 집중할 수 있도록 돕는 프레임워크이다.</li>
<li><strong>라이브러리 및 설정 자동화 :</strong> Starter 패키지를 통해 환경설정을 자동 관리하고, 내장 Tomcat을 제공하여 서버 설정을 간소화한다.</li>
<li><strong>독립 실행 :</strong> Java SE와 EE 환경 모두에서 독립적으로 실행 가능한 JAR 파일 배포를 지원한다.</li>
</ul>
<h4 id="2-2-starter와-빌드-도구">2-2. Starter와 빌드 도구</h4>
<table>
<thead>
<tr>
<th>항목</th>
<th>실무 활용 상세 설명</th>
</tr>
</thead>
<tbody><tr>
<td><strong>Maven</strong></td>
<td>XML(<code>pom.xml</code>) 기반의 전통적인 빌드 도구이다. 레퍼런스가 많아 안정적이지만, 설정 스크립트가 길어지고 가독성이 떨어질 수 있다.</td>
</tr>
<tr>
<td><strong>Gradle</strong></td>
<td>Groovy/Kotlin DSL(<code>build.gradle</code>) 기반의 최신 빌드 도구이다. 유연한 스크립트 설정과 빠른 빌드 속도를 제공하여 최근 선호된다.</td>
</tr>
<tr>
<td><strong>Spring Boot Starter</strong></td>
<td>특정 기능 구현에 필요한 다수의 JAR 파일을 묶음으로 제공한다. (예: <code>spring-boot-starter-web</code>)</td>
</tr>
</tbody></table>
<hr />
<h3 id="3-spring-bean-생성-및-관리-방법">3. Spring Bean 생성 및 관리 방법</h3>
<h4 id="3-1-컴포넌트-스캔을-통한-빈-등록">3-1. 컴포넌트 스캔을 통한 빈 등록</h4>
<ul>
<li><strong>개념 :</strong> <code>@SpringBootApplication</code>이 위치한 패키지와 그 하위 패키지에서 스테레오타입 어노테이션(<code>@Component</code>, <code>@Service</code>, <code>@Repository</code> 등)이 붙은 클래스를 자동으로 빈으로 등록한다.</li>
<li><strong>내부 동작 원리:</strong> <code>@SpringBootApplication</code>은 내부에 <code>@ComponentScan</code>(자동 빈 스캔)과 <code>@EnableAutoConfiguration</code>을 포함하고 있어, 빌드 도구에 등록된 의존성을 파악해 최적의 환경을 자동으로 구성한다.</li>
</ul>
<pre><code class="language-java">// 서비스 레이어의 역할을 명시하며 자동으로 빈으로 등록된다.
@Service(&quot;service&quot;)
public class ServiceImpl {
    public ServiceImpl() {
        System.out.println(&quot;ServiceImpl&quot;);
    }
}</code></pre>
<blockquote>
<p><strong>팁:</strong> 타 패키지의 클래스를 등록해야 할 경우 <code>@SpringBootApplication(scanBasePackages = {&quot;com.other&quot;})</code> 속성을 통해 스캔 범위를 명시적으로 확장할 수 있다.</p>
</blockquote>
<h4 id="3-2-명시적-빈-등록">3-2. 명시적 빈 등록</h4>
<ul>
<li><strong>개념 :</strong> 설정 클래스에서 개발자가 직접 인스턴스를 반환하는 메서드를 작성하여 수동으로 빈을 등록하는 방식이다. 외부 라이브러리 클래스를 빈으로 등록할 때 주로 사용한다.</li>
</ul>
<pre><code class="language-java">// 설정 정보를 포함하는 클래스임을 명시하고, 수동으로 빈을 등록한다.
@Configuration
public class DeptConfiguration {
    @Bean
    public DeptDAO createDeptDAO() {
        return new DeptDAO();
    }
}</code></pre>
<ul>
<li><strong>명시적 빈 등록의 핵심 관점 :</strong><ul>
<li><strong>외부 라이브러리 제어 :</strong> 코드를 직접 수정할 수 없는 외부 라이브러리 클래스를 빈으로 등록해야 할 때, 별도의 설정 클래스(<code>@Configuration</code>)에서 수동으로 객체를 생성해 주입한다.</li>
<li><strong>싱글톤 보장 :</strong> <code>@Configuration</code>이 선언된 클래스는 스프링이 내부적으로 CGLIB 프록시 객체로 감싸 관리하므로, 내부 메서드를 여러 번 호출해도 항상 동일한 인스턴스(싱글톤) 반환을 보장한다.</li>
</ul>
</li>
</ul>
<hr />
<h3 id="4-의존성-주입di-방식">4. 의존성 주입(DI) 방식</h3>
<h4 id="4-1-생성자-주입-constructor-injection">4-1. 생성자 주입 (Constructor Injection)</h4>
<ul>
<li><strong>개념 :</strong> Spring에서 가장 권장하는 의존성 주입 방식으로, 객체 생성 시점에 의존성을 주입받아 불변성을 보장한다.</li>
</ul>
<pre><code class="language-java">// Spring 4.3 이상에서는 생성자가 단일할 경우 @Autowired 생략이 가능하다.
@Service(&quot;service&quot;)
public class DeptServiceImpl {
    DeptDAO dao;

    // 생성자를 통한 의존성 주입
    public DeptServiceImpl(DeptDAO dao) {
        this.dao = dao;
    }
}</code></pre>
<h4 id="4-2-복수-빈-주입-및-qualifier">4-2. 복수 빈 주입 및 @Qualifier</h4>
<ul>
<li><strong>개념 :</strong> 동일한 인터페이스를 구현한 빈이 여러 개일 경우, 주입할 빈을 명시적으로 지정해야 한다.</li>
</ul>
<table>
<thead>
<tr>
<th>어노테이션</th>
<th>상세 설명</th>
</tr>
</thead>
<tbody><tr>
<td><strong><code>@Qualifier(&quot;빈이름&quot;)</code></strong></td>
<td>의존성 주입 시 주입할 특정 빈의 이름을 명시적으로 지정한다.</td>
</tr>
<tr>
<td><strong><code>@Primary</code></strong></td>
<td>여러 빈이 존재할 때 주입될 기본 빈 후보를 설정한다. (<code>@Qualifier</code>가 우선순위가 더 높다)</td>
</tr>
</tbody></table>
<pre><code class="language-java">public interface CommonDAO {}

@Repository(&quot;deptDAO&quot;)
public class DeptDAO implements CommonDAO {}

@Repository(&quot;empDAO&quot;)
public class EmpDAO implements CommonDAO {}

@Service
public class ComponentService {
    private final CommonDAO dao;

    // 인터페이스 타입의 빈이 복수일 때 @Qualifier로 특정 빈을 명시하여 주입한다.
    public ComponentService(@Qualifier(&quot;deptDAO&quot;) CommonDAO dao) {
        this.dao = dao;
    }
}</code></pre>
<hr />
<h3 id="5-빈의-스코프scope-및-생명주기">5. 빈의 스코프(Scope) 및 생명주기</h3>
<h4 id="5-1-bean-scope-종류-및-지정-방법">5-1. Bean Scope 종류 및 지정 방법</h4>
<ul>
<li><strong>개념 :</strong> 빈이 생성되어 존재하는 범위를 지정한다. <code>@Scope</code> 어노테이션의 속성값을 명시하여 설정한다.</li>
</ul>
<table>
<thead>
<tr>
<th>스코프 (Scope)</th>
<th>상세 설명</th>
</tr>
</thead>
<tbody><tr>
<td><strong>Singleton</strong></td>
<td>기본값. IoC 컨테이너 내에 단 하나의 인스턴스만 생성되어 여러 요청에서 공유한다.</td>
</tr>
<tr>
<td><strong>Prototype</strong></td>
<td>요청(주입) 시점마다 매번 새로운 인스턴스를 생성하여 반환한다. 여러 사용자가 공유할 수 없다.</td>
</tr>
</tbody></table>
<pre><code class="language-java">// 프로토타입 스코프 지정 예시
@Component
@Scope(&quot;prototype&quot;)
public class MyPrototypeBean {
    public MyPrototypeBean() {
        System.out.println(&quot;MyPrototypeBean 생성&quot;);
    }
}</code></pre>
<h4 id="5-2-빈-생명주기-콜백-초기화-및-자원-반납">5-2. 빈 생명주기 콜백 (초기화 및 자원 반납)</h4>
<ul>
<li><strong><code>@PostConstruct</code>:</strong> 객체 생성 및 의존성 주입 직후 실행될 초기화 메서드에 지정한다. (서블릿의 <code>init()</code>과 유사)</li>
<li><strong><code>@PreDestroy</code>:</strong> IoC 컨테이너 종료 전 자원 반납이나 정리 작업이 필요한 메서드에 지정한다. (서블릿의 <code>destroy()</code>와 유사)</li>
</ul>
<hr />
<h3 id="6-외부-환경-설정-external-configuration">6. 외부 환경 설정 (External Configuration)</h3>
<h4 id="6-1-환경-변수-관리-및-우선순위">6-1. 환경 변수 관리 및 우선순위</h4>
<ul>
<li><strong>개념 :</strong> 운영 환경과 개발 환경의 설정값(서버 포트, DB 연결 정보 등)을 자바 코드와 분리하여 <code>application.properties</code> 또는 <code>application.yml</code> 파일에서 관리한다. 외부에 설정된 환경 변수는 고유한 우선순위를 가지며 상위 설정이 하위 설정을 오버라이딩한다.</li>
</ul>
<pre><code>[우선순위 관리 구조]
Environment (최상위 인터페이스)
   └── PropertySource
         1. CommandLinePropertySource (가장 높음, ex: --server.port=8090)
         2. Java System Properties (-Dserver.port=8092)
         3. OS Environment Variables (SERVER_PORT=8093)
         4. Config Data (가장 낮음, application.yml)</code></pre><h4 id="6-2-환경-변수-호출-및-사용-방법">6-2. 환경 변수 호출 및 사용 방법</h4>
<ul>
<li><strong>@Value (단일 값 조회) :</strong> 클래스의 필드에 지정된 키값에 해당하는 환경 변수 데이터를 직접 주입받는다.</li>
<li><strong>@ConfigurationProperties (멀티 값 조회) :</strong> 관련 있는 설정값들을 묶어서 계층 구조의 자바 클래스 단위로 바인딩하여 관리하며, 메인 클래스에 <code>@ConfigurationPropertiesScan</code> 활성화가 필요하다.</li>
</ul>
<pre><code class="language-java">// @Value를 이용한 단일 환경 변수 호출
@Component
public class MyValueComponent {
    @Value(&quot;${server.port}&quot;)
    private String serverPort;
}

// @ConfigurationProperties를 이용한 그룹 속성 호출
/*
 * [application.yml 에 등록된 환경변수 예시]
 * mail:
 * host: smtp.google.com
 * port: 587
 *
 * [application.properties 에 등록된 환경변수 예시]
 * mail.host=smtp.google.com
 * mail.port=587
 */
@Component
@ConfigurationProperties(prefix = &quot;mail&quot;)
public class MyMailProperties {
    private String host;
    private int port;

    public void setHost(String host) { this.host = host; }
    public void setPort(int port) { this.port = port; }
}</code></pre>
<hr />
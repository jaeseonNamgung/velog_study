<h1 id="error-executing-ddl-alter-table-card-drop-foreign-key">Error executing DDL &quot;alter table card drop foreign key</h1>
<pre><code class="language-yaml">org.hibernate.tool.schema.spi.CommandAcceptanceException: Error executing DDL &quot;alter table card drop foreign key FK8inky10unwdfjsml9b01lg2tu&quot; via JDBC [Table '' doesn't exist]</code></pre>
<p><code>schema.sql</code> <code>data.sql</code>  사용 시 <code>jpa.hibernate.ddl-auto: none</code> 처리되어 있는지 확인</p>
<h1 id="name-or-service-not-known-communications-link-failure">Name or service not known, Communications link failure</h1>
<pre><code class="language-java">Caused by: java.net.UnknownHostException: local: Name or service not known
// or
Caused by: com.mysql.cj.exceptions.CJCommunicationsException: Communications link failure</code></pre>
<p>이 오류는 <strong>Spring Boot 애플리케이션</strong>에서 데이터베이스(MySQL, Redis 등)와의 네트워크 통신 시, <strong>잘못된 호스트 이름</strong>을 사용했을 때 발생합니다. 특히, Docker Compose 환경에서는 데이터베이스의 호스트 이름이 <strong>컨테이너 이름</strong>으로 지정되어야 하는데, 잘못된 호스트 이름(<code>localhost</code> 또는 <code>local</code>)을 사용하면 해당 데이터베이스 컨테이너와 통신할 수 없어 이 오류가 발생합니다.</p>
<pre><code class="language-yaml">services:
  database:
    container_name: thred_db
    restart: unless-stopped
    image: mysql</code></pre>
<ul>
<li>잘못된 예시</li>
</ul>
<pre><code class="language-yaml">environment:
  - SPRING_DATASOURCE_URL=jdbc:local://local:3306/thRedDb</code></pre>
<ul>
<li>컨테이너 이름으로 설정</li>
</ul>
<pre><code class="language-yaml">environment:
  - SPRING_DATASOURCE_URL=jdbc:mysql://thred_db:3306/thRedDb</code></pre>
<h1 id="table--doesnt-exist">Table '' doesn't exist</h1>
<pre><code class="language-java">Caused by: java.sql.SQLSyntaxErrorException: Table 'thRedDb.member' doesn't exist </code></pre>
<p><strong>1. <code>jpa.hibernate.ddl-auto: none</code></strong>
<strong>2. properties 또는 yaml 설정 파일에 <code>sql.init.mode: always</code> ,<code>defer-datasource-initialization: true</code>  추가</strong></p>
<p><code>sql.init.mode: always</code>: 데이버베이스 초기화 스크립트(<code>schema.sql</code>과 <code>data.sql</code>) 를 실행할 조건을 제어합니다.</p>
<p>옵션 값:</p>
<ul>
<li><p><strong><code>never</code></strong>:</p>
<ul>
<li>초기화 스크럽트를 <strong>절대 실행하지 않음</strong></li>
<li><code>schema.sql</code>과 <code>data.sql</code>이 무시됨</li>
</ul>
</li>
<li><p><strong><code>embedded</code></strong> <em>(기본값)</em>:</p>
<ul>
<li>데이터베이스가 <strong>임베디드 데이터베이스</strong>(H2, HSQL, Derby)일 때만 초기화 스크립트를 실행</li>
<li>외부 데이터베이스(MySQL, Oracle, PostgreSQL 등)에서는 실행하지 않음</li>
</ul>
</li>
<li><p><strong><code>always</code></strong>:</p>
<ul>
<li><p>데이터베이스 종류에 상관없이 항상 초기화 스크립트(<code>schema.sql</code>과 <code>data.sql</code>)를 실행</p>
</li>
<li><p>Spring Boot 2.5 이상에서는 외부 데이터베이스에서 강제로 실행하려면 이 옵션을 사용해야 합니다.</p>
<p><code>spring.jpa.defer-datasource-initialization</code>은 <strong>JPA 초기화 순서</strong>와 데이터베이스 초기화 간의 의존성을 조정하는 설정입니다.</p>
<p>기본적으로, Spring Boot는 <code>schema.sql</code>과 <code>data.sql</code> 스크립트를 JPA 엔티티 스키마가 생성된 후 실행합니다. 하지만, JPA 스키마 생성 전에 <code>schema.sql</code>과 <code>data.sql</code>을 실행하고 싶을 때, 이 설정을 사용합니다.</p>
<p>옵션 값:</p>
</li>
<li><p><strong><code>true</code></strong>:</p>
<ul>
<li>JPA의 데이터베이스 초기화(Hibernate가 테이블 생성) 전에 <strong><code>schema.sql</code>과 <code>data.sql</code>을 실행</strong>.</li>
<li>즉, 초기화 스크립트 실행을 <strong>JPA 작업보다 앞서 수행</strong>합니다.</li>
</ul>
</li>
<li><p><strong><code>false</code></strong> <em>(기본값)</em>:</p>
<ul>
<li>JPA 엔티티가 스키마를 생성한 후에 <code>schema.sql</code>과 <code>data.sql</code>을 실행.</li>
</ul>
</li>
</ul>
</li>
</ul>
<p><strong>3. docker compose yaml db 컨테이너에서 <code>restart</code> 설정이 <code>unless-stopped</code>로 설정되어 있는지 확인</strong></p>
<p><img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/551fe3c6-c325-441c-b866-86502718934a/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/64f9568d-585d-46ef-bf86-4a78ad53d238/image.png" /></p>
<ul>
<li><strong><code>restart: always</code></strong><ul>
<li>컨테이너가 중지되거나 재부팅될 때 <strong>무조건 재시작</strong>합니다.</li>
<li>심지어 애플리케이션 내부에서 오류가 발생해 종료되더라도 컨테이너는 계속 재시작됩니다.</li>
</ul>
</li>
<li><strong><code>restart: on-failure</code></strong><ul>
<li>컨테이너가 <strong>비정상 종료</strong>(<code>exit code != 0</code>)될 때만 재시작합니다.</li>
<li>애플리케이션이 정상적으로 종료(<code>exit code = 0</code>)되면 재시작하지 않습니다.</li>
</ul>
</li>
</ul>
<h3 id="spring-boot가-db에-연결을-시도할-때-db가-준비되지-않은-상태"><strong>Spring Boot가 DB에 연결을 시도할 때 DB가 준비되지 않은 상태</strong></h3>
<p>Spring Boot 애플리케이션이 실행될 때 MySQL 컨테이너가 아직 초기화 중이라면, 스프링이 데이터베이스 연결을 시도하다가 실패할 수 있습니다. 특히, <code>schema.sql</code>과 <code>data.sql</code>이 실행되는 동안 Spring Boot가 DB에 접근하려고 하면 오류가 발생할 가능성이 높습니다.</p>
<ul>
<li><p><strong><code>restart: always</code>의 문제점</strong>:</p>
<p>  DB 컨테이너가 계속 재시작되기 때문에 Spring Boot 애플리케이션은 반복적으로 연결을 시도하지만 DB가 준비되기 전에 Spring Boot가 연결을 시도해 실패합니다.</p>
</li>
<li><p><strong><code>restart: on-failure</code>의 장점</strong>:</p>
<p>  컨테이너가 오류로 중단되지 않으면 재시작되지 않으므로, 초기화 도중에도 오류가 없으면 컨테이너는 멈추지 않고 정상적으로 실행됩니다.</p>
<p>  결과적으로 DB 컨테이너의 상태가 안정적으로 유지되어 Spring Boot와 연결이 가능합니다.</p>
</li>
</ul>
<p><strong>4. 테이블 대소문자 확인</strong></p>
<p>MySQL 8.0 버전에서 <code>lower_case_table_names</code> 속성의 기본값은 <strong>운영 체제에 따라 다릅니다.</strong></p>
<ul>
<li><strong>Windows</strong>: 기본값은 <code>1</code>로, 테이블 이름이 대소문자를 구분하지 않습니다.</li>
<li><strong>Mac, Unix/Linux</strong>: 기본값은 <code>0</code>으로, 테이블 이름이 대소문자를 구분합니다.</li>
</ul>
<p>즉 테이블 이름의 대소문자를 <strong>구분하지 않으려면</strong>, <code>lower_case_table_names</code> 속성을 <code>1</code>로 설정해야 합니다.</p>
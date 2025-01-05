<h1 id="rabbitmq란">RabbitMQ란?</h1>
<p>RabbitMQ는  AMQP를 구현한 오픈소스 메세지 브로커로서 시스템 간의 데이터를 안전하고 효율적으로 전달하는 데 사용된다. 다양한 분산 시스템에서 메시지 큐를 통해 데이터를 교환하고 비동기적으로 처리된다.</p>
<h2 id="사용하는-이유">사용하는 이유</h2>
<ol>
<li>시스템 간에 메시지 큐를 통해 비동기적으로 통신 가능</li>
<li>시스템 간의 느슨한 결합 -&gt; 생산자와 소비자가 직접적으로 통신하지 않고 RabbitMQ를 통해 간접적으로 통신하므로 각 시스템 간의 의존성이 줄어든다.</li>
<li>메시지 신뢰성과 내구성 강화</li>
<li>메시지 큐잉을 통한 분산 처리 가능</li>
<li>TTL(Time To Live) 설정 및 모니터리 가능</li>
<li>다양한 프로토콜 및 라이브러리 지원</li>
</ol>
<h1 id="amqp란">AMQP란</h1>
<blockquote>
<p>AMQP는 메시지 지향 미들웨어(MOM) 시스템 간에 통신하기 위한 개방형 네트워크 프로토콜이다. 간단히 말해서, 송신자(Producer)와 수신자(Consumer) 사이에서 메시지를 안전하게 교환하는 표준 프로토콜이다.</p>
</blockquote>
<h2 id="amqp의-구조">AMQP의 구조</h2>
<ol>
<li>Producer: 메시지를 생성하고 전송하는 역할, 메세지를 Exchange에 publish 한다.</li>
<li>Exchange: Producer로부터 받은 메시지를 큐로 라우팅</li>
<li>Queue: Exchange로부터 라우팅된 메시지를 저장하는 공간</li>
<li>Consumer: 큐에서 메시지를 읽어 처리하는 역할</li>
<li>Binding: Exchange 가 어떤 방식으로 Queue에 메시지를 보낼지 정의</li>
</ol>
<h3 id="binding바인딩-전략">Binding(바인딩) 전략</h3>
<ol>
<li>Direct Exchange: 메세지의 라우팅 키와 정확히 일치하는 Queue에 값을 전달</li>
<li>Fanout Exchange: Binding된 모든 Queue에 값을 전달</li>
<li>Topic Exchange: 특정 라우팅 패턴이 일치하는 Queue로 값을 전달</li>
<li>Headers Exchange: Key-Value로 정의된 Header 속성을 통해 값을 전달<h1 id="springboot에서-rabbitmq-사용-법">SpringBoot에서 RabbitMQ 사용 법</h1>
<h2 id="의존성-설정">의존성 설정</h2>
<pre><code class="language-java">// RabbitMQ
implementation 'org.springframework.boot:spring-boot-starter-amqp'
testImplementation 'org.springframework.amqp:spring-rabbit-test'
implementation 'org.springframework.boot:spring-boot-starter-reactor-netty:3.0.0'</code></pre>
</li>
</ol>
<h2 id="rabbitmq-yaml-설정">RabbitMQ Yaml 설정</h2>
<pre><code class="language-java">Spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    virtual-host: /</code></pre>
<h2 id="rabbitmq-설정">RabbitMQ 설정</h2>
<pre><code class="language-java">@EnableRabbit
@RequiredArgsConstructor
@Configuration
public class RabbitMQConfig {

    private static final String CHAT_QUEUE_NAME = &quot;chat.queue&quot;;
    private static final String CHAT_EXCHANGE_NAME = &quot;chat.exchange&quot;;
    private static final String CHAT_ROUTING_KEY = &quot;room.*&quot;;

    @Value(&quot;${spring.rabbitmq.host}&quot;)
    private String host;
    @Value(&quot;${spring.rabbitmq.port}&quot;)
    private int port;
    @Value(&quot;${spring.rabbitmq.username}&quot;)
    private String username;
    @Value(&quot;${spring.rabbitmq.password}&quot;)
    private String password;

    // Queue 등록
    @Bean
    public Queue queue() {
        return new Queue(CHAT_QUEUE_NAME);
    }

    // Exchange 설정 - TopicExchange
    @Bean
    public TopicExchange topicExchange(){
        return new TopicExchange(CHAT_EXCHANGE_NAME);
    }

    // Exchange와 Queue바인딩
    @Bean
    public Binding binding(){
        return BindingBuilder.bind(queue()).to(topicExchange()).with(CHAT_ROUTING_KEY);
    }

    // RabbitMQ 와의 연결을 관리하는 클래스
    @Bean
    public CachingConnectionFactory connectionFactory(){
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
        connectionFactory.setHost(host);
        connectionFactory.setPort(port);
        connectionFactory.setUsername(username);
        connectionFactory.setPassword(password);
        return connectionFactory;
    }

    // RabbitMQ 와의 메시지 통신을 담당하는 클래스
    @Bean
    public RabbitTemplate rabbitTemplate(){
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory());
        rabbitTemplate.setMessageConverter(new Jackson2JsonMessageConverter());
        return rabbitTemplate;
    }

    // RabbitMQ 메시지를 JSON 형식으로 보내고 받을 수 있음
    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}</code></pre>
<h2 id="websocket-설정">WebSocket 설정</h2>
<pre><code class="language-java">@EnableWebSocketMessageBroker
@RequiredArgsConstructor
@Configuration
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint(&quot;/ws/chat&quot;)
                .setAllowedOrigins(&quot;*&quot;)
                .withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.setApplicationDestinationPrefixes(&quot;/pub&quot;);
        registry.setPathMatcher(new AntPathMatcher(&quot;.&quot;));
        registry.enableStompBrokerRelay(&quot;/queue&quot;);
    }
}
</code></pre>
<ol>
<li>/queue: 개별 사용자에게 메세지를 보낼 때 사용 (예: 1:1채팅)</li>
<li>/topic: 다수의 구독자에게 메시지를 보낼 때 사용 (발행/구독 패턴)</li>
<li>/exchange: 메시지 라우팅 및 복잡한 메세지 처리가 필요할 때 사용</li>
<li>/amq/queue: 예약된 시스템 큐를 사용할 때 사용</li>
</ol>
<h2 id="constroller-구현">Constroller 구현</h2>
<pre><code class="language-java">@MessageMapping(&quot;chat.message.{roomId}&quot;) //여기로 전송되면 메서드 호출 -&gt; WebSocketConfig prefixes 에서 적용한건 앞에 생략
public ResponseEntity&lt;ChatResponse&gt; saveChat(@DestinationVariable(&quot;roomId&quot;) Long roomId, ChatRequest chatRequest){
    rabbitTemplate.convertAndSend(&quot;chat.exchange&quot;, &quot;room.&quot;+roomId, chatRequest);
    return ResponseEntity.ok(chatService.saveChatHistory(roomId, chatRequest));
}</code></pre>
<blockquote>
<p>주의 사항 
RabbitMQ가 기본적으로 STOMP를 지원하지 않기 때문에 STOMP 플러그인을 활성화해야 합니다.</p>
</blockquote>
<pre><code>rabbitmq-plugins enable rabbitmq_stomp</code></pre><h3 id="reference">Reference</h3>
<p><a href="https://velog.io/@black_han26/AMQPAdvanced-Message-Queuing-Protocol">https://velog.io/@black_han26/AMQPAdvanced-Message-Queuing-Protocol</a></p>
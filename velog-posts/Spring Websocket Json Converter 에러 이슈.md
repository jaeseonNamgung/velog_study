<h1 id="❌-문제-상황">❌ 문제 상황</h1>
<p>Client 와 Server간 통신 할 때 Json Converter 에러가 발생했다.</p>
<pre><code class="language-java">     @MessageMapping(&quot;/message/{chatRoomId}&quot;)
     @SendTo(&quot;/sub/chat/{chatRoomId}&quot;)
     public ChatMessageResponse sendMessage(
            @DestinationVariable(&quot;chatRoomId&quot;) Long chatRoomId,
            @Payload ChatMessageRequest chatMessageRequest,
            @Login Long senderId
             ) {
         log.debug(&quot;[sendMessage] chatRoomId: {}&quot;, chatRoomId);
         log.debug(&quot;[sendMessage] chatMessageRequest: {}&quot;, chatMessageRequest);
         log.debug(&quot;[sendMessage] senderId: {}&quot;, senderId);
         return chatService.sendMessage(chatRoomId, senderId, chatMessageRequest);
            }
</code></pre>
<pre><code class="language-java">Unhandled exception from message handler method
org.springframework.messaging.converter.MessageConversionException: Could not read JSON: 
Cannot deserialize value of type java.lang.Long from Object value (token JsonToken.START_OBJECT)
at [Source: REDACTED (StreamReadFeature.INCLUDE_SOURCE_IN_LOCATION disabled); line: 1, column: 1]</code></pre>
<h1 id="❗원인">❗원인</h1>
<p>Spring은 메서드의 파라미터를 주입할 때 <strong>HandlerMethodArgumentResolver</strong>라는 전략 패턴을 사용한다.</p>
<p><code>@Login</code> 어노테이션은 커스텀 어노테이션으로, <code>HttpServletRequest</code>의 <code>Authorization</code> 헤더에서 AccessToken을 추출해 사용자 ID(userId)로 변환하는 기능을 한다. 이 기능은 <strong>HTTP 통신에서 동작하는 HandlerMethodArgumentResolver</strong> 기반으로 구현되어 있다.</p>
<p>하지만 <code>@MessageMapping</code>을 사용하는 <strong>Spring WebSocket(STOMP)</strong> 환경에서는 HTTP 요청이 아닌 STOMP 메시지를 처리한다. 이 경우에는 HTTP용이 아닌 <strong>WebSocket 전용 HandlerMethodArgumentResolver</strong>가 작동하게 된다.</p>
<p>따라서 <code>@Login</code> 어노테이션은 <strong>WebSocket 환경에서는 작동하지 않으며</strong>, 이로 인해 <strong>accessToken을 userId로 변환하는 처리에서 오류가 발생한 것이다.</strong></p>
<h1 id="👨💻-문제-해결">👨‍💻 문제 해결</h1>
<pre><code class="language-java">        @MessageMapping(&quot;/message/{chatRoomId}&quot;)
    @SendTo(&quot;/sub/chat/{chatRoomId}&quot;)
    public ChatMessageResponse sendMessage(
            @DestinationVariable(&quot;chatRoomId&quot;) Long chatRoomId,
            @Payload ChatMessageRequest chatMessageRequest,
            @Header(&quot;Authorization&quot;) String accessToken
            ) {
        log.debug(&quot;[sendMessage] chatRoomId: {}&quot;, chatRoomId);
        log.debug(&quot;[sendMessage] chatMessageRequest: {}&quot;, chatMessageRequest);
        log.debug(&quot;[sendMessage] accessToken: {}&quot;, accessToken);
        return chatService.sendMessage(chatRoomId, accessToken, chatMessageRequest);
    }</code></pre>
<p>@Header 어노테이션을 사용해 Header 값을 받아서 accessToken을 처리하면 된다.</p>
<blockquote>
<p>주의!
@RequestHeader는 <strong>HTTP 요청</strong> 컨텍스트에서만 작동하므로 <strong>Spring WebSocket(STOMP)</strong> 환경에서 사용 불가능하다.</p>
</blockquote>
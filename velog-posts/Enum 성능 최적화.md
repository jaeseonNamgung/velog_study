<p>Enum에서 특정 문자열을 기준으로 해당하는 Enum 값을 찾는 메서드를 자주 사용합니다. 예를 들어, find 또는<code>findByValue</code>와 같은 메서드를 통해 문자열 값과 Enum 값을 매칭합니다. Stream API를 사용하여 Enum 값을 찾는 방식은 간단하고 직관적이지만, 반복적으로 호출될 경우 성능 상의 문제가 발생할 수 있습니다. 이에 Jay 님이 작성한 Enum 최적화 방법을 참고하여, <code>HashMap</code>을 사용하여 값을 캐싱하고 빠르게 찾는 방법을 기록하고자 합니다</p>
<p>Enum에서 특정 값(예: 문자열)을 기준으로 Enum 인스턴스를 찾는 방법 3가지 예시를 보여드리겠습니다. 각각의 방법은 Stream과 HashMap을 활용한 구현으로 구성됩니다.</p>
<h2 id="arraysstream-사용">Arrays.stream() 사용</h2>
<pre><code class="language-java">@RequiredArgsConstructor
@Getter
public enum Status {
  ACTIVE(&quot;활성화&quot;),
  INACTIVE(&quot;비활성&quot;),
  SUSPENDED(&quot;정지&quot;),
  UNKNOWN(&quot;알수없음&quot;);

  private final String description;

  public static Status find(String description) {
    return Arrays.stream(values())
        .filter(value -&gt; value.description.equals(description))
        .findAny()
        .orElse(Status.UNKNOWN);
  }
}</code></pre>
<h2 id="streamof-사용">Stream.of() 사용</h2>
<pre><code class="language-java">@RequiredArgsConstructor
@Getter
public enum Status {
  ACTIVE(&quot;활성화&quot;),
  INACTIVE(&quot;비활성&quot;),
  SUSPENDED(&quot;정지&quot;),
  UNKNOWN(&quot;알수없음&quot;);

  private final String description;

  public static Status find(String description) {
    return Stream.of(values())
        .filter(value-&gt; value.description.equals(description))
        .findAny()
        .orElse(Status.UNKNOWN);
  }
}</code></pre>
<h2 id="hashmap-사용-→-가장-빠름">HashMap 사용 → 가장 빠름</h2>
<pre><code class="language-java">@RequiredArgsConstructor
@Getter
public enum Status {
  ACTIVE(&quot;활성화&quot;),
  INACTIVE(&quot;비활성&quot;),
  SUSPENDED(&quot;정지&quot;),
  UNKNOWN(&quot;알수없음&quot;);

  private final String description;

  private static final Map&lt;String, Status&gt; descriptions = Collections.unmodifiableMap(Stream.of(values()).collect(Collectors.toMap(Status::getDescription, Function.identity())));

  public static Status find(String description) {
    return Optional.ofNullable(descriptions.get(description)).orElse(UNKNOWN);
  }
}</code></pre>
<p>위 코드는 다음과 같은 방식으로 동작합니다:</p>
<ol>
<li><p><strong>불변 <code>Map</code> 생성</strong></p>
<p> <code>descriptions</code>는 <code>description</code>을 키로, <code>Status</code>를 값으로 하는 불변 <code>Map</code>입니다. 이를 생성하기 위해 <code>Collectors.toMap</code>을 사용하고, 외부에서 수정할 수 없도록 <code>Collections.unmodifiableMap</code>으로 감쌉니다. </p>
</li>
<li><p><strong><code>Collectors.toMap</code> 사용</strong></p>
<ul>
<li>첫 번째 매개변수는 <code>Status::getDescription</code>으로, 각 <code>Status</code>의 <code>description</code> 값을 키로 사용합니다.</li>
<li>두 번째 매개변수는 <code>Function.identity()</code>로, 각 <code>Status</code> 인스턴스 자체를 값으로 사용합니다.</li>
</ul>
</li>
</ol>
<h1 id="성능-측정">성능 측정</h1>
<p>각각의 메서드에서 동일한 값(예: 특정 문자열)을 기준으로 Enum 값을 찾는 작업을 1억 번 반복하여 성능을 측정하였습니다.  </p>
<pre><code class="language-java">@Test
  void enum_속도_측정_stream_of_이용() {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    for (int i = 0; i &lt; 100000000; i++) {
      Status.findWithStreamOf(&quot;활성화&quot;);
    }
    for (int i = 0; i &lt; 100000000; i++) {
      Status.findWithStreamOf(&quot;unexpected value&quot;);
    }

    stopWatch.stop();
    System.out.println(&quot;values: &quot; + Status.findWithStreamOf(&quot;활성화&quot;));
    System.out.println(&quot;Elapsed Time (Stream.of 이용) : &quot; + stopWatch.getTotalTimeSeconds() + &quot;s&quot;);
  }

  @Test
  void enum_속도_측정_array_stream_이용() {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    for (int i = 0; i &lt; 100000000; i++) {
      Status.findWithArraysStream(&quot;활성화&quot;);
    }
    for (int i = 0; i &lt; 100000000; i++) {
      Status.findWithArraysStream(&quot;unexpected value&quot;);
    }

    stopWatch.stop();
    System.out.println(&quot;values: &quot; + Status.findWithArraysStream(&quot;활성화&quot;));
    System.out.println(&quot;Elapsed Time (Stream.of 이용) : &quot; + stopWatch.getTotalTimeSeconds() + &quot;s&quot;);
  }

  @Test
  void enum_속도_측정_hash_map_이용() {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    for (int i = 0; i &lt; 100000000; i++) {
      Status.findWithHashMap(&quot;활성화&quot;);
    }
    for (int i = 0; i &lt; 100000000; i++) {
      Status.findWithHashMap(&quot;unexpected value&quot;);
    }

    stopWatch.stop();
    System.out.println(&quot;values: &quot; + Status.findWithHashMap(&quot;활성화&quot;));
    System.out.println(&quot;Elapsed Time (Stream.of 이용) : &quot; + stopWatch.getTotalTimeSeconds() + &quot;s&quot;);
  }</code></pre>
<h2 id="측정-결과">측정 결과</h2>
<p><img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/40084c71-3650-45b2-ba49-3fea3be1ea88/image.png" /></p>
<p>성능 결과를 보면, <code>Array.stream</code>과 <code>Streams.of</code> 방식은 속도 차이가 미미했으나, <code>HashMap</code>을 사용한 방식은 두 방법에 비해 약 <strong>10배 이상 빠른 결과</strong>를 보였습니다.  다음 프로젝트 부터 Enum 사용 시 HashMap 사용을 고려해봐도 좋을 거 같습니다!</p>
<h3 id="reference">Reference</h3>
<p><a href="https://pjh3749.tistory.com/279">https://pjh3749.tistory.com/279</a></p>
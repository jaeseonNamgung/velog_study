<h1 id="aws-기초-elb--asg">AWS 기초: ELB + ASG</h1>
<h2 id="고가용성-및-확장성">고가용성 및 확장성</h2>
<ul>
<li>확장성: 확장성은 애플리케이션 시스템이 조정을 통해 더 많은 양을 처리할 수 있다는 의미이다. (규모를 늘림)<ul>
<li>수직 확장성: 인스턴스의 크기를 확장 <ul>
<li>aws 용어: 확장 -&gt; scale up, 축소 -&gt; scale down</li>
<li>데이터베이스와 같이 분산되지 않은 시스템에서 흔히 사용</li>
<li>일반적으로 확장할 수 있는 정도에는 한계가 있다. (하드웨어 제한)</li>
</ul>
</li>
<li>수평 확장성(= 탄력성): 애플리케이션에서 인스턴스나 시스템의 수를 
늘리는 방법<ul>
<li>aws 용어: 수를 늘림 -&gt; scale out, 수를 줄임 -&gt; scale in</li>
<li>수평 확장을 했다는 건 분배 시스템이 있다는걸 의미(웹이나 현대적 애플리케이션)</li>
</ul>
</li>
</ul>
</li>
<li>고가용성(High Avaliability, HA): 고가용성이란 애플리케이션 또는 시스템을 적어도 둘 이상의 AWS의 AZ나 데이터 센터에서 가동 중이라는 걸 의미 <ul>
<li>고가용성은 보통 수평 확장과 함께 사용되는 개념(늘 그런 것은 아님)</li>
<li>고가용성의 목표는 데이터 센터에서의 손실에서 살아남는 것으로 센터 하나가 멈춰도 계속 작동이 가능하게 끔 하는 것이다. </li>
</ul>
</li>
</ul>
<h1 id="로드-밸런싱load-balancing">로드 밸런싱(Load balancing)</h1>
<ul>
<li>로드 밸런서(Load balances)는 서버 혹은 서버셋으로 트랙픽을 백엔드나 다운스트림 EC2 인스턴스 또는 서버들로 전달하는 역할을 한다. <h2 id="로드-밸런서가-필요한-이유">로드 밸런서가 필요한 이유</h2>
부하를 다수의 다운스트림 인스턴스로 분산하기 위해서이다. <ul>
<li>애플리케이션에 단일 액세스 지점(DNS)을 노출하게 되고 다운스트림 인스턴스의 장애를 원할히 처리할 수 있다.</li>
<li>인스턴스의 상태를 확인할 수 있다.</li>
<li>SSL 종료 가능</li>
<li>웹 사이트에 암호화된 HTTPS 트래픽을 가질 수 있다. </li>
<li>쿠키로 고정도를 강화할 수 있고 영역에 걸친 고가용성을 가질 수 있으며 클라우드 내에서 개인 트래픽과 공공 트래픽을 분리할 수 있다. </li>
</ul>
</li>
</ul>
<h2 id="일래스틱-로드-밸런서elastic-load-balancer">일래스틱 로드 밸런서(Elastic Load Balancer)</h2>
<ul>
<li>일래스틱 로드 밸런서는 관리형 로드 밸런서이다. </li>
<li>AWS가 관리하며, 어떤 경우에도 작동할 것을 보장한다.</li>
<li>AWS가 업그레이드와 유지 관리 및 고가용성을 책임지며 로드 밸런서의 작동 방식을 수정할 수 있게 끔 일부 구성 놉(Knobs)도 제공한다.</li>
<li>로드 밸런서는 다수의 AWS의 서비스들과 통합되어 있다. (EC2 인스턴스와도 통합이 가능)</li>
</ul>
<h3 id="상태-확인health-checks">상태 확인(Health Checks)</h3>
<p>상태 확인은 일래스틱 로드 밸런서가 EC2 인스턴스의 작동이 올바르게 되고 있는지의 여부를 확인하기 위해 사용된다. 만약 제대로 작동하는 중이 아니라면 해당 인스턴스로는 트래픽을 보낼 수 없다. (상태 확인은 포트와 라우트에서 이뤄진다.)</p>
<h2 id="aws-로드-밸런서-종류">AWS 로드 밸런서 종류</h2>
<ol>
<li><p>클래식 로드 밸런서(Classc Load Balancer): 기존 세대나 V1이라고도 하며 2009년에 만들어졌다. CLB라도 불린다.</p>
<ul>
<li>HTTP, HTTPS, TCP, SSL와 secure TCP를 지원</li>
<li>AWS에서는 클래식 로드 밸런서 사용을 권장하지 않음</li>
<li>AWS에서 더 이상 사용되지 않는다. (콘솔에서도 X) </li>
</ul>
</li>
<li><p>애플리케이션 로드 밸런서(Application Load Balancer): 2016년에 출시된 애플리케이션 로드 밸런서로 ALB라고 불린다. (V2)</p>
<ul>
<li>HTTP, HTTPS와 WebSocket 프로토콜을 지원 (7계층)</li>
<li>리다이렉트 지원(HTTP에서 HTTPS로 트래픽을 자동 리다이렉트하려는 경우)</li>
<li>머신 간 다수 HTTP 애플리케이션의 라우팅에 사용(target groups)</li>
<li>동일 EC2 인스턴스 상의 여러 애플리케이션에 부하를 분산</li>
<li>URL의 기본 경로 라우팅 지원(example.com/users, example.com/posts)</li>
<li>URL의 호스트 이름에 기반한 라우팅 지원(one.example.com, other.example.com)</li>
<li>쿼리 문자열과 헤더에 기반한 라우팅 지원(example.com/user?id=123&amp;order=false)</li>
</ul>
</li>
<li><p>네트워크 로드 밸런서(Network Load Balancer): 2017년에 출시된 네트워크 로드 밸런서이고 NLB라고 불린다. (V2)</p>
<ul>
<li>TCP, TLS, secure TCP와 UDP 프로토콜을 지원</li>
</ul>
</li>
<li><p>케이트웨이 로드 밸런서(Gateway Load Balancer): 2020년에 나온 게이트웨이 로드 밸런서이고 GWLB라고 불린다. </p>
<ul>
<li>3계층과 IP프로토콜에서 작동 (네트워크층에서 작동)</li>
</ul>
</li>
</ol>
<h2 id="로드-밸런서-보안-그룹">로드 밸런서 보안 그룹</h2>
<ul>
<li>유저는 HTTP나 HTTPS를 사용해 어디서든 로드 밸런서에 접근이 가능
<img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/8902d4e6-382f-44d4-8915-feb33b5ededc/image.png" /></li>
<li>EC2 인스턴스는 로드 밸런서를 통해 곧장 들어오는 트래픽만 허용
<img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/95763cc2-a5a8-4816-8b8f-b57bbeb0cf20/image.png" /></li>
</ul>
<h2 id="애플리케이션-로드-밸런서-대상-그룹target-groups">애플리케이션 로드 밸런서 대상 그룹(Target Groups)</h2>
<ul>
<li>대상 그룹이 될 수 있는 것들: <ul>
<li>EC2 <ul>
<li>오토 스케일링 그룹에 의해 관리 될 수 있다. </li>
</ul>
</li>
<li>ECS </li>
<li>람다 함수</li>
<li>IP 주소 - 반드시 사설 IP 주소여야 한다.</li>
</ul>
</li>
<li>ALB는 여러 대상 그룹으로 라우팅할 수 있다.</li>
<li>상태 확인은 대상 그룹 레벨에서 이뤄진다.</li>
</ul>
<h1 id="network-load-balancerv2-nlb">Network Load Balancer(v2, NLB)</h1>
<ul>
<li>4계층 로드 밸런서 이므로 TCP와 UDP 트래픽을 처리할 수 있다.</li>
<li>네트워크 로드 밸런서는 굉장히 고성능이다.<ul>
<li>초당 수백만 건의 요청을 처리할 수 있다.</li>
<li>지연 시간도 애플리케이션 로드 밸런서 대비 짧다.</li>
</ul>
</li>
<li>가용 영역 당 하나의 고정 IP만 있다.</li>
<li>각 가용 영역에 탄력적 IP를 배정할 수 있다.<ul>
<li>여러 고정 IP가 있는 애플리케이션을 노출해야 할 때 무척 유용</li>
<li>만약 시험에 애플리케이션 1개, 혹은 서로 다른 2개, 3개의 IP로만 액세스 가능하다고 하면 NLB를 떠올려야 한다.</li>
<li>고도의 성증이나 TCP, UDP, 고정 IP를 보면 네트워크 로드 밸런서를 떠올려야 한다.</li>
</ul>
</li>
<li>NLB는 AWS 프리 티어에는 포함돼 있지 않는다.</li>
</ul>
<h2 id="네트워크-로드-밸런서---대상-그룹">네트워크 로드 밸런서 - 대상 그룹</h2>
<ul>
<li>EC2 인스턴스</li>
<li>IP 주소 - private IP 주소여야 한다.</li>
<li>애플리케이션 로드 밸런서 앞에 NLB를 둘 수 있다.</li>
<li>NLB의 대상 그룹이 하는 상태 확인은 세가지 프로토콜을 지원한다.<ul>
<li>TCP 프로토콜, HTTP 프로토콜, HTTPS 프로토콜</li>
</ul>
</li>
</ul>
<h1 id="gateway-load-balancergwlb">Gateway Load Balancer(GWLB)</h1>
<ul>
<li>GWLB는 배포 및 확장과 AWS의 타사 네트워크 가상 어플라이언스의 플릿 관리에 사용된다.</li>
<li>GWLB는 네트워크의 모든 트래픽이 방화벽을 통과하게 하거나 침입 탐지 및 방지 시스템에 사용된다. <ul>
<li>그래서 IDPS나 심층 패킷 분석 시스템 또는 일부 페이로드를 수정할 수 있지만 네트워크 수준에서 가능하다.</li>
</ul>
</li>
<li>GWLB는 모든 로드 밸런서보다 낮은 수준에서 실행된다.<ul>
<li>IP 패킷의 네트워크 계층인 L3이다.</li>
</ul>
</li>
</ul>
<h2 id="gwlb-기능">GWLB 기능</h2>
<ul>
<li>투명 네트워크 게이트웨이이다.<ul>
<li>VCP의 모든 트래픽이 GWLB가 되는 단일 엔트리와 출구를 통과하기 때문이다.</li>
<li>대상 그룹의 가상 어플라이언스 집합에 전반적으로 그 트래픽을 분산해 로드 밸런서가 된다.</li>
</ul>
</li>
<li>시험볼 때 6081번 포트의 GENEVE 프로토콜을 사용해야한다.</li>
</ul>
<h2 id="gwlb-대상-그룹">GWLB 대상 그룹</h2>
<ul>
<li>EC2 인스턴스</li>
<li>IP 주소 - private IP, 개인 IP 이여야 한다.</li>
</ul>
<h1 id="sticky-sessions-session-affinity">Sticky Sessions (Session Affinity)</h1>
<ul>
<li>고정성 혹은 고정 세션을 실행하는 것으로 그 개념은 로드 밸런서에 2가지 요청을 수행하는 클라이언트가 요청을 응답하기 위해 백엔드에 동일한 인스턴스를 갖는 것이다.</li>
<li>CLB와 ALB에서도 설정 가능하다.</li>
<li>쿠키는 클라이언트에서 로드 밸런서로 요청의 일부로서 전송되는 것이다.</li>
<li>고정성과 만료 기간이 존재<ul>
<li>쿠키가 만료되면 클라이언트가 다른 EC2 인스턴스로 리다이렉션 된다. </li>
</ul>
</li>
<li>세션 만료를 사용 시에는 사용자의 로그인과 같은 중요한 정보를 취하는 세션 데이터를 잃지 않기 위해 사용자가 동일한 백엔트 인스턴스에 연결된다.  </li>
<li>고정성을 활성화 하면 백엔드 EC2 인스턴스 부하에 불균형을 초래할 수 있다. <ul>
<li>일부 인스턴스는 고정 사용자를 갖게 된다.</li>
</ul>
</li>
</ul>
<h2 id="sticky-sessions---cookie-names">Sticky Sessions - Cookie Names</h2>
<ul>
<li><p>고정 세션에는 2가지 유형의 쿠키가 있다.</p>
<ul>
<li><p>애플리케이션 기반 쿠키</p>
<ul>
<li>사용자 정의 쿠키</li>
<li>애플리케이션에서 생성된다.</li>
<li>애플리케이션에 필요한 모든 사용자 정의 속성을 포함할 수 있다.</li>
<li>쿠키 이름은 각 대상 그룹별로 개별적으로 지정해야 한다.<ul>
<li>AWSALB, AWSALBAPP, AWSALBTG 같은 이름으로 사용하면 안된다. (ELB에서 사용하기 때문)</li>
<li>애플리케이션 쿠키</li>
<li>로드 밸런서 자체에서 생성</li>
<li>쿠키 이름: AWSALBAPP</li>
</ul>
</li>
</ul>
</li>
<li><p>기간 기반 쿠키</p>
<ul>
<li>로드 밸런서에서 생성된 쿠키</li>
<li>ALB에서 이름: AWSALB</li>
<li>CLB에서 이름: AWSELB</li>
<li>특정 기간을 기반으로 만료되며 그 기간이 로드 밸러서 자체에서 생성된다.</li>
</ul>
</li>
</ul>
</li>
</ul>
<h1 id="cross-zone-load-balancing">Cross-Zone Load Balancing</h1>
<ul>
<li>애플리케이션 로드 밸런서일 경우<ul>
<li>기본적으로 교차 영역 로드 밸런싱이 활성화되어 있다.</li>
<li>비활성화는 대상 그룹 레벨에서 가능하다.</li>
<li>데이터가 가용 영역으로 넘어가도 요금이 부과되지는 않는다.</li>
</ul>
</li>
<li>네트워크 로드 밸런서 &amp; 게이트웨이 로드 밸런서<ul>
<li>기본적으로 교차 영역 로드 밸런싱은 비활성화되어 있다.</li>
<li>활성화시 요금이 부과된다.</li>
</ul>
</li>
</ul>
<h1 id="ssltls---기본">SSL/TLS - 기본</h1>
<ul>
<li>SSL 인증서는 클라이언트와 로드 밸런서 사이에서 트래픽이 전송되는 동안 암호화 되도록 해준다.<ul>
<li>즉 데이터가 네트워크를 통과하는 중에는 암호화 되어 있고 발신자와 수신자에만 해독할 수 있다.</li>
</ul>
</li>
<li>SSL은 '보안 소켓 계층'이라는 뜻이며, 연결을 암호화할 때 사용된다.</li>
<li>TLS는 SSL의 새 버전이고 <code>전송 계층 보안</code>을 의미</li>
<li>Public SSL 인증서는 인증 기관에서 발행하고, 이 기관에는 Comodo, Symantec, GoDaddy, Globalsign, Digicert등이 있다.</li>
<li>로드 밸런서에 첨부된 퍼블릭 SSL 인증서를 활용하면 클라이언트와 로드 밸런서 사이의 연결을 암호화할 수 있다.</li>
<li>SSL 인증서에는 사용자가 정한 만료 날짜가 있으며, 인증서의 진위를 확인하기 위해 주기적으로 갱신해야 한다.</li>
</ul>
<h2 id="로드-밸런서---ssl-인증서">로드 밸런서 - SSL 인증서</h2>
<p><img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/143c45c3-dc7d-4253-a0c8-9f6d5cbd545e/image.png" /></p>
<ul>
<li>로드 밸런서는 X.509 인증서를 로딩하는데, SSL 혹은 TLS 서버 인증서라  불린다.</li>
<li>AWS에서는 ACM(AWS 인증서 매니저)을 활용해 SSL 인증서를 관리할 수 있다.<ul>
<li>원하면 ACM에 인증서를 업로드 할 수도 있다.</li>
</ul>
</li>
<li>HTTPS 리스너:<ul>
<li>HTTPS 리스너를 설정할 때는 기본 인증서를 지정해줘야 한다.</li>
<li>인증서 목록을 추가해 여러 도메인을 지원할 수 있다. (옵션)</li>
<li>클라이언트는 SNI(서버 이름 표시)라는 걸 활용해 도달하려는 호스트 이름을 지정할 수 있다.</li>
<li>원하는 경우에는 HTTPS에 특정 보안 정책을 설정해서 레거시 클라이언트라고도 부르는 SSL과 TLS의 구버전을 지원할 수도 있다.</li>
</ul>
</li>
</ul>
<h1 id="ssl---server-name-indicationsni">SSL - Server Name Indication(SNI)</h1>
<ul>
<li>SNI는 무척 중요한 문제를 해결하는데 바로 하나의 웹 서버에 여러 SSL 인증서를 로드해 웹 서버가 여러 웹사이트를 지원하게 만드는 방법이다.</li>
<li>클라이언트가 초기 SSL 핸드셰이크에서 대상 서버의 호스트 이름을 표시해야 하는 새 프로토콜이다.</li>
<li>클라이언트가 '이 웹 사이트에 연결하고 싶다'고 하면 서버는 어떤 인증서를 로드해야 하는지 알 수 있다.</li>
</ul>
<blockquote>
<p>모든 클라이언트가 SNI를 지원하는 건 아니다. 더 최신 세대인 ALB와 NLB를 활용할 때만 적용된다. 혹은 CloudFront에서 적용된다. (CLB 적용 X)</p>
</blockquote>
<h2 id="ssl-인증서-지원">SSL 인증서 지원</h2>
<ul>
<li><p>Classic Load Balancer(v1) : </p>
<ul>
<li>클래식 로드 밸런서는 하나의 SSL 인증서만 지원 가능</li>
<li>여러 SSL 인증서로 여러 호스트 이름을 사용하고 싶으면 클래식 로드 밸런서를 여러 개 사용하면 된다.</li>
</ul>
</li>
<li><p>Application Load Balancer(v2):</p>
<ul>
<li>여러 SSL 인증서로 여러 리스너를 지원할 수 있다.</li>
<li>SNI 활용 가능</li>
</ul>
</li>
<li><p>Network Load Balancer(v2)</p>
<ul>
<li>여러 SSL 인증서로 여러 리스너를 지원할 수 있다.</li>
<li>SNI 활용 가능</li>
</ul>
</li>
</ul>
<h1 id="elb-connection-draining연결-드레이닝">ELB Connection Draining(연결 드레이닝)</h1>
<ul>
<li>연결 드레이닝 naming:<ul>
<li>클래식 로드 밸런서를 사용할 경우: 연결 드레이닝</li>
<li>애플리케이션 밸런서나 네트워크 로드 밸런서를 사용하는 경우: 등록 취소 지연</li>
</ul>
</li>
<li>인스턴스가 등록 취소, 혹은 비정상인 상태에 있을 때 인스턴스에 어느 정도의 시간을 주어 인-플라이트 요청, 즉 활성 요청을 완료할 수 있도록 하는 기능이다.</li>
<li>연결이 드레이닝 되면 즉 인스턴스가 드레이닝 되면 ELB는 등록 취소 중인 EC2 인스턴스로 새로운 요청을 보내지 않는다.</li>
<li>연결 드레이닝 파라미터는 매개변수로 표시할 수 있다.<ul>
<li>1부터 3,600초 사이의 값으로 설정할 수 있다. (기본적으로 300초 즉 5분이다.)</li>
<li>값을 0으로 설정하면 전부 다 비활성화 된다.</li>
</ul>
</li>
</ul>
<h1 id="오토-스케일링-그룹">오토 스케일링 그룹</h1>
<p>웹 사이트나 애플리케이션을 배포하면 시간이 지나면서 로드가 변할 수 있다. 점점 많은 사람이 방문할 수 있기 때문이다.
AWS에서는 EC2 인스턴스 생성 API 호출로 아주 빠르게 서버를 생성하고 제거할 수 있다. 이를 자동화 하고 싶을 때 오토 스케일링 그룹(ASG)를 생성한다.</p>
<ul>
<li><p>ASG의 목표</p>
<ul>
<li>스케일 아웃: EC2 인스턴스를 추가해서 늘어난 로드에 맞추는 것</li>
<li>스케일 인: 줄어든 로드에 맞추기 위해 EC2 인스턴스를 제거하는 것</li>
<li>그래서 ASG의 크기는 시간이 지나면서 달라진다. </li>
<li>전체적으로 매개변수를 정의해서 ASG에서 언제든 실행할 수 있는 최소 및 최대 EC2 인스턴스 수를 정할 수 있다. </li>
<li>ASG의 강력한 기능 중 하나로 로드 밸런서와 연결하면 ASG에 포함된 EC2 인스턴스가 로드 밸런서에 연결된다.</li>
<li>또 다른 기능으로 어떤 인스턴스가 비정상이라고 여겨지면 이것을 종료하고 새 EC2 인스턴스를 만들어 대체한다. </li>
</ul>
</li>
<li><p>오토 스케일링 그룹은 무료이며, 그 아래에 생성되는 EC2 인스턴스와 같은 리소스에 대해서만 비용을 내면 된다.</p>
</li>
</ul>
<h2 id="오토-스케일링-그룹-속성">오토 스케일링 그룹 속성</h2>
<ul>
<li>ASG Launch Template<ul>
<li>AMI + 인스턴스 타입</li>
<li>EC2 사용자 데이터</li>
<li>EBS 볼륨</li>
<li>보안 그룹</li>
<li>SSH Key Pair</li>
<li>IAM 역할</li>
<li>네트워크와 서브넷 정보</li>
<li>로드 밸런서 정보 등</li>
</ul>
</li>
<li>ASG 최소 크기, 최대 크기, 초기 용량과 스케일링 정책을 정의</li>
<li>스케일링 정책</li>
</ul>
<h2 id="오토-스케일링-그룹---오토-스케일링-정책">오토 스케일링 그룹 - 오토 스케일링 정책</h2>
<h3 id="동적-스케일링-정책">동적 스케일링 정책</h3>
<ul>
<li>대상 추척 스케일링:<ul>
<li>가장 단순하고 설정하기도 쉽다.</li>
<li>예: 모든 EC2 인스턴스에서 오토 스케일링 그룹의 평균 CPU 사용률을 추적하여 이 수치가 40%대에 머무를 수 있도록 할 때 사용한다.</li>
</ul>
</li>
<li>단순과 단계 스케일링:<ul>
<li>CloudWatch 경보를 설정 (전체 ASG에 대한 CPU 사용률이 70%를 초과하는 경우 용량을 두 유닛 추가하도록 설정할 수 있다.)</li>
</ul>
</li>
<li>예약된 작업:<ul>
<li>나와 있는 사용 패턴을 바탕으로 스케일링을 예상</li>
</ul>
</li>
</ul>
<h3 id="예측-스케일링-정책">예측 스케일링 정책</h3>
<ul>
<li>예측 스케일링 정책: AWS 내 오토 스케일링 서비스를 활용하여 지속적으로 예측을 생성할 수 있다.</li>
</ul>
<h2 id="스케일업하기에-좋은-메트릭">스케일업하기에 좋은 메트릭</h2>
<ul>
<li>CPU 활용률: 인스턴스 전체의 평균 CPU 활용률</li>
<li>RequestCountPerTarget: EC2 인스턴스당 요청 수가 안정적인지 확인
• 평균 네트워크 입력/출력(애플리케이션이 네트워크 바인딩된 경우)</li>
<li>사용자 지정 메트릭(CloudWatch를 사용하여 푸시하는 메트릭)</li>
</ul>
<h2 id="자동-스케일링-그룹---스케일링-냉각-시간">자동 스케일링 그룹 - 스케일링 냉각 시간</h2>
<ul>
<li>스케일링 활동이 발생하면 냉각 시간(기본값 300초)이 된다.</li>
<li>냉각 기간 동안 ASG는 추가 인스턴스를 시작하거나 종료하지 않는다.(메트릭이 안정화될 수 있도록 허용)</li>
<li>조언: 즉시 사용할 수 있는 AMI를 사용하여 구성 시간을 단축하여 요청을 더 빠르게 처리하고 재사용 대기 시간을 단축해야 한다.</li>
</ul>
<h2 id="auto-scaling---instance-refresh">Auto Scaling - Instance Refresh</h2>
<ul>
<li>목표: 시작 템플릿을 업데이트한 다음 모든 EC2 인스턴스를 다시 만든다.<ul>
<li>이를 위해 인스턴스 새로 고침의 기본 기능을 사용할 수 있다.</li>
<li>최소 상태 백분율 설정</li>
<li>준비 시간 지정(인스턴스를 사용할 준비가 될 때까지의 시간)</li>
</ul>
</li>
</ul>
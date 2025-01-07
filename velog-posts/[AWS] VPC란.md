<h1 id="vpc란">VPC란</h1>
<p><img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/a3121a81-9d48-4de3-8a63-731e8e875cec/image.png" /></p>
<p>VPC(virture private cloud)란 클라우드 환경에서의 가상 네트워크 망이 퍼블릭과 프라이빗의 논리적으로 독립된 네트워크 공간으로 구성된 것을 말한다. </p>
<p>vpc 안에는 퍼블릭 서브넷 , 프라이빗 서브넷, 라우트 테이블, 인터넷 게이트웨이 등이 구성된다. </p>
<p>VPC를 생성하기 위해서는 VPC 아이피 범위를 RFC1918 사설 아이피 대역에 맞추어 구축해야 한다. 사설 아이피는 VPC안에서 독립적으로 사용하는 아이피이다. </p>
<p>사설 아이피 대역: </p>
<ul>
<li>10.0.0.0 ~ 10.255.255.255(10/8 prefix)</li>
<li>172.16.0.0 ~ 172.31.255.255(182.16/12 prefix)</li>
<li>192.168.0.0 ~ 192.168.255.255(192.168/16 prefix)</li>
</ul>
<p>한 번 설정된 아이피는 수정할 수 없으며 각 VPC는 하나의 리전에 종속된다. </p>
<h2 id="서브넷">서브넷</h2>
<p>서브넷은 VPC에서 작은 단위로 나눈 개념이다.  서브넷은 퍼블릭 서브넷, 프라이빗 서브넷으로 나뉜다. </p>
<ul>
<li>퍼블릭 서브넷: 퍼블릭 서브넷은 외부 인터넷과 통신이 가능한 서브넷이다.</li>
<li>프라이빗 서브넷: 프라이빗 서브넷은 외부 인터넷과는 통신이 불가능하고 같은 VPC 안에 서브넷과만 통신이 가능하다.</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/425ac55b-7007-464c-9c66-ef82e1af3db0/image.png" /></p>
<h2 id="라우터">라우터</h2>
<p>라우터는 네트워크 통신을 수행할 때 어느 네트워크로 갈지 경로를 정해주는 역할을 라우팅이라고 하며,  라우팅을 하기 위한 장비를 라우터라고 한다. </p>
<p>** VPC 생성 시 자동으로 가상 라우터가 생성된다. ** </p>
<h2 id="라우팅-테이블">라우팅 테이블</h2>
<p>라우팅 테이블은 라우팅을 하기 위해 정보를 저장하는 테이블이다. </p>
<p>** VPC생성 시 자동으로 가상 라우팅 테이블이 생성된다. **</p>
<h2 id="게이트웨이">게이트웨이</h2>
<p>VPC와 인터넷이 통신하기 위해서는 게이트웨이를 거쳐야지만 통신이 가능하다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/606cf1a8-b7eb-488b-831c-967f19f1b66f/image.png" /></p>
<ul>
<li>인터넷 게이트웨이: 인터넷 구간으로 나가는 관문, VPC당 1개만 연결 가능</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/3112d724-3009-49b3-a577-bc390ecea6cf/image.png" /></p>
<ul>
<li>NAT 게이트웨이: 프라이빗 서브넷이 인터넷과 통신이 필요할 때 퍼블릭 서브넷상에서 NAT 게이트웨이를 통해 인터넷 게이트웨이와 통신 할 수 있다. (주로 펌웨어나 혹은 주기적인 업데이트가 필요할 때 아웃바운드 트래픽만 혀용 되야 할 때 사용된다. )</li>
</ul>
<h2 id="네트워크-acl-보안그룹">네트워크 ACL, 보안그룹</h2>
<p>네트워크 ACL과 보안그룹은 방화벽과 같은 역할을 하며 인바운드 트래픽과 아웃바운드 트래픽 보안 정책을 설정할 수 있다. 우선 네트워크 ACL은 Stateless이며 모든 트래픽을 처리하기 위해 명시적인 규칙을 따른다.  네트워크 ACL은 모든 트래픽을 허용하도록 기본 설정되어 있기 때문에 불필요한 트래픽을 차단하도록 따로 설정해야 된다. 네트워크 ACL은 주로 서브넷 수준에서 구현된다. </p>
<p>보안그룹은 Statefull이며 인바운드 규칙에 대한 아웃바운드 응답을 자동으로 허용한다. (아웃바운드가 none 이여도 인바운드에서 정의된 규칙이 허용되면 아웃바운드가 none이여도 허용된다.)  보안그룹은 모든 트래픽의 접근을 차단되도록 기본 설정되어 있기 때문에 필요한 설정은 허용해줘야 된다.  보안그룹은 주로 인스턴스(EC2) 수준에서 구현된다. </p>
<h3 id="refremce">Refremce</h3>
<p>aws 공식 문서</p>
<p><a href="https://velog.io/@moonblue/vpc%EB%9E%80">https://velog.io/@moonblue/vpc란</a></p>
<p><a href="https://medium.com/harrythegreat/aws-%EA%B0%80%EC%9E%A5%EC%89%BD%EA%B2%8C-vpc-%EA%B0%9C%EB%85%90%EC%9E%A1%EA%B8%B0-71eef95a7098">https://medium.com/harrythegreat/aws-가장쉽게-vpc-개념잡기-71eef95a7098</a></p>
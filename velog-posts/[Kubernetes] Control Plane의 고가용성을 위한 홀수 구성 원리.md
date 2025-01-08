<p>kubelet: 노드에 배포되는 에이전트로 kubernetes 마스터 노드의 API 서버와 통신하며 노드가 수행해야 할 명령을 받아 실행하고 노드의 상태등을 마스터 노드의 전달한다. 
kubectl: kubernetes 상태를 확인하고 원하는 상태를 요청한다. 즉 Cluster를 관리하는 명령어 tool이다. (예: get, describe, edit, delete 등)
kubeadm: Cluster 생성을 위한 도구로 kubeadm init, kubeadm join 등을 이용해 master, worker node에 Cluster 환경을 구축한다. </p>
<h1 id="control-plane">Control Plane</h1>
<p>Control plane은 노드를을 관리하는 중요한 기능을한다. 만약 Control plane이 문제가 생기면 다른 노드에서도 문제가 생기기 때문에 고가용성을 위해서 여러 개의 CP를 구성하게 되는데 여러 개의 CP를 구성할 때는 홀수 개로 구성하게 된다. </p>
<p>Kubernetes와 같은 분산 시스템에서는 CP를 홀수 개로 구성된다. 그 이유는 Quorum 기반의 분산 합의 알고리즘 때문이다. Control Plane은 분산시스템에서 데이터의 일관성과 안정성을 유지하기 위해 etcd와 같은 분산 키-값 저장소를 사용한다. etcd는 Raft 합의 알고리즘을 통해 클러스터의 상태를 관리한다. RAFT 합의 알고리즘이란 요청을 수행할 때 Yes/No로 투표를 하게 되고, 과반수 이상의 노드가 있는 쪽의 요청을 유효한 요청으로 수행하게 되는 알고리즘이다.</p>
<ol>
<li>Quorum 개념</li>
</ol>
<ul>
<li>Quorum은 클러스터를 합의 결정을 내리기 위한 최소한의 노드 수이다.</li>
<li>최소 노드 수: (전체 노드 수 / 2) + 1</li>
</ul>
<ol start="2">
<li>홀 수 개일 때 장점 </li>
</ol>
<ul>
<li>Control Plane 노드가 홀수 개일 때, Quorum 크기를 계산하면 장애 허용 범위가 최대화된다.</li>
<li>예를 들어, 노드 수가 3개일 때, Quorum 크기는 2입니다. 따라서 하나의 노드가 장애를 일으켜도 나머지 두 노드가 정상적으로 작동하며 합의를 유지할 수 있다.</li>
<li>반면, 노드 수가 짝수(예: 4)라면 Quorum 크기는 여전히 3이므로, 추가된 노드가 장애 허용 범위에 기여하지 못한다.</li>
</ul>
<ol start="3">
<li>Quorum 을 만족하지 못하면</li>
</ol>
<ul>
<li>etcd는 읽기 전용 모드로 전환되고, 새로운 작업(스케줄링 등)은 처리할 수 없다.<h3 id="홀-수-개일-때-하나의-노드가-장애가-생겼을-때-상황">홀 수 개일 때 하나의 노드가 장애가 생겼을 때 상황</h3>
홀 수 개일 때 CP가 장애가 되면 CP는 짝수가 되지만 클러스터링 과정에서 우선순위도 함께 결정된다. 즉 짝수 개가 되었을 때 다시 합의 알고리즘을 수행하는 것이 아니라 클러스터링 이후 부터는 우선순위 가 정해진 순서 대로 primary를 정하게 된다. </li>
</ul>
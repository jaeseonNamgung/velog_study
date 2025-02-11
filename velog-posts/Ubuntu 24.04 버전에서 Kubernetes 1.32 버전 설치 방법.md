<h1 id="설치-환경">설치 환경</h1>
<ul>
<li>CPU: 2코어 이상</li>
<li>메모리: 2GB 이상</li>
<li>필요한 호스트: 총 3대<ul>
<li>마스터 노드: 1대</li>
<li>워커 노드: 2대</li>
</ul>
</li>
<li>운영 체제: Ubuntu 24.04 LTS (각 호스트에 설치)</li>
<li>가상화 환경:<ul>
<li>Mac OS (M2)</li>
<li>UTM 가상 머신 활용</li>
</ul>
</li>
</ul>
<h1 id="도커-설치">도커 설치</h1>
<pre><code class="language-bash">#  패키지 목록 업데이트 &amp; 필수 패키지 설치
sudo apt-get update
sudo apt-get install ca-certificates curl
# GPG 키 저장소 생성 및 Docker GPG 키 추가
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Docker 저장소를 APT 소스 목록에 추가
echo \
  &quot;deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release &amp;&amp; echo &quot;${UBUNTU_CODENAME:-$VERSION_CODENAME}&quot;) stable&quot; | \
  sudo tee /etc/apt/sources.list.d/docker.list &gt; /dev/null

sudo apt-get update</code></pre>
<pre><code class="language-bash"># 도커 설치
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin</code></pre>
<pre><code class="language-bash"># 도커 소켓 권한 설정
sudo chmod 666 /var/run/docker.sock
# 현재 사용자를 Docker 그룹에 추가
sudo usermod -aG docker $USER</code></pre>
<h1 id="쿠버네티스-설치">쿠버네티스 설치</h1>
<h2 id="쿠버네티스-런타임-구성">쿠버네티스 런타임 구성</h2>
<blockquote>
<p>Kubernetes v1.23.x와 v1.24.x 사이에는 큰 차이점이 존재한다. Kubernetes v1.24.0부터는 docker를 버렸다는 것이다. 공식 문서의 내용을 간단히 요약하면 다음과 같다.</p>
<ul>
<li>1.24 버전 이전에는 docker라는 specific한 CRI를 사용하고 있었다.</li>
<li>Kubernetes에서 docker 외에도 다양한 CRI를 지원하기 위해, CRI standard라는 것을 만들었다.</li>
<li>Docker는 CRI standard를 따르지 않고 있다.</li>
<li>Kubernetes는 docker 지원을 위해 dockershim이라는 코드를 만들어서 제공했다.</li>
<li>Kubernetes 개발 참여자들이 docker라는 특수 CRI를 위해 별도로 시간을 할애하는 것이 부담스럽다.</li>
<li>Kubernetes v1.24부터 dockershim 지원 안하기로 했다.</li>
</ul>
<p>쿠버네티스 공식 문서: <a href="https://kubernetes.io/ko/docs/setup/production-environment/container-runtimes/">https://kubernetes.io/ko/docs/setup/production-environment/container-runtimes/</a></p>
<p>참고 블로그: <a href="https://littlemobs.com/blog/kubernetes-package-repository-deprecation/">https://littlemobs.com/blog/kubernetes-package-repository-deprecation/</a></p>
</blockquote>
<ol>
<li>메모리 스왑 비활성화</li>
</ol>
<pre><code class="language-bash">sudo swapoff -a &amp;&amp; sudo sed -i '/swap/s/^/#/' /etc/fstab</code></pre>
<ol start="2">
<li>방화벽 비활성화 [방화벽이 켜져있는 경우]</li>
</ol>
<pre><code class="language-bash"># firewalld 설치 필요 
sudo apt-get install -y firewalld

sudo systemctl stop firewalld &amp;&amp; sudo systemctl disable firewalld
</code></pre>
<ol start="3">
<li>노드 간 통신을 위한 iptables 설정</li>
</ol>
<pre><code class="language-bash">cat &lt;&lt;EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# 필요한 sysctl 파라미터를 설정하면, 재부팅 후에도 값이 유지된다.
cat &lt;&lt;EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# 재부팅하지 않고 sysctl 파라미터 적용하기
sudo sysctl --system</code></pre>
<ol start="4">
<li>Containerd 설정</li>
</ol>
<pre><code class="language-bash">sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml</code></pre>
<ul>
<li>containerd config default: 기본 설정을 출력하는 명령어</li>
<li><code>| sudo tee /etc/containerd/config.toml</code>: 해당 출력을 <code>/etc/containerd/config.toml</code> 파일에 저장</li>
<li>tee 명령어를 사용하면 출력을 파일에 저장하면서 동시에 화면에도 표시할 수 있다.</li>
</ul>
<ol start="5">
<li>Cgroup 설정 (systemd 사용)</li>
</ol>
<blockquote>
<p>리눅스에서, <strong>control group</strong>은 프로세스에 할당된 리소스를 제한하는데 사용된다.
kubelet과 그에 연계된 컨테이너 런타임 모두 컨트롤 그룹(control group)들과 상호작용 해야 하는데, 이는 파드 및 컨테이너 자원 관리가 수정될 수 있도록 하고 cpu 혹은 메모리와 같은 자원의 요청(request)과 상한(limit)을 설정하기 위함이다. 컨트롤 그룹과 상호작용하기 위해서는, kubelet과 컨테이너 런타임이 cgroup 드라이버를 사용해야 한다. 매우 중요한 점은, kubelet과 컨테이너 런타임이 같은 cgroup group 드라이버를 사용해야 하며 구성도 동일해야 한다는 것이다.</p>
<p>두 가지의 cgroup 드라이버가 이용 가능하다.</p>
<ul>
<li><code>cgroupfs</code></li>
<li><code>systemd</code></li>
</ul>
<p>참고: <a href="https://kubernetes.io/ko/docs/setup/production-environment/container-runtimes/">https://kubernetes.io/ko/docs/setup/production-environment/container-runtimes/</a></p>
</blockquote>
<ul>
<li><code>/etc/containerd/config.toml</code> 파일에서 SystemdCgroup을 true로 변경</li>
<li>주의사항:</li>
</ul>
<pre><code class="language-bash">[plugins.&quot;io.containerd.grpc.v1.cri&quot;.containerd.runtimes.runc]
  ...
  [plugins.&quot;io.containerd.grpc.v1.cri&quot;.containerd.runtimes.runc.options]
    SystemdCgroup = true</code></pre>
<ol start="6">
<li>Containerd 재시작</li>
</ol>
<pre><code class="language-bash">sudo systemctl restart containerd
# containerd 상태 확인
sudo systemctl status containerd</code></pre>
<h2 id="kubeadm으로-클러스터-구성"><strong>kubeadm으로 클러스터 구성</strong></h2>
<pre><code class="language-bash">sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg</code></pre>
<ol>
<li>Google Public Key 다운로드</li>
</ol>
<pre><code class="language-bash">curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg</code></pre>
<ol start="2">
<li>Kubernetes 패키지 저장소 URL 추가</li>
</ol>
<pre><code>echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list</code></pre><pre><code class="language-bash">sudo apt-get update
sudo apt-get install -y kubelet=1.32.1-1.1 kubeadm=1.32.1-1.1  kubectl=1.32.1-1.1
sudo apt-mark hold kubelet kubeadm kubectl</code></pre>
<h2 id="kubernetes-클러스터-초기화-master-node-에서만-실행">Kubernetes 클러스터 초기화 (Master Node 에서만 실행!)</h2>
<ol>
<li>kubeadm 초기화</li>
</ol>
<pre><code class="language-bash">sudo kubeadm init --apiserver-advertise-address 192.168.64.20  --pod-network-cidr=192.168.0.0/16</code></pre>
<ul>
<li><code>—apiserver-advertise-address</code> : 마스터노드의 IP</li>
<li><code>--pod-network-cidr=</code> : 설치하고자 하는 CRI의 IP주소 대역<ul>
<li>Calico: 192.168.0.0/16</li>
</ul>
</li>
</ul>
<ol start="2">
<li>kubectl 권한 설정</li>
</ol>
<pre><code class="language-bash">
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
</code></pre>
<ol start="3">
<li>Pod CNI 애드온 설치 (Calico)</li>
</ol>
<pre><code class="language-bash">kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/tigera-operator.yaml</code></pre>
<pre><code class="language-bash">kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/custom-resources.yaml</code></pre>
<h1 id="자동-완성-설정">자동 완성 설정</h1>
<pre><code class="language-bash">source &lt;(kubectl completion bash) 
echo &quot;source &lt;(kubectl completion bash)&quot; &gt;&gt; ~/.bashrc </code></pre>
<pre><code class="language-bash">alias k=kubectl
complete -o default -F __start_kubectl k</code></pre>
<h2 id="워커-노드-구성">워커 노드 구성</h2>
<ul>
<li>Master Node를 초기화 후 출력된 <code>kubeadm join</code> 명령어를 사용하여 워커 노드를 구성</li>
</ul>
<pre><code class="language-bash">sudo kubeadm join 192.168.64.20:6443 --token 3s23ar.yhfowtvgyj88x5o7 \
    --discovery-token-ca-cert-hash sha256:111654d61fa28e3812a8a18b2e8f8f606b108ceeebcca45e6ff89f6d639cb813</code></pre>
<h2 id="각종-오류-상항">각종 오류 상항</h2>
<p>[Kubernetes 저장소 오류]</p>
<pre><code class="language-bash">E: The repository 'https://pkgs.k8s.io/core:/stable:/v1.24/deb  InRelease' is not signed.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.</code></pre>
<ul>
<li>GPG 키 다시 추가</li>
</ul>
<pre><code class="language-bash">sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key |
sudo tee /etc/apt/keyrings/kubernetes-apt-keyring.asc &gt;/dev/null</code></pre>
<ul>
<li>설치된 kubernetes.list 파일을 제거하고 다시 알맞은 Kubernetes 저장소 다시 설치</li>
</ul>
<pre><code class="language-bash">sudo rm -rf /etc/apt/sources.list.d/kubernetes.list</code></pre>
<p>[일반적인 오류와 해결 방법]</p>
<ul>
<li><strong>오류:</strong> <code>container runtime is not running</code></li>
</ul>
<pre><code>sudo rm /etc/containerd/config.toml
sudo systemctl restart containerd</code></pre><ul>
<li>권한 오류</li>
</ul>
<pre><code class="language-json">I0209 08:14:08.141011    3771 version.go:256] remote version is much newer: v1.32.1; falling back to: stable-1.30
[init] Using Kubernetes version: v1.30.9
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
    [ERROR IsPrivilegedUser]: **user is not running as root**
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher</code></pre>
<p>root 권한이 아니라서 생기는 오류 <code>sudo</code> 명령어를 사용하면 된다.</p>
<ul>
<li><code>kubeadm init</code> 오류</li>
</ul>
<pre><code class="language-json">I0209 08:14:35.166408    3952 version.go:256] remote version is much newer: v1.32.1; falling back to: stable-1.30
[init] Using Kubernetes version: v1.30.9
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
    [ERROR Port-6443]: Port 6443 is in use
    [ERROR Port-10259]: Port 10259 is in use
    [ERROR Port-10257]: Port 10257 is in use
    [ERROR FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml]: /etc/kubernetes/manifests/kube-apiserver.yaml **already exists**
    [ERROR FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml]: /etc/kubernetes/manifests/kube-controller-manager.yaml **already exists**
    [ERROR FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml]: /etc/kubernetes/manifests/kube-scheduler.yaml **already exists**
    [ERROR FileAvailable--etc-kubernetes-manifests-etcd.yaml]: /etc/kubernetes/manifests/etcd.yaml **already exists**
    [ERROR Port-10250]: Port 10250 is in use
    [ERROR DirAvailable--var-lib-etcd]: /var/lib/etcd is not empty
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher</code></pre>
<p>이 오류는 이미 <code>kubeadm init</code> 을 했기 때문에 yaml 파일이 존재해서 생기는 오류이다. 아래와 같이 기존에 존재한 yaml 파일을 제거해준 후 다시 초기화 해주면 된다.</p>
<pre><code class="language-bash"># kubeadm 초기화
sudo kubeadm reset -f</code></pre>
<h3 id="범외-minikube-시작-예시">범외 Minikube 시작 예시</h3>
<pre><code class="language-yaml">minikube start --driver='docker' --profile='thred-node' --cni='calico' --kubernetes-version='stable' --nodes=3</code></pre>
<h3 id="1-메모리-스왑-비활성화">1. 메모리 스왑 비활성화</h3>
<pre><code>sudo swapoff -a &amp;&amp; sudo sed -i '/swap/s/^/#/' /etc/fstab</code></pre><h3 id="2-노드-간-통신을-위한-iptables-설정">2. 노드 간 통신을 위한 iptables 설정</h3>
<pre><code>cat &lt;&lt;EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat &lt;&lt;EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sudo sysctl --system</code></pre><h3 id="3-방화벽-비활성화">3. 방화벽 비활성화</h3>
<pre><code>sudo ufw disable</code></pre><h3 id="4-패키지-업데이트-및-필수-패키지-설치">4. 패키지 업데이트 및 필수 패키지 설치</h3>
<pre><code>sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl</code></pre><h3 id="5-google-public-key-다운로드">5. Google Public Key 다운로드</h3>
<pre><code>sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg</code></pre><h3 id="6-kubernetes-패키지-저장소-url-추가">6. Kubernetes 패키지 저장소 URL 추가</h3>
<pre><code>echo &quot;deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /&quot; | sudo tee /etc/apt/sources.list.d/kubernetes.list</code></pre><h3 id="7-gpg-키-다운로드-및-추가">7. GPG 키 다운로드 및 추가</h3>
<pre><code>curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg</code></pre><h3 id="8-kubelet-kubeadm-kubectl-설치">8. kubelet, kubeadm, kubectl 설치</h3>
<pre><code>sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl</code></pre><h3 id="9-kubernetes-클러스터-초기화">9. Kubernetes 클러스터 초기화</h3>
<pre><code>sudo kubeadm init</code></pre><h3 id="-일반적인-오류와-해결-방법">* 일반적인 오류와 해결 방법:</h3>
<ul>
<li><strong>오류:</strong> <code>container runtime is not running</code></li>
</ul>
<pre><code>sudo rm /etc/containerd/config.toml
sudo systemctl restart containerd</code></pre><h3 id="10-워커-노드-연결">10. 워커 노드 연결</h3>
<p>초기화 후 출력된 <code>kubeadm join</code> 명령어를 사용하여 워커 노드를 연결:</p>
<pre><code>kubeadm join &lt;MASTER_IP&gt;:6443 --token &lt;TOKEN&gt; --discovery-token-ca-cert-hash sha256:&lt;HASH&gt;</code></pre><h3 id="11-모든-사용자에-대한-kubectl-설정">11. 모든 사용자에 대한 kubectl 설정</h3>
<pre><code>mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config</code></pre><h3 id="-권한-문제-해결-방법">* 권한 문제 해결 방법:</h3>
<pre><code>sudo chmod 644 /etc/kubernetes/admin.conf</code></pre><h3 id="12-pod-네트워크-애드온-설치-calico">12. Pod 네트워크 애드온 설치 (Calico)</h3>
<pre><code>curl -O https://raw.githubusercontent.com/gasida/KANS/main/kans3/calico-kans.yaml
kubectl apply -f calico-kans.yaml</code></pre><h3 id="13-레포지토리-오류-해결">13. 레포지토리 오류 해결</h3>
<h3 id="잘못된-레포지토리가-추가된-경우">잘못된 레포지토리가 추가된 경우:</h3>
<pre><code>sudo rm /etc/apt/sources.list.d/kubernetes.list</code></pre><h3 id="올바른-레포지토리를-다시-추가">올바른 레포지토리를 다시 추가:</h3>
<pre><code>echo &quot;deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /&quot; | sudo tee /etc/apt/sources.list.d/kubernetes.list</code></pre><h3 id="14-minikube-시작-예시">14. Minikube 시작 예시</h3>
<pre><code class="language-yaml">minikube start --driver='docker' --profile='three-node' --cni='calico' --kubernetes-version='stable' --nodes=3</code></pre>
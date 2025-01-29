<p>Git에는 민감한 정보를 올릴 수 없기 때문에, 각종 설정 파일은 Jenkins에 저장하고 있습니다. 빌드 시, Jenkins에 저장된 설정 파일을 resources 디렉토리에 복사(COPY)하여 사용하고 있습니다.</p>
<h1 id="문제-발생">문제 발생</h1>
<ul>
<li>Jenkins 빌드 과정 중 아래와 같은 오류가 발생했습니다.</li>
</ul>
<pre><code class="language-yaml">cp: cannot create regular file 'application-dev.yaml': Permission denied</code></pre>
<p><img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/7754a4ff-3fc2-4a0c-a12a-44169c0e38dc/image.png" /></p>
<ul>
<li>Jenkins 스크립트는 아래와 같이 설정 파일들을 특정 디렉토리에 복사하도록 작성되어 있습니다.</li>
</ul>
<pre><code class="language-yaml">dir('src/main/resources'){
                    withCredentials([
                            file(credentialsId: 'application', variable: 'APPLICATION'),
                            file(credentialsId: 'application-dev', variable: 'APPLICATION_DEV'),
                            file(credentialsId: 'serviceAccountKey', variable: 'SERVICE_ACCOUNT_KEY')
                    ]) {
                        sh '''
                            echo &quot;Copying application file&quot;
                            cp ${APPLICATION} application.yaml
                            cp ${APPLICATION_DEV} application-dev.yaml
                            cp ${SERVICE_ACCOUNT_KEY} serviceAccountKey.json
                        '''
                    }
                }</code></pre>
<h1 id="문제-원인">문제 원인</h1>
<p>Jenkins 컨테이너 내부에서 권한 문제로 인해 <code>application-dev.yaml</code> 파일을 생성할 수 없는 상태입니다.</p>
<p><code>docker exec -it -u root jenkins /bin/bash</code> 명령어로 Jenkins 컨테이너에 접속한 뒤, <code>ls -al</code> 명령어를 사용하여 문제 디렉토리(<code>/var/jenkins_home/workspace/&lt;jenkins build name&gt;/src/main/resources</code>)를 확인한 결과, 해당 디렉토리에 <strong>읽기 권한만</strong> 있는 것을 확인했습니다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/543d2186-331d-480c-b4ca-c1181a2fde0f/image.png" /></p>
<h1 id="해결-방법">해결 방법</h1>
<p>디렉토리에 <strong>쓰기 권한</strong>을 추가하여 문제를 해결할 수 있습니다. 아래 명령어를 실행하면 됩니다.</p>
<pre><code class="language-bash">sudo chown -R jenkins:jenkins /var/jenkins_home/workspace/thred_pipeline_dev/src/main/resources
sudo chmod -R u+w /var/jenkins_home/workspace/thred_pipeline_dev/src/main/resources</code></pre>
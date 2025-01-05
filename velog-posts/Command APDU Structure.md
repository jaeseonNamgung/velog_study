<h1 id="command-apdu-format">Command APDU Format</h1>
<p>Command Application Protocol Data Unit (APDU)는 4바이트의 Mandatory(필수) Header와 가변 길이의 Conditional(조건) Body로 구성된다.
<img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/24e66534-3cfb-4fd2-b7d4-0774e70e376f/image.png" /></p>
<ul>
<li>Command APDU에서 전송되는 데이터 바이트 수는 Lc(Command 데이터 필드의 길이)로 표현한다.</li>
<li>Response APDU에서 예상 최대 데이터 바이트 수는 Le(예상 데이터 길이)로 표현한다.<blockquote>
<p>만약 Le가 존재하고 그 값이 0이면, 최대 256바이트까지의 데이터가 반환될 것으로 예상할 수 있다. Command 메시지에서 Le가 필요한 경우, Le는 항상 '00'으로 설정해야 한다.</p>
</blockquote>
<h2 id="contents-of-command-apdu의-구성-요소">Contents of command APDU의 구성 요소</h2>
</li>
</ul>
<h3 id="command-apdu의-4가지-주요-유형">Command APDU의 4가지 주요 유형</h3>
<ol>
<li>Header Only (No Body)<ul>
<li>구성: CLA + INS + P1 + P2 (총 4 바이트)</li>
<li>설명: Body가 없는 간단한 명령으로, 데이터 전송이 필요 없을 때 사용.</li>
<li>예시: 상태 요청, 기기 초기화 등</li>
</ul>
</li>
<li>Header + Le (Response Expected)<ul>
<li>구성: CLA + INS + P1 + P2 + Le</li>
<li>설명: 명령을 전송하고 응답 데이터를 받을 때 사용. Body가 없지만, 응답 데이터의 최대 크기를 Le로 지정할 수 있다.</li>
<li>예시: 파일 읽기 명령, 상태 확인 등</li>
</ul>
</li>
<li>Header + Lc + Data (No Response Expected)<ul>
<li>구성: CLA + INS + P1 + P2 + Lc + Data</li>
<li>설명: 데이터를 포함한 명령을 전송하지만, 응답 데이터는 기대하지 않을 때 사용. Lc는 전송되는 데이터의 길이를 나타낸다.</li>
<li>예시: 파일 쓰기 명령, 설정 변경 등</li>
</ul>
</li>
<li>Header + Lc + Data + Le (Response Expected)<ul>
<li>구성: CLA + INS + P1 + P2 + Lc + Data + Le</li>
<li>설명: 데이터를 전송하고 응답 데이터를 기대하는 명령. Le는 응답 데이터의 최대 크기를 지정하며, Lc는 전송되는 데이터의 길이를 나타낸다.</li>
<li>예시: 암호화된 데이터를 전송하고 그에 대한 응답을 기대하는 경우<h2 id="cla와-ins란">CLA와 INS란</h2>
CLA는 ** 명령의 클래스**를 나타낸다. 즉 이 명령이 속한 프로토콜이나 규격을 정의하는 역할을 한다. </li>
</ul>
</li>
</ol>
<ul>
<li>예시:<ul>
<li>CLA = 0x00: ISO/IEC 7816-4 표준 명령</li>
<li>CLA = 0x80: 특정 응용 프로그램 또는 시스템에 맞춘 사용자 정의 명령<h3 id="class-바이트-코딩">Class 바이트 코딩</h3>
Class 바이트의 가장 상위 니블(4비트)은 명령의 유형을 나타낸다.
<img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/1d2feb47-9ee8-48e4-a2e3-28491d0c2667/image.png" /></li>
</ul>
</li>
<li>'0' : 산업 간 명령</li>
<li>'8' : 이 사양에 고유한 명령</li>
<li>그외 값: 이 사양의 범위 밖<h3 id="cla-바이트-코딩-규칙">CLA 바이트 코딩 규칙</h3>
</li>
</ul>
<table>
<thead>
<tr>
<th><strong>b8</strong></th>
<th><strong>b7</strong></th>
<th><strong>b6</strong></th>
<th><strong>b5</strong></th>
<th><strong>b4</strong></th>
<th><strong>b3</strong></th>
<th><strong>b2</strong></th>
<th><strong>b1</strong></th>
<th><strong>값</strong></th>
<th><strong>의미</strong></th>
</tr>
</thead>
<tbody><tr>
<td>0</td>
<td>0</td>
<td>0</td>
<td>0</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>'0X'</td>
<td>ISO/IEC 7816-4 첫 번째 산업 간 표준값.</td>
</tr>
<tr>
<td>1</td>
<td>0</td>
<td>1</td>
<td>0</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>'AX'</td>
<td>'0X'와 같으며, 특별히 명시된 경우 사용.</td>
</tr>
<tr>
<td>1</td>
<td>0</td>
<td>0</td>
<td>0</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>'8X'</td>
<td>'0X'와 같은 구조, 문서에 따라 정의됨.</td>
</tr>
<tr>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>X</td>
<td>X</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>보안 메시징 여부.</td>
</tr>
<tr>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>X</td>
<td>X</td>
<td>-</td>
<td>논리 채널 번호(0~3).</td>
</tr>
<tr>
<td>- CLA = 0x00, INS = 0x0A: logical channel 0을 이용한 SELECT FILE 명령</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>- CLA = 0x01, INS = 0x0A: logical channel 1을 이용한 SELECT FILE 명령</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>- CLA = 0x02, INS = 0x0A: logical channel 2를 이용한 SELECT FILE 명령</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>- CLA = 0x03, INS = 0x0A: logical channel 3을 이용한 SELECT FILE 명령</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>INS는 실행할 명령 또는 작업을 구체적으로 지정한 코드이다. INS는 CLA가 정의한 클래스 내에서 특정한 명령어를 구분하며, 이 명령에 따라 스마트 카드는 해당 작업을 수행하게 된다.</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>- 예시:</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>- INS = 0xA4: 선택(SELECT) 명령 (파일이나 응용 프로그램을 선택하는 명령)</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>- INS = 0xB2: 레코드 읽기(READ RECORD) 명령</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>- INS = 0x20: 인증(VERIFY) 명령 (PIN 확인 등)</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</tbody></table>
<h2 id="instruction-byte-ins-바이트의-코딩">Instruction Byte (INS 바이트)의 코딩</h2>
<p>INS 바이트의 명령어의 구체적인 동작을 정의하며, 각 명령에 대해 CLA(명령 클래스)와 관련하여 다음과 같이 정의된다.
<img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/74f0e14b-933a-4f7f-b299-71b37f34f770/image.png" /></p>
<ul>
<li>'8x' '1E': 애플리케이션 차단 (APPLICATION BLOCK)</li>
<li>'8x' '18': 애플리케이션 차단 해제 (APPLICATION UNBLOCK)</li>
<li>'8x' '16': 카드 차단 (CARD BLOCK)</li>
<li>'0x' '82': 외부 인증 (EXTERNAL AUTHENTICATE)</li>
<li>'8x' 'AE': 애플리케이션 암호문 생성 (GENERATE APPLICATION CRYPTOGRAM)</li>
<li>'0x' '84': 도전 값 요청 (GET CHALLENGE)</li>
<li>'8x' 'CA': 데이터 조회 (GET DATA)</li>
<li>'8x' 'A8': 처리 옵션 요청 (GET PROCESSING OPTIONS)</li>
<li>'0x' '88': 내부 인증 (INTERNAL AUTHENTICATE)</li>
<li>'8x' '24': PIN 변경/차단 해제 (PERSONAL IDENTIFICATION NUMBER (PIN) CHANGE/UNBLOCK)</li>
<li>'0x' 'B2': 레코드 읽기 (READ RECORD)</li>
<li>'0x' 'A4': 파일 선택 (SELECT)</li>
<li>'0x' '20': 인증 (VERIFY)</li>
</ul>
<h2 id="p1-p2-parameter-bytes">P1, P2 (Parameter Bytes)</h2>
<ul>
<li>어떤 값이든 가질 수 있으며, 사용되지 않을 경우 <strong>'00'</strong> 값으로 설정된다.<h2 id="데이터-필드-바이트-코딩">데이터 필드 바이트 코딩</h2>
</li>
<li>Command APDU의 데이터 필드는 데이터 요소의 문자열로 구성된다.</li>
<li>Response APDU의 데이터 필드는 템플릿으로 캡슐화된 데이터 객체나 데이터 객제의 문자열로 구성된다.<h1 id="response-apdu-format">Response APDU Format</h1>
Response APDU Format은 가변 길이의 Conditional Body와 2바이트의 Mandatory Trailer로 구성된다.
<img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/5ff845a7-7ddb-4e93-9d15-ba6b4affd9ac/image.png" /></li>
<li>Data: Conditional Body로 Command에 대한 Response 데이터가 포함된다.</li>
<li>SW1, SW2: Mandatory Trailer로 상태 코드를 나타내며, Command의 성공 여부나 오류 상태를 나타낸다.</li>
</ul>
<h2 id="sw1-sw2상태-바이트">SW1, SW2(상태 바이트)</h2>
<p>상태 바이트 SW1과 SW2는 모든 응답 메시지에서 <strong>전송 계층(Transport Layer)</strong>에 의해 <strong>응용 계층(Application Layer)</strong>으로 반환되며, 명령의 처리 상태를 나타낸다. 이 바이트들은 명령이 성공적으로 처리되었는지, 경고가 발생했는지, 또는 오류가 발생했는지를 코드화하여 응용 계층에 전달한다.
<img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/cc9c74a3-c0db-40e1-a658-c6e1c8ca261f/image.png" /></p>
<ul>
<li>SW1: 상태 코드의 첫 번째 바이트로, 상태의 주요 범주(정상 처리, 경고, 오류 등)를 나타낸다.</li>
<li>SW2: 상태 코드의 두 번째 바이트로, SW1이 나타내는 상태의 세부 정보를 제공한다.</li>
</ul>
<h3 id="주요-상태-코드">주요 상태 코드</h3>
<ul>
<li><p>SW1 : '6x' 또는 '90': 정상, 경고, 또는 오류 상태를 나타내며 ISO/IEC 7816-4에 따라 된다.</p>
</li>
<li><p>'61xx': SW2는 사용할 수 있는 추가 응답 바이트 수를 나타낸다.</p>
</li>
<li><p>'6Cxx': 잘못된 길이 Le, SW2는 정확한 길이를 나타낸다.
<img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/1841ed7f-a4b3-46e8-b0bf-369270fd853b/image.png" /></p>
</li>
<li><p>정상 처리 완료</p>
<ul>
<li>'90' '00': 프로세스 완료 (SW2의 다른 값은 예약됨)</li>
</ul>
</li>
<li><p>경고 처리</p>
<ul>
<li>'62' '83': 비휘발성 메모리 상태는 변경되지 않음; 선택된 파일이 무효화됨</li>
<li>'63' '00': 비휘발성 메모리 상태가 변경됨; 인증 실패</li>
<li>'63' 'Cx': 비휘발성 메모리 상태가 변경됨; 제공된 카운터 'x' (0-15)</li>
</ul>
</li>
<li><p>오류 검사</p>
<ul>
<li>'69' '83': 명령 허용되지 않음; 인증 방법 차단됨</li>
<li>'69' '84': 명령 허용되지 않음; 참조된 데이터 무효화됨</li>
<li>'69' '85': 명령 허용되지 않음; 사용 조건이 충족되지 않음</li>
<li>'6A' '81': 잘못된 P1, P2 매개변수; 지원되지 않는 기능</li>
<li>'6A' '82': 잘못된 P1, P2 매개변수; 파일을 찾을 수 없음</li>
<li>'6A' '83': 잘못된 P1, P2 매개변수; 레코드를 찾을 수 없음</li>
<li>'6A' '88': 참조된 데이터(데이터 객체)를 찾을 수 없음</li>
</ul>
</li>
</ul>
<h3 id="refrence">Refrence</h3>
<p><a href="https://jasmine125.tistory.com/581">https://jasmine125.tistory.com/581</a></p>
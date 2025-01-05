<h1 id="tlv-tag-length-value">TLV (Tag-Length-Value)</h1>
<p>TLV는 데이터 구조의 일종으로, 데이터의 태그(Tag), 길이(Length), 값(Value)을 조합하여 데이터를 표현하는 방식이다. </p>
<ul>
<li>Tag:<ul>
<li>데이터의 타입이나 속성을 식별하는 식별자</li>
<li>보통 1바이트 또는 그 이상의 길이를 가짐</li>
<li>예: 01, 0x1F, A1등</li>
</ul>
</li>
<li>Length:<ul>
<li>값의 길이를 나타낸다.</li>
<li>예: 04(4바이트), 0A(10바이트)</li>
</ul>
</li>
<li>Value: <ul>
<li>실제 데이터를 의미</li>
<li>예: 1234, ABCD, 0x9A8F</li>
</ul>
</li>
</ul>
<h2 id="tlv-예시">TLV 예시</h2>
<ul>
<li>Tag: 0x01 (1바이트)</li>
<li>Length: 0x03 (3바이트의 길이)</li>
<li>Value: ABC (ASCII 문자열)  <pre><code>01 03 41 42 43</code></pre></li>
<li>01 -&gt; TAG: 1번태그</li>
<li>03 -&gt; Length: 값의 길이가 3바이트</li>
<li>41, 42, 43: Vlaue: 'ABC' (ASCII 코드에서 'A' = 41, 'B' = 42, 'C' = 43)</li>
</ul>
<h1 id="asn1-abstract-syntax-notation-one">ASN.1 (Abstract Syntax Notation One)</h1>
<p>ASN.1은 데이터 구조를 정의하고 인코딩하기 위한 표준이다. 국제 표준화 기구(ISO)와 국제전기통신연합(ITU-T)에서 정의하였으며, 주로 네트워크 프로토콜, 인증서, 데이터 교환 및 통신 시스템에서 사용된다.</p>
<h2 id="asn1의-구조-및-구성-요소">ASN.1의 구조 및 구성 요소</h2>
<ul>
<li>모듈 정의:<ul>
<li>모듈은 ASN.1 구문을 정의하는 단위로, 각 모듈은 특정 목적의 데이터 구조 집합을 포함한다.</li>
<li>모듈 이름은 보통 대문자로 시작하며, 명확한 규칙에 따라 데이터를 그룹화한다.<h3 id="기본-데이터-타입">기본 데이터 타입</h3>
</li>
</ul>
</li>
<li>BOOLEAN: 참(TRUE) 또는 거짓(FALSE)의 논리 값</li>
<li>INTEGER: 정수형 데이터 타입</li>
<li>REAL: 실수형 데이터 타입</li>
<li>BIT STRING: 이진 데이터로 구성된 문자열</li>
<li>OCTET STRING: 바이트 단위의 데이터 문자열</li>
<li>NULL: 값이 없음을 나타냄</li>
<li>OBJECT IDENTIFIER: 특정 개체를 식별하기 위한 고유 식별자<pre><code>// 예시
age INTEGER ::= 25
isStudent BOOLEAN ::= TRUE</code></pre><h3 id="문자열-데이터-타입">문자열 데이터 타입</h3>
</li>
<li>UTF8String: 유니코드 UTF-8 형식의 문자열</li>
<li>IA5String: ASCII 문자만 허용되는 문자열</li>
<li>PrintableString: 숫자와 특정 특수 문자만 허용되는 문자열</li>
<li>VisibleString: 인쇄 가능한 ASCII 문자로 구성된 문자열</li>
<li>GeneralString: 다양한 문자 인코딩을 지원하는 범용 문자열<pre><code>// 예시
userName UTF8String ::= &quot;John Doe&quot;
address PrintableString ::= &quot;12345 Sample St.&quot;</code></pre><h3 id="구조적-데이터-타입">구조적 데이터 타입</h3>
</li>
<li>SEQUENCE: 특정 순서를 가진 여러 필드의 집합 (순차적 데이터 구조)<ul>
<li>예: SEQUENCE { name UTF8String, age INTEGER }</li>
</ul>
</li>
<li>SEQUENCE OF: 특정 타입의 연속된 데이터 목록을 의미.<ul>
<li>예: SEQUENCE OF INTEGER</li>
</ul>
</li>
<li>SET: 특정 순서를 고려하지 않는 여러 필드의 집합</li>
<li>SET OF: 특정 타입의 연속된 데이터 목록이지만, 순서를 고려하지 않음.</li>
<li>CHOICE: 정의된 여러 타입 중 하나를 선택하여 사용<ul>
<li>예: CHOICE { male BOOLEAN, female BOOLEAN }</li>
</ul>
</li>
<li>ENUMERATED: 특정 값 집합 중 하나를 선택<ul>
<li>예: ENUMERATED { red(0), green(1), blue(2) }<h3 id="특정-데이터-타입">특정 데이터 타입</h3>
</li>
</ul>
</li>
<li>ANY: 모든 데이터 타입을 가질 수 있는 범용 타입</li>
<li>DEFINED BY: 다른 필드의 값을 기준으로 데이터 타입을 선택<pre><code>// 예시
Age ::= INTEGER (0..120)
Name ::= UTF8String (SIZE(1..50))</code></pre><h3 id="데이터-타입의-제약-조건constraints">데이터 타입의 제약 조건(Constraints)</h3>
ASN.1에서는 제약 조건을 사용하여 데이터의 유효 범위를 지정할 수 있다.<pre><code>// 예시
Age ::= INTEGER (0..120)
Name ::= UTF8String (SIZE(1..50))</code></pre></li>
<li>Age는 0부터 120까지의 값만 가질 수 있는 INTEGER 타입</li>
<li>Name은 1~50자 길이의 UTF-8 문자열</li>
</ul>
<pre><code>// Person 이라는 데이터 구조는 name, age, gender라는 세 개의 필드로 구성된 SEQUENCE이다.
Person ::= SEQUENCE {
  name    UTF8String,
  age     INTEGER,
  gender  ENUMERATED { male (0), female (1) }
}
</code></pre><h2 id="asn1의-모듈-정의">ASN.1의 모듈 정의</h2>
<p>ASN.1은 모듈(Module) 단위로 데이터 구조를 정의하여, 모듈 내의 데이터 타입을 다른 모듈과 분리된 공간으로 관리할 수 있다.</p>
<pre><code>UserModule DEFINITIONS ::= BEGIN

  User ::= SEQUENCE {
    userID       INTEGER,
    userName     UTF8String,
    userStatus   ENUMERATED { active(0), inactive(1), pending(2) }
  }

  Address ::= SEQUENCE {
    streetName   UTF8String,
    houseNumber  INTEGER
  }

END</code></pre><p>위 예제는 UserModule이라는 모듈을 정의하고, 그 안에 User와 Address라는 두 가지 데이터 구조를 정의합니다.</p>
<ul>
<li>데이터 형식 식별자:<ul>
<li>각 데이터 타입은 태그(Tag)를 사용하여 식별된다. 태그는 Universal, Application, Context-Specific, Private 네 가지 클래스가 있으며, 값의 의미를 규정한다.</li>
</ul>
</li>
<li>제약 조건:<ul>
<li>특정 데이터 값에 대한 제약 조건을 정의할 수 있다.</li>
<li>예: age INTEGER (0..120) - 나이 값은 0에서 120 사이의 정수만 허용</li>
</ul>
</li>
<li>Class:<ul>
<li>Universal: 모든 어플리케이션에서 의미가 동일하며 X.208에 정의<ul>
<li>예: 0x02 - INTEGER, 0x04 - OCTET STRING, ...</li>
</ul>
</li>
<li>Application: 특정 어플리케이션에서 의미를 갖는 타입</li>
<li>Context-Specific: 주어진 구조와 타입에 특정한 의미를 갖는 타입</li>
<li>Private: 특정 기업에서 의미를 갖는 타입<h1 id="ber-tlv의-구성-요소">BER-TLV의 구성 요소</h1>
</li>
</ul>
</li>
<li>Tag 필드:<ul>
<li>Tag 필드는 데이터의 종류나 속성을 식별하기 위한 필드이다.</li>
<li>1바이트 이상으로 구성될 수 있으며, 여러 개의 바이트를 사용하여 복합적인 Tag를 표현할 수 있다.</li>
<li>첫 번째 바이트의 구조:<ul>
<li>비트 8-7: Class(클래스) - 00: Universal, 01: Application, 10: Context-Specific, 11: Private</li>
<li>비트 6: Type(유형) - 0: Primitive(단순), 1: Constructed(복합)</li>
<li>비트 5~1: Tag Number(태그 번호) - 기본 태그 값 또는 다중 바이트 태그의 시작값</li>
</ul>
</li>
</ul>
</li>
</ul>
<pre><code>| Bit 8  | Bit 7  | Bit 6  | Bit 5  | Bit 4  | Bit 3  | Bit 2  | Bit 1  |
| Class  | Class  | Type   | Tag Number (0–30)                        |</code></pre><p><img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/f569ac41-6cbb-4ba0-bd86-0d3a60f5a988/image.png" />
태그 번호가 0~30 범위에 있는 경우에는 단일 바이트로 인코딩된다. 그러나 태그 번호가 31 이상인 경우에는 다중 바이트 인코딩 방식으로 변환되어 추가 바이트가 필요하게 된다.
<img alt="" src="https://velog.velcdn.com/images/sunnamgung8/post/40f98ac4-6a06-4ad6-b355-98dccffdb994/image.png" /></p>
<p>만약 첫 번째 바이트가 11111로 subsequence bytes에 해당 된다면Another byte가 있는지 여부를 두 번째 바이트에서 확인해야된다.</p>
<h3 id="태그-번호의-인코딩-예시">태그 번호의 인코딩 예시</h3>
<ol>
<li>단일 바이트 태그(Single Octet):<ul>
<li>태그 번호가 0~30일 경우에는 위에서 설명한 5비트의 Tag Number 부분에 번호가 들어간다.</li>
<li>예시:<ul>
<li>Tag = 5, Class = Universal(00), Type = Primitive(0) 일 때:</li>
</ul>
</li>
</ul>
</li>
</ol>
<pre><code>| 0 | 0 | 0 | 0 | 0 | 1 | 0 | 1 |
</code></pre><ol start="2">
<li>다중 바이트 태그(Multi-Octet)</li>
</ol>
<ul>
<li><p>태그 번호가 31 이상일 경우, Tag Number 필드에 5비트 이상의 값을 담아야 하기 때문에 다중 바이트로 인코딩된다.</p>
</li>
<li><p>첫 번째 바이트에서 Tag Number 부분을 11111(2진수)로 설정하고, 이어지는 바이트에 실제 태그 번호를 표현한다.</p>
</li>
<li><p>예시:</p>
<ul>
<li>Tag = 37, Class = Universal, Type = Constucted 일 때:<pre><code>| 0 | 0 | 1 | 1 | 1 | 1 | 1 | 1 |     // 첫 바이트
| 0 | 0 | 1 | 0 | 0 | 1 | 0 | 1 |     // 두 번째 바이트: 태그 번호 37</code></pre></li>
</ul>
</li>
<li><p>Length 필드:</p>
<ul>
<li>Length 필드는 Value 필드의 길이를 정의하며, 1바이트 이상으로 표현될 수 있다.
길이가 작은 경우(0~127 바이트)에는 단일 바이트로 길이를 표현하고, 128 바이트 이상일 때는 다중 바이트로 길이를 나타낸다.</li>
<li>단일 바이트 길이: 0x00~0x7F(최대 127)</li>
<li>다중 바이트 길이: 0x81, 0x82, 0x83, ...(최대 길이 2^31-1 바이트)</li>
</ul>
</li>
</ul>
<h3 id="length-인코딩-방식">Length 인코딩 방식</h3>
<ul>
<li>Short Form (단일 바이트)<ul>
<li>Bit 8 = 0</li>
<li>Bits 7-1: 길이 값 (1~127바이트)</li>
</ul>
</li>
<li>Long Form (다중 바이트)<ul>
<li>Bit 8 = 1</li>
<li>Bits 7-1: 추가 Length Octet의 수 (추가 바이트들을 이용해 길이 표현)</li>
</ul>
</li>
<li>Indefinite Form (불명확한 길이)<ul>
<li>Bit 8 = 1</li>
<li>Bits 7-1 = 0000000</li>
<li>End-of-Content (0x00 0x00)으로 종료<ul>
<li>EOC = 00₁₆ 00₁₆ (16진수 <code>0x00 0x00</code>) //Content 종료를 의미</li>
</ul>
</li>
<li>Constructed 인코딩일 때 사용</li>
<li>값(Value) 필드가 여러 개의 TLV 구조로 구성되어 있고, 데이터 길이를 미리 알 수 없는 경우 적합함 </li>
</ul>
</li>
<li>Vlaue 필드<ul>
<li>Value 필드는 길이에 따라 실제 데이터를 저장하는 부분이다.
복합 Tag의 경우, Value 필드 내에 또 다른 TLV 구조가 포함될 수 있다.</li>
</ul>
</li>
</ul>
<blockquote>
<p>  Octet: Octet:   8 Bit로 구성된 데이터 단위로 1 Byte를 표현하는게 아니라 8 Bit를 표현함(1 Byte가 8 Bit가 아닌 컴퓨터도 있음)</p>
</blockquote>
<h1 id="primitive--constructed">Primitive &amp; Constructed</h1>
<ul>
<li>Primitive: Tag | Length | Value가 한 줄 그대로 출력된다.</li>
<li>Constructed: Tag Length는 그대로이지만 Value가 또 하나의 TLV 구조가 되는 형태이다.</li>
</ul>
<h3 id="refrence">Refrence</h3>
<p>  <a href="https://octob.medium.com/emv-qr-ber-tlv-asn-1-9625cf733022">https://octob.medium.com/emv-qr-ber-tlv-asn-1-9625cf733022</a>
  <a href="https://velog.io/@huk00j/BER-TLV">https://velog.io/@huk00j/BER-TLV</a></p>
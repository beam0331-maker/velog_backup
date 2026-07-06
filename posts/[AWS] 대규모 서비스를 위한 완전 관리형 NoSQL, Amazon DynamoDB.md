# [AWS] 대규모 서비스를 위한 완전 관리형 NoSQL, Amazon DynamoDB

작성일: Sun, 05 Jul 2026 11:54:00 GMT
링크: https://velog.io/@beam0331/AWS-%EB%8C%80%EA%B7%9C%EB%AA%A8-%EC%84%9C%EB%B9%84%EC%8A%A4%EB%A5%BC-%EC%9C%84%ED%95%9C-%EC%99%84%EC%A0%84-%EA%B4%80%EB%A6%AC%ED%98%95-NoSQL-Amazon-DynamoDB

---

<h2 id="1-amazon-dynamodb의-정의-및-특징">1. Amazon DynamoDB의 정의 및 특징</h2>
<h3 id="11-dynamodb란">1.1 DynamoDB란?</h3>
<p>Amazon DynamoDB는 대규모 트래픽 환경에서 뛰어난 확장성과 초고속 성능을 제공하는 <strong>완전 관리형(Managed), 분산형 NoSQL 데이터베이스 서비스</strong>이다.</p>
<h3 id="12-핵심-특징">1.2 핵심 특징</h3>
<ul>
<li><strong>완전 관리형 서비스 (Managed):</strong> 개발자가 하드웨어 프로비저닝, OS 패치, 데이터베이스 설정, 클러스터 확장 등의 인프라 관리 작업을 수행할 필요가 없다. AWS가 내부 인프라 운영을 100% 전담한다.</li>
<li><strong>일관된 초고속 성능:</strong> 데이터의 규모(건수, 용량)에 관계없이 요청에 대해 항상 <strong>수 밀리초(ms) 단위의 예측 가능한 초고속 지연 시간(Latency)</strong>을 유지한다.</li>
<li><strong>내장된 수평 확장성:</strong> 내부적으로 데이터를 여러 파티션에 자동으로 쪼개어 분산 저장(샤딩)하는 구조를 취한다. 덕분에 트래픽이 폭발할 때도 중단 없이 무한에 가까운 용량과 처리량을 감당할 수 있다.</li>
</ul>
<hr />
<h2 id="2-핵심-아키텍처-및-컴포넌트">2. 핵심 아키텍처 및 컴포넌트</h2>
<h3 id="21-partition-key-파티션-키와-내부-샤딩-규칙">2.1 Partition Key (파티션 키)와 내부 샤딩 규칙</h3>
<ul>
<li><strong>개념:</strong> 테이블 생성 시 지정하는 고유 키로, DynamoDB의 내부 메커니즘에서 <strong>'샤딩 키(Shard Key)'</strong> 역할을 한다.</li>
<li><strong>작동 원리:</strong> DynamoDB는 입력된 파티션 키 값을 자체 내부 해시 함수에 통과시켜 위치 좌표를 얻는다. 이후 <strong>일관된 해시(Consistent Hashing)</strong> 규칙에 따라 물리적으로 분리된 여러 파티션 중 어느 곳에 데이터를 저장하고 찾아올지 자동으로 결정한다.</li>
<li><strong>설계 표준 (핫스팟 방지):</strong> 특정 파티션 키에 데이터나 트래픽이 집중되는 <strong>핫스팟(Hotspot) 현상</strong>을 막아야 한다. <code>국적</code>이나 <code>상태값</code>처럼 쏠림이 심한 데이터 대신, <code>user_id</code>나 <code>order_id</code>와 같이 수학적으로 고르게 분산되는 고유 값을 파티션 키로 지정해야 설계 표준에 부합한다.</li>
</ul>
<h3 id="22-용량-모드-capacity-mode">2.2 용량 모드 (Capacity Mode)</h3>
<p>시스템의 트래픽 유형에 따라 비용과 처리 성능을 최적화할 수 있도록 두 가지 용량 모드를 지원한다.</p>
<ul>
<li><strong>프로비저닝 모드 (Provisioned Mode):</strong><ul>
<li>초당 필요한 읽기 용량 단위(RCU)와 쓰기 용량 단위(WCU)를 <strong>사전에 예약 설정</strong>하는 방식이다.</li>
<li>트래픽이 비교적 일정하고 예측 가능할 때 비용을 크게 절감할 수 있으며, 필요 시 자동으로 용량을 조절하는 오토 스케일링(Auto Scaling)과 연계하여 운영한다.</li>
</ul>
</li>
<li><strong>온디맨드 모드 (On-Demand Mode):</strong><ul>
<li>용량을 미리 예약하지 않고, <strong>실제 요청이 발생하여 읽고 쓴 만큼만 비용을 지불</strong>하는 방식이다.</li>
<li>트래픽 변동성이 극심하거나 예측이 불가능한 서비스에서 유연하게 대처할 수 있으며, AWS가 백그라운드에서 파티션을 즉각 늘려 대규모 트래픽을 감당한다.</li>
</ul>
</li>
</ul>
<h3 id="23-데이터-일관성-모델-consistency">2.3 데이터 일관성 모델 (Consistency)</h3>
<p>데이터 유실을 막기 위해 가용영역(AZ) 간에 데이터를 복제하는 DynamoDB의 특성상, 조회 시 두 가지 일관성 모델을 선택할 수 있다.</p>
<ul>
<li><strong>최종적 일관성 (Eventually Consistent) - 기본값:</strong><ul>
<li>데이터를 쓰고 난 직후 복제본에 데이터가 동기화되는 아주 찰나의 시간(수백 ms) 동안은 간헐적으로 수정 전 구버전 데이터가 읽힐 수 있다.</li>
<li>복제를 완벽하게 기다리지 않으므로 응답 속도가 가장 빠르며, 처리 비용이 강한 일관성에 비해 <strong>50% 저렴</strong>하다. 일반적인 SNS, 게시판 등 대부분의 서비스에 권장된다.</li>
</ul>
</li>
<li><strong>강한 일관성 (Strongly Consistent):</strong><ul>
<li>읽기 요청 시 복제본 중 가장 최신 데이터가 반영된 곳을 찾아 읽어옴으로써, <strong>방금 바뀐 데이터의 즉각적인 조회를 100% 보장</strong>한다.</li>
<li>비용이 2배로 들고 응답 시간이 소폭 늘어날 수 있지만, <strong>결제, 금융, 실시간 재고 관리</strong> 등 오차가 절대 허용되지 않는 핵심 업무(Tier-1) 시스템에 필수적으로 적용한다.</li>
</ul>
</li>
</ul>
<hr />
<h2 id="3-고성능-조회를-위한-확장-기능">3. 고성능 조회를 위한 확장 기능</h2>
<h3 id="31-gsi-global-secondary-index">3.1 GSI (Global Secondary Index)</h3>
<ul>
<li><strong>목적:</strong> 최초 설정한 파티션 키 외에 다른 필드로 초고속 조회를 수행해야 하는 <strong>조회 패턴 확장</strong> 요구사항에 대응한다.</li>
<li><strong>원리:</strong> 기존 테이블의 파티션 키가 <code>user_id</code>일 때 <code>email</code>로 검색하려면 테이블 전체를 뒤지는 풀 스캔(Full Scan)이 발생해 성능이 저하된다. 이때 <code>email</code>을 파티션 키로 하는 GSI를 생성하면, AWS가 백그라운드에서 실시간 복제되는 가상의 샤딩 테이블을 별도로 운용하여 초고속 조회를 가능하게 한다.</li>
</ul>
<h3 id="32-dax-dynamodb-accelerator">3.2 DAX (DynamoDB Accelerator)</h3>
<ul>
<li><strong>목적:</strong> 대규모 이벤트 등으로 초당 수백만 건의 읽기 트래픽이 몰려 데이터베이스 본체에 과부하가 걸리는 것을 방지한다.</li>
<li><strong>원리:</strong> DynamoDB 전면에 배치되는 <strong>완전 관리형 인메모리 캐시 하드웨어 엔진</strong>이다. 자주 찾는 데이터를 메모리에 적재하여 응답 속도를 밀리초(ms)에서 <strong>마이크로초(μs) 단위</strong>로 낮춘다.</li>
<li><strong>주의사항:</strong> 캐시 특성상 '최종적 일관성' 기반으로 작동하므로, 실시간 결제 데이터 등 강한 일관성이 필요한 아키텍처에 도입할 때는 데이터 신뢰도 문제를 신중하게 검토해야 한다.</li>
</ul>
<hr />
<h2 id="4-글로벌-재해-복구-아키텍처-global-tables">4. 글로벌 재해 복구 아키텍처 (Global Tables)</h2>
<ul>
<li><strong>개념:</strong> 전 세계의 여러 AWS 리전(Region)에 걸쳐 동일한 DynamoDB 테이블을 실시간 다중 복제하는 기술이다.</li>
<li><strong>작동 방식:</strong> <strong>비동기(Asynchronous) 복제</strong> 방식을 취하며, 서로 다른 리전 간에 수백 ms에서 수 초 내외의 미세한 동기화 지연이 발생할 수 있다.</li>
<li><strong>아키텍처 목적:</strong> 특정 AWS 리전 전체가 마비되는 대규모 재해 상황 시 타 리전으로 즉각 서비스를 전환하는 <strong>재해 복구(DR)</strong> 용도로 쓰인다. 동시에 해외 현지 사용자에게 가장 가까운 리전에서 데이터를 빠르게 제공하여 글로벌 지연 시간을 최소화하는 목적도 가진다.</li>
</ul>
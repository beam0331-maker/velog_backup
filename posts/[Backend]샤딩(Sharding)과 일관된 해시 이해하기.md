# [Backend]샤딩(Sharding)과 일관된 해시 이해하기

작성일: Sun, 05 Jul 2026 11:26:03 GMT
링크: https://velog.io/@beam0331/Backend%EC%83%A4%EB%94%A9Sharding%EA%B3%BC-%EC%9D%BC%EA%B4%80%EB%90%9C-%ED%95%B4%EC%8B%9C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0

---

<h2 id="1-샤드shard와-샤딩sharding의-개념">1. 샤드(Shard)와 샤딩(Sharding)의 개념</h2>
<h3 id="11-샤드shard란">1.1 샤드(Shard)란?</h3>
<ul>
<li><strong>정의</strong>: '조각, 파편'이라는 뜻으로, 전체 데이터셋 중 일부를 떼어내어 저장하고 있는 <strong>독립된 하나의 물리적 데이터베이스 서버</strong>를 의미한다.</li>
<li><strong>예시</strong>: 1억 명의 회원 데이터를 2,500만 명씩 나누어 담은 4대의 DB 서버가 있다면, 이 각각의 서버(1~4호기)가 곧 '샤드'이다.</li>
</ul>
<h3 id="12-샤딩sharding이란">1.2 샤딩(Sharding)이란?</h3>
<ul>
<li>하나의 거대한 데이터베이스를 여러 개의 샤드로 쪼개어 분산 저장하는 <strong>수평적 확장(Scale-Out) 기술</strong>이다.</li>
<li><strong>효과</strong>: 단일 DB의 성능 및 용량 한계를 극복하고, 특정 샤드 장애 시 다른 샤드로 피해가 가지 않도록 격리하는 내결함성(FT)을 확보할 수 있다</li>
</ul>
<hr />
<h2 id="2-샤딩-키-설계-3대-원칙">2. 샤딩 키 설계 3대 원칙</h2>
<p>데이터를 나눌 기준이 되는 값을 <strong>'샤딩 키(Shard Key)'</strong>라고 하며, 시스템이 자동으로 특정 샤드에 접근하도록 만들기 위해 아래 3대 원칙에 따라 정교하게 설계해야한다.</p>
<h3 id="①-균등-분산-핫스팟-방지">① 균등 분산 (핫스팟 방지)</h3>
<ul>
<li><strong>목적</strong>: 특정 샤드 서버 한 곳에 데이터와 트래픽이 몰려 과부하가 걸리는 <strong>핫스팟(Hotspot) 현상</strong>을 방지해야한다.</li>
<li><strong>방법</strong>: 회원 번호(<code>user_id</code>) 등의 고유 키값을 해시 함수에 통과시켜 수학적으로 모든 샤드에 골고루 분산되도록 설계한다.</li>
</ul>
<h3 id="②-조회-패턴과의-일치-cross-shard-join-최소화">② 조회 패턴과의 일치 (Cross-Shard Join 최소화)</h3>
<ul>
<li><strong>목적</strong>: 사용자가 서비스를 이용할 때 자주 함께 묶어서 조회하는 데이터는 가급적 같은 샤드에 모여 있어야 성능이 저하되지않는다.</li>
<li><strong>방법</strong>: '회원 정보'와 '그 회원의 주문 내역'처럼 연관된 데이터는 동일한 샤딩 키를 공유하여 <strong>같은 물리 서버 내에 저장</strong>되도록 설계한다. 여러 샤드를 넘나들며 데이터를 합치는 행위(Cross-Shard Join)를 최소화해야 한다.</li>
</ul>
<h3 id="③-확장-가능성-consistent-hashing">③ 확장 가능성 (Consistent Hashing)</h3>
<ul>
<li><strong>목적</strong>: 서비스 성장에 따라 샤드(서버)를 추가하거나 제거할 때, 기존 데이터가 대규모로 이사해야 하는 인프라 마비 사태를 방지해야한다</li>
<li><strong>방법</strong>: 단순 나머지($%$) 연산 방식 대신, <strong>일관된 해시(Consistent Hashing)</strong> 알고리즘을 도입하여 데이터 이동을 최소화한다.</li>
</ul>
<hr />
<h2 id="3-일관된-해시-consistent-hashing-메커니즘">3. 일관된 해시 (Consistent Hashing) 메커니즘</h2>
<p><img alt="" src="https://velog.velcdn.com/images/beam0331/post/c6d58add-2159-442e-b438-acb96c2a8ae0/image.jpg" /></p>
<h3 id="31-작동-원리-가상의-원형-링hash-ring">3.1 작동 원리: 가상의 원형 링(Hash Ring)</h3>
<ol>
<li><strong>서버와 데이터 배치</strong>: 가상의 둥근 링 위에 샤드 서버들과 저장할 데이터들을 동일한 해시 함수를 거쳐 특정 숫자 위치에 점을 찍듯 배치한다</li>
<li><strong>데이터 저장 규칙</strong>: 링 위에 배치된 데이터는 <strong>'자신의 위치에서 시계 방향'</strong>으로 이동하다가 가장 먼저 만나는 샤드 서버에 자동으로 저장된다.</li>
</ol>
<h3 id="32-서버-추가삭제-시의-장점">3.2 서버 추가/삭제 시의 장점</h3>
<ul>
<li><strong>기존 나머지($%$) 연산의 한계</strong>: 서버 수가 변하면 기준 분모가 바뀌므로 <strong>기존 데이터의 약 80%가 대이동</strong>해야 하는 이슈가 발생한다.</li>
<li><strong>일관된 해시의 혁신</strong>: 새 서버가 추가되면, 링 위에서 그 새 서버의 바로 뒤에 있던 기존 서버의 구역 중 <strong>일부만</strong> 새 서버가 넘겨받습니다. 결과적으로 전체 데이터가 움직이는 이슈가 발생하지 않고 <strong>$1/N$ 만큼의 데이터만 이동</strong>하므로 유연한 확장이 가능합니다.</li>
<li><strong>가상 노드(Virtual Nodes)</strong>: 서버들의 분신을 링 위 곳곳에 무작위로 복수 배치하여, 새 서버가 추가될 때 모든 기존 서버들의 부하를 조금씩 떼어오도록 정교하게 균등 분산을 처리한다.</li>
</ul>
<h3 id="33-데이터-조회-메커니즘">3.3 데이터 조회 메커니즘</h3>
<ul>
<li><strong>시계 방향 탐색</strong>: 사용자가 특정 키를 요청하면 시스템이 해당 키의 해시 위치를 계산한 뒤, <strong>저장할 때와 똑같이 시계 방향으로 가장 가까운 서버</strong>를 찾아가 데이터를 꺼내온다</li>
<li><strong>라우팅 테이블 공유</strong>: 실무에서는 모든 샤드 서버가 전체 링의 지도(주소록)를 공유하고 있어, 사용자가 어떤 샤드에 요청을 찌르더라도 주소록을 보고 정확한 목적지 샤드로 <strong>단 한 번만에(1-Hop)</strong> 요청을 토스해 줍니다.</li>
</ul>
https://yohaim.medium.com/6-3-%EC%98%A4%ED%94%88%ED%85%94%EB%A0%88%EB%A9%94%ED%8A%B8%EB%A6%AC-%EB%A9%94%ED%8A%B8%EB%A6%AD-3fe751d7c62f

## 5.3장 실습 정리 — HotROD + Jaeger 데모

### 실습 환경

- AWS EC2 t3.medium (Amazon Linux 2023)
- IP: 44.199.189.165
- 단일 서버에 모든 컴포넌트 실행

### 실행한 컴포넌트

| 컴포넌트 | 버전 | 포트 |
| --- | --- | --- |
| Jaeger all-in-one | 1.38.1 | 16686 (UI), 14250 (agent) |
| Prometheus | 2.39.1 | 9090 |
| Grafana | 9.2.4 | 3000 |
| Loki | 2.6.1 | 3100 |
| Promtail | 2.6.1 | 9080 |
| HotROD | jaeger 1.38.1 내장 | 8080 (UI), 8083 (metrics) |

---

### 아키텍처

```
사용자
  ↓
HotROD UI (8080)
  ↓ 차량 호출 클릭
frontend 서비스
  ├── customer 서비스 → mysql
  ├── driver 서비스 → redis
  └── route 서비스
  ↓ 트레이스 전송
Jaeger all-in-one (16686)

HotROD 앱
  ↓ 로그 → hotrod.log
Promtail → Loki (3100) → Grafana (3000)

HotROD 앱
  ↓ 메트릭 (8083)
Prometheus (9090) → Grafana (3000)
```

---

### 설정 파일 핵심 내용

### Loki (loki-config.yaml)

```yaml
# 책 원문은 schema v11, boltdb였으나 설치 버전(2.6.1) 기준으로 수정 필요
schema_config:
  configs:
    - from: 2018-04-15
      store: tsdb        # 책: boltdb → 실제: tsdb
      schema: v12        # 책: v11 → 실제: v12
      index:
        period: 24h      # 책: 168h → 실제: 24h
```

### Promtail (promtail-config.yaml)

```yaml
clients:
  - url: http://localhost:3100/loki/api/v1/push
scrape_configs:
  - job_name: system
    static_configs:
      - labels:
          job: hotrod
          __path__: /home/ec2-user/hotrod-demo/hotrod.log
```

### Prometheus (prometheus.yml)

```yaml
scrape_configs:
  - job_name: "hotrod-application"
    static_configs:
      - targets: ["localhost:8083"]
```

---

### 실습에서 확인한 것

### 1. Jaeger UI — 트레이스 목록

- HotROD UI에서 차량 호출 클릭 → 트레이스 자동 생성
- 요청마다 소요시간, 에러 여부 한눈에 확인
- `frontend: HTTP GET /dispatch` — 50 Spans, 6개 서비스 관여

<img width="1456" height="819" alt="image" src="https://github.com/user-attachments/assets/13c9755a-2ec5-4614-a727-53275fb2cc68" />


### 2. Jaeger UI — 트레이스 상세 (Span 트리)

차량 호출 1번의 내부 흐름:

```
frontend /dispatch (697ms)
  ├── customer /customer (336ms)  ← 병렬 실행
  │     └── mysql SQL SELECT (335ms) ← 병목
  └── driver FindNearest (163ms)  ← 병렬 실행
        ├── redis FindDriverIDs
        ├── redis GetDriver × N
        └── ❌ redis GetDriver (31ms) ← 에러, error=true 태그
```

**핵심:** 어느 서비스에서 병목이 발생했는지, 에러가 어디서 났는지 로그 없이 한눈에 파악 가능

<img width="1456" height="819" alt="image" src="https://github.com/user-attachments/assets/ef429a6d-bc55-46af-b9aa-a0c9b7e93903" />


### 3. Jaeger UI — System Architecture

```
              frontend
            ↙    ↓    ↘
        driver  route  customer
  (→redis 216)  (160)  (→mysql 16)
```

숫자 = 호출 횟수. route가 160번으로 가장 많음 (드라이버 후보 10명 경로를 각각 계산하는 구조)

<img width="1456" height="819" alt="image" src="https://github.com/user-attachments/assets/0bf366cd-57a3-4f87-adad-6a0d33cc224c" />


### 4. Grafana Explore — Loki 로그 확인

쿼리: `{job="hotrod"}`

- 633개 로그 수집 확인
- 로그 본문에 trace_id, span_id 포함되어 있음

```json
{
  "service": "driver",
  "trace_id": "0ebe19dfa14473fd",
  "span_id": "77ff999c75df6e420",
  "error": "redis timeout"
}
```

<img width="1456" height="819" alt="image" src="https://github.com/user-attachments/assets/45a4118f-753d-4924-8ce8-921029256a02" />


### 5. 로그 → 트레이스 연결

1. Loki 로그에서 `trace_id: 0ebe19dfa14473fd` 확인
2. Jaeger UI 우측 상단 **Lookup by Trace ID** 에 붙여넣기
3. 해당 트레이스 즉시 조회 성공

**이것이 로그와 트레이스의 상관관계(Correlation)** — trace_id가 두 신호를 연결하는 고리

<img width="1456" height="819" alt="image" src="https://github.com/user-attachments/assets/66470608-20bd-447c-a6f3-d6f3304cb8aa" />
<img width="1456" height="819" alt="image" src="https://github.com/user-attachments/assets/660cc9ad-9454-48a7-82f7-85cd141ea23b" />

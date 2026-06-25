https://yohaim.medium.com/5-1-%EC%83%81%EA%B4%80%EA%B4%80%EA%B3%84-2e435ce9c96f

### 이번 주에 해본 것

**그라파나 관측가능성 데모 환경 구성 및 실행 (책 Chapter 5.1)**

AWS EC2 인스턴스 2대를 직접 구성해서 아래 스택을 설치하고 실행했습니다.

| 서버 | 설치한 것 |
| --- | --- |
| server1 | Grafana, Prometheus, Tempo, Promtail, 샘플 앱 |
| server2 | Loki |

**직접 실행한 것들:**

- Grafana에 Prometheus, Loki, Tempo 데이터소스 연결
- `traces_spanmetrics_calls_total` 쿼리로 메트릭 확인
- Loki에서 애플리케이션 로그 수집 확인
- Tempo에서 Trace 목록 및 상세 화면 확인

- <img width="1920" height="1080" alt="week05_1" src="https://github.com/user-attachments/assets/76473dde-43d9-40a7-8612-ea556ecf1881" />

Prometheus - Exemplar

`traces_spanmetrics_calls_total`

→ Tempo metrics-generator가 자동 생성한 요청 수 메트릭 조회

<img width="1920" height="1080" alt="week05_2" src="https://github.com/user-attachments/assets/7cb66571-8fa1-4d3d-8b36-22bd537b6625" />

Loki - 로그 수집 확인

`{job="tracing-example"}`

→ tracing-example 앱에서 발생한 로그 전체 조회

<img width="1920" height="1080" alt="week05_3" src="https://github.com/user-attachments/assets/13e31d40-073c-4b4b-b210-4419d69b3f0e" />

Tempo - Trace 목록 확인

<img width="1920" height="1080" alt="week05_4" src="https://github.com/user-attachments/assets/0dbb40f8-8e0d-41f9-be10-4c1e499288d2" />

Tempo - Trace 상세 타임라인 확인

`{resource.service.name="tracing-example" && name="handle-request"}`

→ tracing-example 서비스의 handle-request span 목록 조회

---

### 새로 알게 된 점

- 메트릭, 로그, 트레이스가 Grafana에서 연결됨
    - 각각 따로따로 보는 게 아니라, 메트릭 그래프에서 이상한 시점을 발견하면 클릭 한 번으로 그 시점의 Trace로 이동할 수 있음

---

### 막힌 부분 / 같이 보고 싶은 질문

1. **Tempo config 버전 호환성 문제**
    - 책의 설정 파일(`search_enabled`, `metrics_generator_enabled`)이 최신 버전(2.4.1)에서 필드명이 바뀌어서 오류 발생
2. **tracing-example 바이너리**
    - 책에서 제공하는 샘플 앱 바이너리를 찾지 못해서 직접 Go로 샘플 앱을 만들어서 대체함
        - 1초마다 가상의 요청을 처리 -  0~500ms 사이 랜덤하게 기다렸다가 완료
        - 처리할 때마다 Tempo로 Trace 전송 - `handle-request`라는 span을 만들어서 Tempo에 보냄

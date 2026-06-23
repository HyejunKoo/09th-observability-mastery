# 인프라 모니터링 고도화

## Grafana + Prometheus 기반 Observability 플랫폼 구축 PoC
- 목적: 인프라 모니터링 고도화를 위한 오픈소스 기반 Observability 플랫폼 검토 및 PoC 수행

---

# 1. 개요

## 1.1 배경 및 목적

- Grafana + Prometheus 도입을 통한 VMware Aria Operations(vROPs) 및 VMware Aria Operations for Logs(vRLI) 보완
- 기존 한계점
  - 대시보드 커스터마이징 자유도 제한
  - 그룹사별 세분화된 뷰 구성 어려움
  - 알림(Alert) 유연성 부족
  - 운영자 관점 직관적 시각화 미흡
- 데이터 수집: VM 리소스, ESXi 호스트, 로그 이벤트, Horizon 세션

**각 도구의 포지션**
|구분|vROPs|vRLI|Grafana|Prometheus|
|--|--|--|--|--|
|역할|인프라 성능 모니터링|로그 수집/분석|시각화/대시보드|메트릭 수집/저장|
|특화|VMware 생태계|로그 분석|커스텀 대시보드|범용 메트릭|
|라이선스|유료(VMware)|유료(VMware)|무료|무료|
|커스터마이징|제한적|제한적|매우 자유로움|자유로움|

## 1.2 목표 아키텍처 

```

```

# 1-3. 환경 구성
|항목|내용|
|--|--|
|테스트 환경|Windows VDI + vROPs|
|Prometheus 버전|3.5.4|
|Grafanas 버전|OSS 최신버전|
|windows_exporter 버전|0.31.7|

# 2. 수행 단계별 결과

## 2-1. Prometheus 설치 및 실행

## 2-2. windows_exporter 설치

## 2-3. Grafana 설치 및 Prometheus 연동

## 2-4. vROPs REST API 연동

## 2-5. Grafana Infinity Plugin 연동 (진행중)
- Infinity Plugin 설치 완료
- vROPs API 연결 설정 진행 중
- SSL 인증서 이슈 해결 완료
- 토큰 인증 방식 최적화 진행 중


# 3. 향후 계획
1. Grafana ↔ vROPs API 연동 완성
   - ESXi 호스트 리소스 대시보드 구성
2. vRLI API 연동
   - 로그 이벤트 시각화 대시보드 구성
3. Horizon API 연동
   - 그룹사별 세션 현황 대시보드 구성

# 4. 참고

## 4-1. 이슈 및 해결 내역

|이슈|원인|해결 방법|
|--|--|--|
|Prometheus 실행 오류|data\queries.active 파일 점유|파일 삭제 후 재실행|
|vROPs API XML 응답|Accept 헤더 누락|Accept: application/json 헤더 추가|
|vROPs API TLS 오류|SSL 인증서 검증 실패|TLS 1.2 명시 + 검증 우회 설정|
|windows_exporter UNKNOWN|최초 수집 대기|15~30초 대기 후 UP 전환 확인|
|대시보드 N/A|대시보드 버전 불일치|메트릭명 기준 맞는 ID로 교체|

## 4-2. 보안 고려사항

|항목|조치|
|--|--|
|API 토큰|주기적 갱신 필요 (유효시간 존재)|
|네트워크|기존 허용된 통신 경로만 사용|

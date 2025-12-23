# intelligent-governance-platfrom
수백 개의 테이블을 관리해야 한다. 매번 수동으로 돌릴 수 없으니 스케줄러가 돌면서 메타데이터를 수집하고, ML 검사를 수행한 뒤, 문제가 있는 테이블은 사용자가 못 쓰게 막아야(Soft Delete/Deprecate) 한다.

## 구현 목표

1. Orchestration: `Apache Airflow` (혹은 Prefect)를 사용하여 DAG를 구성한다.
2. Task Flow: `Ingestion(수집)` -> `Profiling(통계추출)` -> `ML Detection(이상탐지)` -> `Governance Action(조치)` 순서로 파이프라인을 짠다.
3. Governance Action: ML 모델이 이상 징후를 발견하면, Python 클라이언트를 통해 Data Catalog API를 호출하여 해당 데이터셋에 'Deprecate(사용중지)' 배지를 단다.
4. Containerization: 이 모든 과정(Airflow, ML Script, Catalog)을 `docker-compose` 하나로 실행되게 구성한다.

```
intelligent-governance-platform/
├── dags/                          # Airflow DAGs (워크플로우 정의)
│   └── governance_pipeline.py     # 메인 파이프라인 스크립트
├── plugins/                       # Airflow에서 가져다 쓸 커스텀 모듈
│   ├── governance/                # 거버넌스 로직 패키지
│   │   ├── __init__.py
│   │   ├── collectors.py          # 메타데이터/통계 수집 모듈 (Profiling)
│   │   ├── detectors.py           # ML 이상 탐지 모델 로직 (Scikit-learn)
│   │   └── actions.py             # DataHub API 연동 모듈 (Tagging/Soft Delete)
│   └── utils/
│       └── db_connector.py        # DB 연결 헬퍼
├── data/                          # 학습용 데이터나 임시 저장소
│   ├── raw/
│   └── models/                    # 학습된 ML 모델 저장 (.pkl)
├── tests/                         # 테스트 코드 (필수!)
├── docker-compose.yaml            # Airflow & DataHub 실행 환경
├── requirements.txt               # 의존성 라이브러리
└── README.md                      # 프로젝트 설명서
```

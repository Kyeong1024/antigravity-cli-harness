# Antigravity CLI 하네스

한 줄의 도메인 설명을 **Antigravity CLI**를 위한 실무적 다중 에이전트 팀으로 변환하는 메타 스킬입니다.

## 기능

하네스에 *"X에 대한 하네스를 만들어줘"*라고 말하면 한 번에 다음을 스캐폴딩합니다:

- `.agents/plugins/{domain}-plugin/` 아래의 **플러그인**
- 각각 정의된 역할, 도구 세트 및 시스템 프롬프트를 가진 **서브에이전트** (`agents/{name}/agent.json`)
- 그 에이전트들이 사용하는 **스킬** (`skills/{name}/SKILL.md`)
- 명시적인 데이터 전달 및 오류 처리 규칙으로 팀을 조율하는 **오케스트레이터 스킬**
- 향후 세션에서 하네스를 자동으로 트리거할 수 있도록 **`AGENTS.md`**의 포인터 항목

그 결과는 일회성 스크립트가 아닌 재사용 가능하고 진화 가능한 팀 아키텍처입니다.

## 왜 하네스를 사용하나?

단일 에이전트 프롬프트는 작업이 여러 전문 분야를 넘으면 한계에 도달합니다 (예: 분석 → 빌드 → QA). 하네스는 다음과 같이 이를 해결합니다:

1. **전문성 분할** — 각 서브에이전트가 자체 컨텍스트 윈도우를 가지고 초점을 맞춘 역할을 함.
2. **협업 표준화** — 파일 기반 워크스페이스 (`_workspace/`)와 오케스트레이터를 통해 표준화된 협업.
3. **세션 간 생존** — 정의가 디스크에 살아있어 팀이 재현 가능하고 시간 경과에 따라 개선 가능.

## 생성된 구조

```
.agents/
└── plugins/
    └── {domain}-plugin/
        ├── plugin.json
        ├── agents/
        │   └── {agent-name}/
        │       └── agent.json          # 서브에이전트 정의
        └── skills/
            ├── {orchestrator}/
            │   └── SKILL.md            # 팀을 조율하는 워크플로우
            └── {skill-name}/
                ├── SKILL.md            # 단일 기능 작동 방식
                └── rules/              # 점진적으로 로드되는 참조
AGENTS.md                               # 트리거 포인터 + 변경 로그
```

## 아키텍처 패턴

하네스는 도메인에 따라 6가지 팀 패턴 중 하나를 선택합니다:

| 패턴 | 사용 시기 |
|---|---|
| **파이프라인** | 순차적, 종속적 단계 |
| **팬아웃 / 팬인** | 병렬로 수행되는 독립적인 작업 |
| **전문가 풀** | 전문가로의 조건부 라우팅 |
| **생산자–검토자** | 생성 후 루프에서 QA |
| **감독자** | 중앙 에이전트가 상태 및 디스패치 관리 |
| **계층적 위임** | 재귀적 서브 위임 |

## 실행 모드

| 모드 | 시기 |
|---|---|
| **서브에이전트** *(기본)* | ≥2 전문 분야가 협업; 각각 `invoke_subagent`를 통해 격리된 컨텍스트에서 실행 |
| **병렬 서브에이전트** | 동시에 실행할 독립적인 작업 |
| **직접 실행** | 에이전트 분리가 오버헤드인 단순한 일회성 작업 |

## 워크플로우

메타 스킬은 7개 단계를 거칩니다:

0. **감사** — 기존 플러그인 감지, 새 빌드 vs. 확장 vs. 유지보수 결정.
1. **도메인 분석** — 작업 유형, 코드베이스, 사용자 기술 수준 식별.
2. **팀 아키텍처** — 실행 모드 + 패턴 선택, 작업을 전문 분야로 분할.
3. **서브에이전트 정의** — 각 `agent.json`을 역할, 도구, I/O 프로토콜로 작성.
4. **스킬 생성** — 강압적이고 트리거 친화적인 설명과 `rules/`로의 점진적 공개로 `SKILL.md` 파일 작성.
5. **오케스트레이션** — 파일 기반 데이터 전달, 오류 처리, 후속 지원으로 팀 조율.
6. **검증** — 구조 검사, 트리거 검사 (should-trigger + near-miss), 드라이 런.
7. **진화** — 각 실행 후 피드백 수집; 에이전트/스킬 업데이트 및 `AGENTS.md`의 변경 로그.

## 설치

`.agents/plugins/harness-plugin/` 디렉토리를 모든 Antigravity CLI 프로젝트에 떨어뜨립니다. `harness` 스킬은 다음과 같은 요청에 자동으로 트리거됩니다:

- "build a harness for {domain}"
- "set up a harness", "design a harness"
- "audit / sync the harness", "harness status"

## 사용법

Antigravity CLI 세션에서:

```
> Build a harness for a content marketing pipeline.
```

스킬은 다음과 같이 작동합니다:

1. 기존의 `.agents/plugins/`와 `AGENTS.md`를 감사합니다.
2. 팀을 제안하고 (예: 연구자 → 작성자 → 편집자 → QA) 확인합니다.
3. 플러그인을 생성하고, `agent.json`/`SKILL.md` 파일을 작성하고, `AGENTS.md`에 트리거를 등록합니다.
4. 검증을 실행하고 보고서를 제출합니다.

후속 턴 (`"analyst 단계를 다시 해줘"`, `"security reviewer를 추가해줘"`, `"harness audit"`)은 확장 / 유지보수 모드의 동일한 스킬로 처리됩니다.

## 참고자료

스킬은 `.agents/plugins/harness-plugin/skills/harness/rules/` 아래에 내부 가이드와 함께 제공됩니다:

- `agent-design-patterns.md` — 패턴 카탈로그, 분리 기준
- `team-examples.md` — 전체 팀 정의 예제
- `orchestrator-template.md` — 오류 처리가 있는 오케스트레이터 스켈레톤
- `skill-writing-guide.md` — `SKILL.md` 작성 패턴
- `skill-testing-guide.md` — 트리거 및 실행 테스트 방법론
- `qa-agent-guide.md` — QA 서브에이전트 설계

## 감사의 말

[revfactory/harness](https://github.com/revfactory/harness)에서 포팅되었으며, Antigravity CLI의 플러그인 및 서브에이전트 모델에 맞게 재작업되었습니다.

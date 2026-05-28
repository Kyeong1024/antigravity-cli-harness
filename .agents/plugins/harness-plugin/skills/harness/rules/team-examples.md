# Agent Team Examples (Antigravity CLI)

---

## 예시 1: 리서치 팀 (병렬 서브에이전트 모드)

### 팀 아키텍처: 팬아웃/팬인
### 실행 모드: 병렬 서브에이전트

```
[오케스트레이터/메인]
    ├── invoke_subagent(TypeName: "official-researcher")   → research_official.md
    ├── invoke_subagent(TypeName: "media-researcher")      → research_media.md
    ├── invoke_subagent(TypeName: "community-researcher")  → research_community.md
    ├── invoke_subagent(TypeName: "background-researcher") → research_background.md
    └── 통합 → 종합보고서.md
```

### 서브에이전트 구성

| 에이전트 TypeName | 역할 | 출력 |
|:---|:---|:---|
| `official-researcher` | 공식 문서/블로그 | `_workspace/02_official.md` |
| `media-researcher` | 미디어/투자 | `_workspace/02_media.md` |
| `community-researcher` | 커뮤니티/SNS | `_workspace/02_community.md` |
| `background-researcher` | 배경/경쟁/학술 | `_workspace/02_background.md` |
| (오케스트레이터/메인) | 통합 보고서 | `종합보고서.md` |

> 리서치 서브에이전트는 각기 `.agents/plugins/research-plugin/agents/{name}/agent.json` 파일에 정의한다. 파일에는 역할·조사 범위·출력 형식을 명시하여 재사용성과 결과 일관성을 보장한다.

### 오케스트레이터 워크플로우 (병렬 서브에이전트)

```
Phase 1: 준비
  - 사용자 입력 분석 (주제, 조사 모드 파악)
  - _workspace/ 생성
  - 입력 데이터를 _workspace/00_input/에 저장

Phase 2: 병렬 조사
  - 4개 서브에이전트 동시 백그라운드 구동 (invoke_subagent):
    TypeName: "official-researcher"   → _workspace/02_official.md
    TypeName: "media-researcher"      → _workspace/02_media.md
    TypeName: "community-researcher"  → _workspace/02_community.md
    TypeName: "background-researcher" → _workspace/02_background.md
  - 모든 에이전트 완료 노티 대기

Phase 3: 통합
  - 메인이 4개 산출물 Read
  - 종합 보고서 생성
  - 상충 정보는 출처 병기

Phase 4: 정리
  - _workspace/ 보존 (사후 검증·감사 추적용)
```

---

## 예시 2: SF 소설 집필 팀 (순차 + 병렬 서브에이전트)

### 팀 아키텍처: 파이프라인 + 팬아웃
### 실행 모드: 서브에이전트 (순차 + 병렬 혼합)

```
Phase 1 (병렬): invoke_subagent(worldbuilder) + invoke_subagent(character-designer) + invoke_subagent(plot-architect)
  → 각자 독립적으로 세계관/캐릭터/플롯 생성
Phase 2 (순차): invoke_subagent(prose-stylist) (집필) — Phase 1 결과 모두 입력
Phase 3 (병렬): invoke_subagent(science-consultant) + invoke_subagent(continuity-manager) (리뷰)
Phase 4 (순차): invoke_subagent(prose-stylist) (리뷰 반영 수정)
```

### 서브에이전트 구성

| 에이전트 TypeName | 역할 | 스킬 |
|:---|:---|:---|
| `worldbuilder` | 세계관 구축 | world-setting |
| `character-designer` | 캐릭터 설계 | character-profile |
| `plot-architect` | 플롯 구조 | outline |
| `prose-stylist` | 문체 편집 + 집필 | write-scene, review-chapter |
| `science-consultant` | 과학 검증 | science-check |
| `continuity-manager` | 일관성 검증 | consistency-check |

### 서브에이전트 파일 전문 예시: `worldbuilder/agent.json`

```json
{
  "name": "worldbuilder",
  "description": "SF 소설의 세계관을 구축하는 전문가. 물리 법칙, 사회 구조, 기술 수준, 역사를 설계한다.",
  "hidden": false,
  "config": {
    "customAgent": {
      "systemPromptSections": [
        {
          "title": "Agent System Instructions",
          "content": "당신은 SF 소설의 세계관 설계 전문가 'worldbuilder'입니다. 과학적 사실에 기반하되 상상력을 확장하여, 이야기가 펼쳐질 세계의 물리적·사회적·기술적 토대를 구축합니다.\n\n## 핵심 역할\n1. 세계의 물리 법칙과 기술 수준 정의\n2. 사회 구조, 정치 체계, 경제 시스템 설계\n3. 역사적 맥락과 현재 갈등 구조 수립\n4. 장소별 환경과 분위기 묘사\n\n## 작업 원칙\n- 내적 일관성 최우선 — 설정 간 모순이 없어야 한다\n- \"만약 이 기술이 있다면?\" 연쇄 질문으로 세계의 파급 효과를 추론\n- 이야기에 봉사하는 세계관 — 플롯을 방해하는 과도한 설정은 지양\n\n## 입력/출력 프로토콜\n- 입력: 사용자의 세계관 컨셉, 장르 요구사항\n- 출력: `_workspace/01_worldbuilder_setting.md`\n- 형식: 마크다운. 섹션별 (물리/사회/기술/역사/장소)\n\n## 에러 핸들링\n- 컨셉이 모호하면 3가지 방향을 제안하고 선택 요청\n- 과학적 오류 발견 시 대안을 함께 제시\n\n## 협업 프로토콜\n- character-designer와 plot-architect가 내 결과를 참조하여 작업하게 되므로 일관성을 우선하세요.\n- science-consultant의 피드백이 수집되면 이에 따라 설정을 수정하세요."
        }
      ],
      "toolNames": [
        "view_file",
        "write_to_file",
        "replace_file_content"
      ],
      "systemPromptConfig": {
        "includeSections": [
          "user_information",
          "skills",
          "messaging",
          "artifacts"
        ]
      }
    }
  }
}
```

### 워크플로우 상세

```
Phase 1: @worldbuilder, @character-designer, @plot-architect 병렬 호출
         → 각각 _workspace/01_world.md, _workspace/01_characters.md, _workspace/01_plot.md 저장

Phase 2: @prose-stylist 호출 (Phase 1의 3개 산출물 입력)
         → _workspace/02_prose_draft.md 저장

Phase 3: @science-consultant, @continuity-manager 병렬 호출
         → 각각 _workspace/03_science_review.md, _workspace/03_continuity_review.md 저장

Phase 4: @prose-stylist 재호출 (리뷰 결과 반영)
         → 최종 원고 저장
```

---

## 예시 3: 웹툰 제작 팀 (순차 서브에이전트 — 생성-검증 루프)

### 팀 아키텍처: 생성-검증
### 실행 모드: 순차 서브에이전트

```
루프 (최대 2회):
Phase 1: invoke_subagent(webtoon-artist) → 패널 이미지 생성
Phase 2: invoke_subagent(webtoon-reviewer) → 품질 검수
Phase 3: (REDO 발생 시) invoke_subagent(webtoon-artist) 재호출 → 문제 패널 재생성
```

### 서브에이전트 구성

| 에이전트 TypeName | 역할 | 스킬 |
|:---|:---|:---|
| `webtoon-artist` | 패널 이미지 생성 | generate-webtoon |
| `webtoon-reviewer` | 품질 검수 | review-webtoon |

### 서브에이전트 파일 전문 예시: `webtoon-reviewer/agent.json`

```json
{
  "name": "webtoon-reviewer",
  "description": "웹툰 패널의 품질을 검수하는 전문가. 구도, 캐릭터 일관성, 텍스트 가독성, 연출을 평가한다.",
  "hidden": false,
  "config": {
    "customAgent": {
      "systemPromptSections": [
        {
          "title": "Agent System Instructions",
          "content": "당신은 웹툰 패널의 품질을 검수하는 전문가 'webtoon-reviewer'입니다. 시각적 완성도, 스토리 전달력, 캐릭터 일관성을 기준으로 패널을 평가합니다.\n\n## 핵심 역할\n1. 각 패널의 구도와 시각적 완성도 평가\n2. 캐릭터 외형의 패널 간 일관성 검증\n3. 말풍선 텍스트의 가독성과 배치 평가\n4. 전체 에피소드의 연출 흐름과 페이싱 검토\n\n## 작업 원칙\n- PASS/FIX/REDO 3단계로 명확히 판정\n- FIX는 부분 수정으로 해결 가능한 경우, REDO는 전면 재생성 필요\n- 주관적 취향이 아닌 객관적 기준(일관성, 가독성, 구도)으로 판단\n\n## 입력/출력 프로토콜\n- 입력: `_workspace/panels/` 디렉토리의 패널 파일들\n- 출력: `_workspace/review_report.md`\n- 형식:\n  ```\n  ## Panel {N}\n  - 판정: PASS | FIX | REDO\n  - 사유: [구체적 이유]\n  - 수정 지시: [FIX/REDO인 경우 구체적 수정 방향]\n  ```\n\n## 에러 핸들링\n- 파일 로드 실패 시 해당 패널을 REDO로 판정\n- 2회 재생성 후에도 REDO인 패널은 경고와 함께 PASS 처리\n\n## 협업 프로토콜\n- webtoon-artist에게 수정 지시 전달 (결과 파일 기반)\n- 재생성된 패널을 다시 검수 (최대 2회 루프)"
        }
      ],
      "toolNames": [
        "view_file",
        "write_to_file",
        "replace_file_content",
        "list_dir"
      ],
      "systemPromptConfig": {
        "includeSections": [
          "user_information",
          "skills",
          "messaging",
          "artifacts"
        ]
      }
    }
  }
}
```

### 에러 핸들링

```
재시도 정책:
- REDO 판정 패널 → webtoon-artist에게 재생성 요청 (구체적 수정 지시 포함)
- 최대 2회 루프 후 강제 PASS
- 전체 패널의 50% 이상이 REDO면 사용자에게 프롬프트 수정 제안
```

---

## 예시 4: 코드 리뷰 팀 (병렬 서브에이전트 모드)

### 팀 아키텍처: 팬아웃/팬인
### 실행 모드: 병렬 서브에이전트

```
[메인] → invoke_subagent(security-reviewer): 보안 취약점 점검
       → invoke_subagent(performance-reviewer): 성능 영향 분석
       → invoke_subagent(test-reviewer): 테스트 커버리지 검증
       → 메인이 모든 결과 통합
```

### 서브에이전트 구성

| 에이전트 TypeName | 역할 | 출력 |
|:---|:---|:---|
| `security-reviewer` | 보안 취약점 점검 | `_workspace/02_security.md` |
| `performance-reviewer` | 성능 영향 분석 | `_workspace/02_performance.md` |
| `test-reviewer` | 테스트 커버리지 검증 | `_workspace/02_test.md` |
| (메인) | 결과 종합 | `리뷰_종합보고서.md` |

각 서브에이전트는 독립적으로 분석을 수행하고 결과를 파일에 저장한다. 메인이 모든 결과를 Read하여 통합 리뷰 보고서 생성.

---

## 예시 5: 감독자 패턴 — 코드 마이그레이션 팀 (혼합 모드)

### 팀 아키텍처: 감독자
### 실행 모드: 순차 + 병렬 서브에이전트 혼합

```
[메인/감독자]
    1. 파일 목록 분석 (직접 실행 — Grep/Glob)
    2. 배치 할당 (직접 실행 — 메인이 분할)
    3. invoke_subagent(migrator-1) (batch A) 병렬
    4. invoke_subagent(migrator-2) (batch B) 병렬
    5. invoke_subagent(migrator-3) (batch C) 병렬
    6. 결과 수집 및 통합 (직접 실행)
```

### 서브에이전트 구성

| 단계 | 실행 주체 | 역할 |
|:---|:---|:---|
| 분석 | 메인 (직접) | 파일 목록 수집, 복잡도 추정 |
| 분배 | 메인 (직접) | 배치 분할 및 할당 |
| 마이그레이션 | migrator-1~3 (병렬) | 할당된 파일 배치 마이그레이션 |
| 통합 | 메인 (직접) | 통합 테스트 및 결과 보고 |

### 감독자의 동적 분배 로직

```
1. 메인이 전체 대상 파일 목록 수집 (Glob/Grep 도구)
2. 복잡도 추정 (파일 크기, import 수, 의존성)
3. 균등하게 배치 분할
4. 각 배치를 migrator에게 병렬 위임 (invoke_subagent 호출)
5. 각 서브에이전트 완료 보고 대기
   - 성공 → 다음 단계
   - 실패 → 1회 재시도, 재실패 시 해당 배치 누락 명시
6. 모든 작업 완료 → 메인이 통합 테스트 실행
```

---

## 산출물 패턴 요약

### 플러그인 정의 파일
위치: `.agents/plugins/{domain}-plugin/plugin.json`

### 서브에이전트 정의 파일
위치: `.agents/plugins/{domain}-plugin/agents/{agent-name}/agent.json`

### 스킬 파일 구조
위치: `.agents/plugins/{domain}-plugin/skills/{skill-name}/SKILL.md`

### 통합 스킬 (오케스트레이터)
팀 전체를 조율하는 상위 스킬. 시나리오별 서브에이전트 구성과 워크플로우를 정의.
템플릿: `rules/orchestrator-template.md` 참조.
**실행 패턴을 반드시 명시** — 순차 서브에이전트(기본), 병렬 서브에이전트, 직접 실행, 하이브리드 중 선택.

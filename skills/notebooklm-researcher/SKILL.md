# NotebookLM Research-to-Slides Workflow

`nlm` CLI를 활용한 자동화된 리서치 및 프레젠테이션 생성 스킬입니다.

## Purpose

리서치 주제를 입력받아 Google NotebookLM의 딥 리서치 → 소스 임포트 → 슬라이드 생성까지 전체 파이프라인을 자동 오케스트레이션합니다.

## Prerequisites

- `nlm` CLI 설치 및 PATH에 등록
- 유효한 NotebookLM 인증 프로필 설정 완료

## Workflow Pipeline

```
┌─────────────────┐    ┌──────────────────┐    ┌────────────────────┐    ┌────────────────┐
│  Auth Checker   │───▶│ Notebook Manager │───▶│ Research Conductor │───▶│ Slides Creator │
│                 │    │                  │    │                    │    │                │
│ • login --check │    │ • notebook list  │    │ • research start   │    │ • slides create│
│ • auth status   │    │ • notebook create│    │ • research status  │    │ • studio status│
│                 │    │                  │    │ • research import  │    │ • download     │
└─────────────────┘    └──────────────────┘    └────────────────────┘    └────────────────┘
```

## Trigger

- "리서치", "research", "notebooklm", "nlm research", "발표 자료", "슬라이드"

## Agent Delegation (7-Section Structure)

### Phase 1: Authentication (auth-checker)

```
1. TASK: nlm login --check --profile work 실행하여 인증 상태 확인
2. EXPECTED OUTCOME: 인증 유효 확인. 실패시 사용자에게 nlm login --profile work 실행 요청
3. REQUIRED SKILLS: auth-checker
4. REQUIRED TOOLS: Bash, AskUserQuestion
5. MUST DO:
   - 반드시 nlm login --check --profile work 먼저 실행
   - 실패시 사용자에게 nlm login --profile work 직접 실행 요청
   - 사용자가 완료 확인 후 재검증
6. MUST NOT DO:
   - nlm login을 --check 없이 직접 실행 금지 (브라우저가 열림)
   - 인증 실패 상태에서 다음 단계 진행 금지
7. CONTEXT: 세션은 약 20분 지속. 실패시 사용자가 직접 nlm login --profile work 실행해야 함
```

### Phase 2: Notebook Setup (notebook-manager)

```
1. TASK: 리서치용 노트북 생성 또는 기존 노트북 선택
2. EXPECTED OUTCOME: notebook-id 반환
3. REQUIRED SKILLS: notebook-manager
4. REQUIRED TOOLS: Bash, AskUserQuestion
5. MUST DO:
   - nlm notebook list --profile {profile} 로 기존 목록 조회
   - nlm notebook create "{title}" --profile {profile} 로 생성 (positional arg)
   - nlm notebook get {id} --profile {profile} 로 확인
6. MUST NOT DO:
   - nlm notebook create --name 형식 사용 금지 (positional arg)
   - nlm notebook info 사용 금지 (nlm notebook get 사용)
7. CONTEXT: title은 사용자 입력 또는 리서치 주제 기반 자동 생성
```

### Phase 3: Research Execution (research-conductor)

```
1. TASK: 딥 리서치 실행 → 완료 대기 → 소스 임포트
2. EXPECTED OUTCOME: 모든 소스가 노트북에 임포트됨
3. REQUIRED SKILLS: research-conductor
4. REQUIRED TOOLS: Bash (타임아웃 360000ms)
5. MUST DO:
   - nlm research start "{query}" --notebook-id {id} --mode deep --profile {profile}
   - nlm research status {notebook_id} --profile {profile} (자동 polling)
   - nlm research import {notebook_id} --profile {profile}
   - nlm source list {notebook_id} 로 임포트 확인
6. MUST NOT DO:
   - nlm research start --query 형식 금지 (query는 positional arg)
   - nlm research status --research-id 형식 금지 (notebook_id가 positional arg)
   - nlm research list/result/sources 금지 (존재하지 않는 명령어)
7. CONTEXT: deep 모드 ~5분, fast 모드 ~30초. status 명령은 자동 polling 수행
```

### Phase 4: Slides Generation (slides-creator)

```
1. TASK: 노트북 소스 기반 슬라이드 생성
2. EXPECTED OUTCOME: 슬라이드 생성 완료 및 다운로드 안내
3. REQUIRED SKILLS: slides-creator
4. REQUIRED TOOLS: Bash (타임아웃 300000ms)
5. MUST DO:
   - nlm source list {notebook_id} 로 소스 존재 확인
   - nlm slides create {notebook_id} --format presenter_slides --language ko --length default --confirm --profile {profile}
   - nlm studio status {notebook_id} 로 생성 완료 확인
   - nlm download slides {notebook_id} 다운로드 명령어 안내
6. MUST NOT DO:
   - nlm slides create --notebook-id 형식 금지 (positional arg)
   - nlm slides status/list/download/share 금지 (존재하지 않는 명령어)
7. CONTEXT: format은 presenter_slides 또는 detailed_deck. 기본 언어 ko
```

## Interview Flow

파이프라인 시작 전 사용자로부터 수집할 정보:

### Required
1. **Research Topic** (연구 주제): 구체적이고 상세한 리서치 주제
2. **Notebook Title** (노트북 제목): 리서치 프로젝트 노트북 이름

### Optional (with defaults)
3. **Profile** (프로필): `work` (기본값)
4. **Research Mode** (리서치 모드): `deep` (기본값)
5. **Slide Format** (슬라이드 형식): `presenter_slides` (기본값)
6. **Slide Language** (슬라이드 언어): `ko` (기본값)
7. **Slide Length** (슬라이드 길이): `default` (기본값)

## Error Handling

| 에러 | 조치 |
|------|------|
| 인증 만료 | 사용자에게 `nlm login --profile {profile}` 안내 후 재시도 |
| 리서치 타임아웃 | `nlm research status {id} --max-wait 0` 으로 단일 체크 |
| 소스 미발견 | 더 넓은 검색어 제안, 재시도 |
| 슬라이드 생성 실패 | 소스 임포트 확인 후 재시도 |
| nlm 미설치 | nlm CLI 설치 안내 |

## Completion Report

```
리서치 완료 보고서
━━━━━━━━━━━━━━━━━━━
노트북: {title} ({notebook_id})
리서치: {source_count}개 소스 발견 및 임포트 완료
슬라이드: 생성 완료

다음 단계:
- nlm download slides {notebook_id} --profile {profile}
- NotebookLM 웹에서 직접 확인 가능
```

## Completion Checklist

- [ ] Authentication verified
- [ ] Notebook created/selected with ID
- [ ] Research completed and all sources imported
- [ ] Slides generated successfully
- [ ] User informed of results and download method

# NotebookLM Researcher Skill

NotebookLM을 활용하여 자동화된 리서치부터 슬라이드 생성까지 전체 워크플로우를 오케스트레이션하는 메인 스킬입니다.

## Overview

이 스킬은 다음 워크플로우를 자동으로 실행합니다:

```
Auth Check → Notebook Create → Deep Research → Import Sources → Create Slides
```

## Workflow Steps

### 1. Authentication Check
- nlm CLI 인증 상태 확인
- 프로필 검증
- 필요시 재인증 안내

### 2. Notebook Management
- 기존 노트북 목록 조회
- 새 노트북 생성
- 사용자가 노트북 선택

### 3. Deep Research Execution
- 사용자 입력 주제로 딥 리서치 시작
- 최대 40개 소스 수집 (deep mode)
- 리서치 진행 상황 모니터링
- 완료 대기

### 4. Source Import
- 수집된 소스를 노트북에 임포트
- 임포트 상태 확인

### 5. Slides Generation
- 사용자 선택 옵션으로 슬라이드 생성
- Presenter Slides 또는 Detailed Deck 형식
- 언어 및 길이 설정
- 생성된 슬라이드 다운로드

## Usage

### Automatic Trigger
Claude Code에서 자연어로 요청:

```
리서치해줘: AI 에이전트 프레임워크 비교 분석
```

### Manual Trigger
직접 스킬 호출:

```
/notebooklm-researcher AI 에이전트 프레임워크 비교 분석
```

## Configuration

스킬 설정을 통해 기본값 지정:

| Option | Default | Description |
|--------|---------|-------------|
| `profile` | `work` | nlm 인증 프로필 |
| `researchMode` | `deep` | `deep` (5분, 40소스) / `fast` (30초, 10소스) |
| `slideFormat` | `presenter_slides` | `presenter_slides` / `detailed_deck` |
| `slideLanguage` | `ko` | BCP-47 언어 코드 |
| `slideLength` | `default` | `default` / `short` |

## Agents Orchestrated

| Agent | Role | Trigger Point |
|-------|------|----------------|
| `auth-checker` | 초기 인증 확인 | 워크플로우 시작 |
| `notebook-manager` | 노트북 생성/선택 | 인증 완료 후 |
| `research-conductor` | 딥 리서치 실행/모니터링 | 노트북 선택 후 |
| `slides-creator` | 슬라이드 생성 | 리서치 완료 후 |

## Error Handling

- **인증 실패**: 사용자에게 `nlm login --profile work` 실행 안내
- **노트북 생성 실패**: 기존 노트북 사용 옵션 제공
- **리서치 타임아웃**: 진행 상황까지 저장 후 수동 완료 옵션
- **슬라이드 생성 실패**: 재시도 또는 수동 생성 안내

## Output

- 생성된 슬라이드 다운로드 링크
- 리서치 요약 (수집된 소스 수, 주요 주제)
- 슬라이드 상세 정보 (형식, 언어, 슬라이드 수)

## Prerequisites

- nlm CLI v1.0.0 이상 설치
- NotebookLM 계정 및 인증

## Troubleshooting

### 인증 문제
```bash
# 프로필 로그인
nlm login --profile work

# 프로필 상태 확인
nlm profile info --profile work
```

### 리서치 진행 상황 확인
```bash
# 진행 중인 리서치 목록
nlm research list --profile work
```

### 수동 슬라이드 생성
```bash
# 노트북에서 직접 슬라이드 생성
nlm slides create --notebook-id {notebook_id} --profile work
```

## API Reference

### Input Parameters
- `topic` (required): 리서치할 주제
- `profile` (optional): nlm 프로필 (기본값: work)
- `researchMode` (optional): deep/fast (기본값: deep)
- `slideFormat` (optional): presenter_slides/detailed_deck (기본값: presenter_slides)
- `slideLanguage` (optional): 언어 코드 (기본값: ko)
- `slideLength` (optional): default/short (기본값: default)

### Output
```json
{
  "success": true,
  "notebook": {
    "id": "notebook_id",
    "name": "리서치 주제"
  },
  "research": {
    "status": "completed",
    "sources": 40,
    "duration": "5m"
  },
  "slides": {
    "format": "presenter_slides",
    "language": "ko",
    "downloadUrl": "https://..."
  }
}
```

## Related Documentation

- [auth-checker](../agents/auth-checker/AGENT.md) - 인증 확인 에이전트
- [notebook-manager](../agents/notebook-manager/AGENT.md) - 노트북 관리 에이전트
- [research-conductor](../agents/research-conductor/AGENT.md) - 리서치 수행 에이전트
- [slides-creator](../agents/slides-creator/AGENT.md) - 슬라이드 생성 에이전트

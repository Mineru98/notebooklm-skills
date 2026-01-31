# NotebookLM Research Skills

NotebookLM 리서치 자동화 플러그인 - `nlm` CLI를 활용한 리서치부터 슬라이드 생성까지 자동화합니다.

## Overview

이 플러그인은 Google NotebookLM을 활용하여 다음 워크플로우를 자동화합니다:

1. **인증 확인** - nlm CLI 프로필 인증 상태 확인
2. **노트북 생성** - 리서치 프로젝트용 노트북 생성
3. **딥 리서치** - 웹 소스 자동 탐색 및 수집 (최대 40개 소스)
4. **슬라이드 생성** - 수집된 소스 기반 발표 자료 자동 생성

## Prerequisites

- [nlm CLI](https://github.com/Mineru98/notebooklm-cli) 설치
- NotebookLM 인증 완료 (`nlm login --profile work`)

## Installation

```bash
nlm skill install https://github.com/Mineru98/notebooklm-skills
```

## Usage

Claude Code에서 다음과 같이 사용:

```
리서치해줘: AI 에이전트 프레임워크 비교 분석
```

또는 직접 스킬 호출:

```
/notebooklm-researcher AI 에이전트 프레임워크 비교 분석
```

## Workflow

```
Auth Check → Notebook Create → Deep Research → Import Sources → Create Slides
```

## Plugin Structure

```
notebooklm-skills/
├── .claude-plugin/
│   ├── marketplace.json      # 마켓플레이스 메타데이터
│   └── plugin.json           # 플러그인 설정
├── skills/
│   └── notebooklm-researcher/
│       └── SKILL.md          # 메인 오케스트레이션 스킬
├── agents/
│   ├── auth-checker/
│   │   └── AGENT.md          # 인증 확인 에이전트
│   ├── notebook-manager/
│   │   └── AGENT.md          # 노트북 관리 에이전트
│   ├── research-conductor/
│   │   └── AGENT.md          # 리서치 수행 에이전트
│   └── slides-creator/
│       └── AGENT.md          # 슬라이드 생성 에이전트
└── README.md
```

## Agents

| Agent | Role |
|-------|------|
| **auth-checker** | nlm CLI 인증 상태 확인 및 재인증 안내 |
| **notebook-manager** | 노트북 목록 조회, 생성, 선택 |
| **research-conductor** | 딥 리서치 시작, 상태 모니터링, 소스 임포트 |
| **slides-creator** | 발표 슬라이드 생성 및 다운로드 안내 |

## Configuration

| Option | Default | Description |
|--------|---------|-------------|
| Profile | `work` | nlm 인증 프로필 |
| Research Mode | `deep` | `deep` (5분, 40소스) / `fast` (30초, 10소스) |
| Slide Format | `presenter_slides` | `presenter_slides` / `detailed_deck` |
| Slide Language | `ko` | BCP-47 언어 코드 |
| Slide Length | `default` | `default` / `short` |

## License

MIT

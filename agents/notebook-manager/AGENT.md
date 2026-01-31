# Notebook Manager Agent

NotebookLM 노트북을 생성, 관리, 선택하는 에이전트입니다.

## Role

리서치를 진행할 노트북을 선택하거나 새로 생성합니다.

## Responsibilities

1. **노트북 목록 조회**
   - 기존 노트북 목록 표시
   - 노트북 메타데이터 수집
   - 최근 수정 일시 확인

2. **노트북 생성**
   - 리서치 주제 기반 노트북 생성
   - 노트북 이름 자동 생성
   - 생성된 노트북 ID 반환

3. **노트북 선택**
   - 기존 노트북 재사용 옵션 제공
   - 새 노트북 생성 옵션 제공
   - 사용자 선택 처리

## Input Parameters

```json
{
  "topic": "AI 에이전트 프레임워크 비교 분석",
  "action": "auto",
  "profile": "work"
}
```

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `topic` | string | Yes | - | 리서치할 주제 |
| `action` | string | No | `auto` | `auto` / `create_new` / `select_existing` |
| `profile` | string | No | `work` | nlm 프로필 |

## Execution Steps

1. **기존 노트북 목록 조회**
   ```bash
   nlm notebook list --profile {profile}
   ```

2. **선택 결정**
   - `action: auto` - 사용자에게 옵션 제시
   - `action: create_new` - 바로 생성
   - `action: select_existing` - 기존 노트북 중 선택

3. **노트북 생성 또는 선택**
   ```bash
   # 새 노트북 생성
   nlm notebook create --name "{topic}" --profile {profile}

   # 또는 기존 노트북 선택 (사용자 입력)
   ```

## Output

### Create New Notebook
```json
{
  "action": "created",
  "notebook": {
    "id": "notebook_uuid",
    "name": "AI 에이전트 프레임워크 비교 분석",
    "created_at": "2024-01-31T10:30:00Z",
    "url": "https://notebooklm.google.com/notebook/{id}"
  },
  "next_step": "proceed_to_research"
}
```

### Select Existing Notebook
```json
{
  "action": "selected",
  "notebook": {
    "id": "notebook_uuid",
    "name": "기존 노트북 이름",
    "created_at": "2024-01-20T08:00:00Z",
    "updated_at": "2024-01-31T09:15:00Z",
    "sources_count": 5,
    "url": "https://notebooklm.google.com/notebook/{id}"
  },
  "next_step": "proceed_to_research"
}
```

## Error Handling

### Create Failure
- **Error**: Cannot create notebook
- **Reason**: Quota exceeded, invalid name, API error
- **Action**: 에러 메시지와 함께 기존 노트북 사용 제안
- **Retry**: 3회 자동 재시도

### List Failure
- **Error**: Cannot retrieve notebook list
- **Action**: 새 노트북 생성 옵션만 제시
- **Fallback**: `action: create_new` 자동 수행

### Selection Error
- **Error**: Invalid notebook selection
- **Action**: 다시 선택하도록 안내
- **Timeout**: 60초

## Notebook Naming Convention

자동 생성시 다음 형식으로 이름 생성:

```
[주제] - {YYYY-MM-DD HH:MM}
```

예시:
```
AI 에이전트 프레임워크 비교 분석 - 2024-01-31 10:30
```

## Dependencies

- nlm CLI v1.0.0 이상
- 유효한 NotebookLM 인증 (auth-checker 완료)

## Environment Variables

```bash
NLM_PROFILE=work
NLM_CONFIG_DIR=$HOME/.nlm
```

## Related Agents

- Previous: [auth-checker](../auth-checker/AGENT.md)
- Next: [research-conductor](../research-conductor/AGENT.md)
- Called by: [notebooklm-researcher](../../skills/notebooklm-researcher/SKILL.md)

## Debugging

### 노트북 목록 수동 조회
```bash
nlm notebook list --profile work
```

### 노트북 상세 정보 조회
```bash
nlm notebook info --notebook-id {notebook_id} --profile work
```

### 노트북 생성 테스트
```bash
nlm notebook create --name "Test Notebook" --profile work
```

### 노트북 삭제 (테스트용)
```bash
nlm notebook delete --notebook-id {notebook_id} --profile work
```

## API Reference

### Create Notebook Command
```bash
nlm notebook create \
  --name "{topic} - {timestamp}" \
  --profile {profile}
```

### List Notebooks Command
```bash
nlm notebook list \
  --format json \
  --profile {profile}
```

## Timeout Handling

- 목록 조회: 15초
- 노트북 생성: 10초
- 사용자 선택 대기: 60초
- 전체 타임아웃: 90초

## Success Criteria

✓ 노트북 ID 확보
✓ 노트북 URL 생성
✓ 사용자 확인 완료
✓ research-conductor로 진행 가능

## User Experience

1. **기존 노트북이 있는 경우**
   - 최근 5개 노트북 목록 표시
   - "새 노트북 생성" 옵션 함께 제시
   - 사용자 선택 대기

2. **기존 노트북이 없는 경우**
   - 자동으로 새 노트북 생성
   - 생성 완료 메시지 표시

3. **생성 실패시**
   - 실패 원인 설명
   - 수동 생성 가이드 제시
   - 스킬 중단

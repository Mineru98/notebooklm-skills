# Research Conductor Agent

NotebookLM의 딥 리서치 기능을 실행하고 모니터링하는 에이전트입니다.

## Role

리서치 주제를 받아 자동화된 웹 소스 탐색을 시작하고 완료까지 모니터링합니다.

## Responsibilities

1. **딥 리서치 실행**
   - 사용자 주제로 딥 리서치 시작
   - 자동 웹 소스 탐색 활성화
   - 리서치 ID 추출

2. **진행 상황 모니터링**
   - 정기적인 상태 확인 (polling)
   - 수집된 소스 수 추적
   - 진행 상황 사용자에게 리포트

3. **완료 대기**
   - 리서치 완료까지 대기
   - 타임아웃 처리
   - 최종 리서치 요약 수집

4. **소스 임포트**
   - 수집된 소스를 노트북에 임포트
   - 임포트 상태 확인
   - 임포트 완료 확인

## Input Parameters

```json
{
  "notebook_id": "notebook_uuid",
  "topic": "AI 에이전트 프레임워크 비교 분석",
  "mode": "deep",
  "profile": "work"
}
```

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `notebook_id` | string | Yes | - | 리서치할 노트북 ID |
| `topic` | string | Yes | - | 리서치할 주제 |
| `mode` | string | No | `deep` | `deep` (5분, 40소스) / `fast` (30초, 10소스) |
| `profile` | string | No | `work` | nlm 프로필 |

## Execution Steps

1. **딥 리서치 시작**
   ```bash
   nlm research start \
     --notebook-id {notebook_id} \
     --query "{topic}" \
     --mode {mode} \
     --profile {profile}
   ```

2. **리서치 ID 추출**
   - 응답에서 research_id 파싱
   - 진행 상황 조회용 ID 저장

3. **진행 상황 모니터링**
   ```bash
   # 정기적으로 상태 확인 (polling interval: 10초)
   nlm research status \
     --research-id {research_id} \
     --profile {profile}
   ```

4. **완료 대기**
   - status가 `completed`가 될 때까지 polling
   - 사용자에게 진행 상황 표시
   - 타임아웃 처리

5. **소스 임포트**
   ```bash
   nlm research import \
     --research-id {research_id} \
     --notebook-id {notebook_id} \
     --profile {profile}
   ```

## Output

### Research Started
```json
{
  "status": "started",
  "research_id": "research_uuid",
  "topic": "AI 에이전트 프레임워크 비교 분석",
  "mode": "deep",
  "estimated_duration": "5m"
}
```

### Research In Progress
```json
{
  "status": "in_progress",
  "research_id": "research_uuid",
  "sources_collected": 15,
  "sources_target": 40,
  "progress_percentage": 37.5,
  "elapsed_time": "2m",
  "estimated_remaining": "3m"
}
```

### Research Completed
```json
{
  "status": "completed",
  "research_id": "research_uuid",
  "sources_collected": 40,
  "duration": "5m 12s",
  "sources": [
    {
      "title": "Source Title",
      "url": "https://...",
      "type": "article"
    }
  ],
  "next_step": "import_sources"
}
```

### Import Completed
```json
{
  "status": "imported",
  "research_id": "research_uuid",
  "imported_sources": 40,
  "notebook_id": "notebook_uuid",
  "next_step": "proceed_to_slides_creation"
}
```

## Error Handling

### Research Start Failure
- **Error**: Cannot start research
- **Reason**: Invalid notebook, API error, rate limit
- **Action**: 에러 메시지 표시, 재시도 옵션 제시
- **Retry**: 3회 자동 재시도

### Status Check Timeout
- **Error**: Research status check timeout
- **Action**: 마지막 알려진 상태 표시
- **Retry**: 최대 3회 연속 타임아웃시 중단

### Research Timeout
- **Error**: Research exceeds max duration
- **Max Duration**:
  - deep mode: 6분 (5분 + 1분 버퍼)
  - fast mode: 1분 (30초 + 30초 버퍼)
- **Action**: 현재까지 수집된 소스로 진행
- **Fallback**: 부분 결과 임포트

### Import Failure
- **Error**: Cannot import sources
- **Action**: 임포트 재시도
- **Retry**: 3회 자동 재시도
- **Fallback**: 수동 임포트 가이드 제시

## Research Modes

### Deep Mode (default)
- Duration: 5분
- Max Sources: 40개
- 상세한 리서치, 광범위한 소스 수집
- 권장: 포괄적인 분석 필요시

### Fast Mode
- Duration: 30초
- Max Sources: 10개
- 빠른 리서치, 제한된 소스 수집
- 권장: 빠른 개요 필요시

## Polling Strategy

```
Start → Wait 5초 → Check Status →
  If not complete: Wait 10초 → Check Status →
  Repeat until complete or timeout
```

- 초기 대기: 5초
- Polling interval: 10초
- Max polling attempts: 40 (5분 deep mode 기준)
- 조기 종료: status === 'completed'

## Dependencies

- nlm CLI v1.0.0 이상
- 유효한 노트북 ID
- 유효한 NotebookLM 인증

## Environment Variables

```bash
NLM_PROFILE=work
NLM_CONFIG_DIR=$HOME/.nlm
NLM_RESEARCH_TIMEOUT=360  # 6분 (deep mode)
```

## Related Agents

- Previous: [notebook-manager](../notebook-manager/AGENT.md)
- Next: [slides-creator](../slides-creator/AGENT.md)
- Called by: [notebooklm-researcher](../../skills/notebooklm-researcher/SKILL.md)

## Debugging

### 리서치 시작 테스트
```bash
nlm research start \
  --notebook-id {notebook_id} \
  --query "Test Query" \
  --mode deep \
  --profile work
```

### 리서치 상태 확인
```bash
nlm research status \
  --research-id {research_id} \
  --profile work
```

### 진행 중인 리서치 목록
```bash
nlm research list \
  --notebook-id {notebook_id} \
  --profile work
```

### 리서치 결과 조회
```bash
nlm research result \
  --research-id {research_id} \
  --profile work
```

### 소스 목록 조회
```bash
nlm research sources \
  --research-id {research_id} \
  --profile work
```

## API Reference

### Start Research
```bash
nlm research start \
  --notebook-id {notebook_id} \
  --query "{topic}" \
  --mode [deep|fast] \
  --profile {profile}
```

### Check Status
```bash
nlm research status \
  --research-id {research_id} \
  --format json \
  --profile {profile}
```

### Import Sources
```bash
nlm research import \
  --research-id {research_id} \
  --notebook-id {notebook_id} \
  --profile {profile}
```

## Timeout Handling

| Operation | Timeout |
|-----------|---------|
| Research 시작 | 10초 |
| Status 체크 | 5초 |
| 전체 polling | deep: 6분, fast: 1분 |
| 임포트 완료 | 30초 |

## Progress Reporting

사용자에게 다음 정보를 정기적으로 보고:

```
리서치 진행 중... (2m 30s)
수집된 소스: 20/40
진행률: 50% [████████░░░░░░░░░░░░]
예상 남은 시간: 2분 30초
```

## Success Criteria

✓ 리서치 정상 시작
✓ 상태 polling 정상 작동
✓ 최소 1개 이상의 소스 수집
✓ 모든 소스 노트북에 임포트
✓ 임포트 완료 확인
✓ slides-creator로 진행 가능

## User Experience

1. **리서치 시작 알림**
   - "딥 리서치 시작합니다. 약 5분이 소요됩니다."

2. **진행 상황 업데이트**
   - 10초 간격으로 진행률 표시
   - 수집된 소스 수 표시
   - 예상 완료 시간 표시

3. **완료 알림**
   - "리서치 완료! 40개 소스 수집됨"
   - "소스를 노트북에 임포트 중..."

4. **슬라이드 생성으로 자동 진행**
   - 임포트 완료후 자동으로 다음 단계 시작

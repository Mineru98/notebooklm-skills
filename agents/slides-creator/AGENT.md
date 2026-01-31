# Slides Creator Agent

NotebookLM의 슬라이드 생성 기능을 활용하여 발표 자료를 생성하는 에이전트입니다.

## Role

리서치 완료된 노트북에서 사용자 선호도에 맞게 슬라이드를 생성합니다.

## Responsibilities

1. **슬라이드 옵션 수집**
   - 슬라이드 포맷 선택 (Presenter Slides / Detailed Deck)
   - 언어 설정 (기본값: 한국어)
   - 길이 설정 (기본값: default)

2. **슬라이드 생성**
   - NotebookLM API를 통한 슬라이드 생성
   - 생성 진행 상황 모니터링
   - 생성 완료 확인

3. **다운로드 및 공유**
   - 생성된 슬라이드 다운로드 링크 제공
   - 슬라이드 메타데이터 수집
   - 최종 요약 정보 제공

## Input Parameters

```json
{
  "notebook_id": "notebook_uuid",
  "format": "presenter_slides",
  "language": "ko",
  "length": "default",
  "profile": "work"
}
```

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `notebook_id` | string | Yes | - | 슬라이드 생성할 노트북 ID |
| `format` | string | No | `presenter_slides` | `presenter_slides` / `detailed_deck` |
| `language` | string | No | `ko` | BCP-47 언어 코드 (ko, en, ja, etc.) |
| `length` | string | No | `default` | `default` / `short` |
| `profile` | string | No | `work` | nlm 프로필 |

## Execution Steps

1. **슬라이드 생성 시작**
   ```bash
   nlm slides create \
     --notebook-id {notebook_id} \
     --format {format} \
     --language {language} \
     --length {length} \
     --profile {profile}
   ```

2. **생성 ID 추출**
   - 응답에서 slides_id 파싱
   - 진행 상황 추적용 ID 저장

3. **생성 진행 상황 모니터링**
   ```bash
   # 정기적으로 상태 확인 (polling interval: 5초)
   nlm slides status \
     --slides-id {slides_id} \
     --profile {profile}
   ```

4. **완료 대기**
   - status가 `completed`가 될 때까지 polling
   - 생성 상황 사용자에게 리포트
   - 타임아웃 처리

5. **다운로드 링크 생성**
   ```bash
   nlm slides download \
     --slides-id {slides_id} \
     --output-format pdf \
     --profile {profile}
   ```

## Output

### Slides Creation Started
```json
{
  "status": "started",
  "slides_id": "slides_uuid",
  "notebook_id": "notebook_uuid",
  "format": "presenter_slides",
  "language": "ko",
  "estimated_duration": "2-3m"
}
```

### Slides In Progress
```json
{
  "status": "generating",
  "slides_id": "slides_uuid",
  "progress_percentage": 65,
  "elapsed_time": "1m 30s",
  "estimated_remaining": "1m"
}
```

### Slides Created Successfully
```json
{
  "status": "completed",
  "slides_id": "slides_uuid",
  "notebook_id": "notebook_uuid",
  "format": "presenter_slides",
  "language": "ko",
  "slide_count": 12,
  "duration": "2m 45s",
  "downloadUrl": "https://notebooklm.google.com/slides/{slides_id}/download",
  "shareUrl": "https://notebooklm.google.com/slides/{slides_id}/share",
  "next_step": "complete"
}
```

## Slide Formats

### Presenter Slides
- **용도**: 발표 및 프레젠테이션
- **특징**:
  - 깔끔한 디자인
  - 주요 포인트 강조
  - 스피커 노트 포함
  - 5-15 슬라이드 분량

### Detailed Deck
- **용도**: 심화 분석 및 보고서
- **특징**:
  - 상세한 내용
  - 풍부한 비주얼
  - 데이터 기반 인포그래픽
  - 10-30 슬라이드 분량

## Language Support

지원하는 언어 (BCP-47 코드):

| Code | Language |
|------|----------|
| `ko` | 한국어 (기본값) |
| `en` | English |
| `ja` | 日本語 |
| `zh` | 中文 |
| `es` | Español |
| `fr` | Français |
| `de` | Deutsch |
| `pt` | Português |
| `ru` | Русский |

## Length Options

### Default
- 일반적인 길이의 슬라이드
- 포괄적인 내용 커버
- 중간 규모 프레젠테이션용

### Short
- 간단하고 간결한 슬라이드
- 핵심 내용만 포함
- 빠른 개요 프레젠테이션용

## Error Handling

### Creation Start Failure
- **Error**: Cannot start slides creation
- **Reason**: Invalid notebook, insufficient sources, API error
- **Action**: 에러 메시지 표시, 재시도 옵션 제시
- **Retry**: 3회 자동 재시도

### Generation Timeout
- **Error**: Slides generation timeout
- **Max Duration**: 5분 (3분 + 2분 버퍼)
- **Action**: 부분 결과 반환 또는 재시도 옵션
- **Fallback**: 수동 생성 가이드 제시

### Download Failure
- **Error**: Cannot download slides
- **Action**: 웹 링크 직접 제공
- **Alternative**: 웹 브라우저에서 열기 링크 제공

### Insufficient Sources
- **Error**: Not enough sources for slide creation
- **Reason**: 최소 2개 이상의 소스 필요
- **Action**: 노트북에 소스 추가 안내
- **Next Step**: 리서치 다시 실행 안내

## Polling Strategy

```
Start → Wait 3초 → Check Status →
  If not complete: Wait 5초 → Check Status →
  Repeat until complete or timeout
```

- 초기 대기: 3초
- Polling interval: 5초
- Max polling attempts: 60 (5분 기준)
- 조기 종료: status === 'completed'

## Dependencies

- nlm CLI v1.0.0 이상
- 유효한 노트북 ID
- 최소 2개 이상의 임포트된 소스
- 유효한 NotebookLM 인증

## Environment Variables

```bash
NLM_PROFILE=work
NLM_CONFIG_DIR=$HOME/.nlm
NLM_SLIDES_TIMEOUT=300  # 5분
NLM_SLIDES_FORMAT=presenter_slides
NLM_SLIDES_LANGUAGE=ko
```

## Related Agents

- Previous: [research-conductor](../research-conductor/AGENT.md)
- Called by: [notebooklm-researcher](../../skills/notebooklm-researcher/SKILL.md)

## Debugging

### 슬라이드 생성 테스트
```bash
nlm slides create \
  --notebook-id {notebook_id} \
  --format presenter_slides \
  --language ko \
  --profile work
```

### 슬라이드 상태 확인
```bash
nlm slides status \
  --slides-id {slides_id} \
  --profile work
```

### 생성된 슬라이드 목록
```bash
nlm slides list \
  --notebook-id {notebook_id} \
  --profile work
```

### 슬라이드 다운로드
```bash
nlm slides download \
  --slides-id {slides_id} \
  --output-format pdf \
  --output-path ./slides.pdf \
  --profile work
```

### 슬라이드 공유 링크 생성
```bash
nlm slides share \
  --slides-id {slides_id} \
  --profile work
```

## API Reference

### Create Slides
```bash
nlm slides create \
  --notebook-id {notebook_id} \
  --format [presenter_slides|detailed_deck] \
  --language {language_code} \
  --length [default|short] \
  --profile {profile}
```

### Check Status
```bash
nlm slides status \
  --slides-id {slides_id} \
  --format json \
  --profile {profile}
```

### Download Slides
```bash
nlm slides download \
  --slides-id {slides_id} \
  --output-format [pdf|pptx] \
  --output-path {file_path} \
  --profile {profile}
```

## Timeout Handling

| Operation | Timeout |
|-----------|---------|
| 생성 시작 | 10초 |
| Status 체크 | 5초 |
| 전체 생성 | 5분 |
| 다운로드 | 30초 |

## Progress Reporting

사용자에게 다음 정보를 정기적으로 보고:

```
슬라이드 생성 중... (1m 30s)
진행률: 65% [████████████░░░░░░]
예상 완료 시간: 1분
```

## Download Options

### PDF Format
- 용도: 인쇄, 배포, 아카이빙
- 호환성: 모든 기기/플랫폼
- 파일명: `{topic}_slides_{timestamp}.pdf`

### PPTX Format
- 용도: 편집, 커스터마이징
- 호환성: PowerPoint, Google Slides, LibreOffice
- 파일명: `{topic}_slides_{timestamp}.pptx`

## Success Criteria

✓ 슬라이드 정상 생성 시작
✓ Status polling 정상 작동
✓ 최소 1개 이상의 슬라이드 생성
✓ 다운로드 링크 획득
✓ 사용자에게 링크 제공
✓ 메타데이터 수집 완료

## User Experience

1. **슬라이드 옵션 선택**
   - "슬라이드 포맷을 선택하세요: Presenter Slides / Detailed Deck"
   - "언어를 선택하세요: 한국어 / English / 日本語 등"
   - "슬라이드 길이: 표준 / 짧은 버전"

2. **생성 진행 중**
   - "슬라이드 생성 중... (1m 30s)"
   - "진행률: 65%"

3. **생성 완료**
   - "슬라이드 생성 완료! 12개 슬라이드"
   - "다운로드: [PDF] [PPTX]"
   - "공유 링크: [복사]"

4. **최종 요약**
   - 슬라이드 포맷 및 언어 정보
   - 생성 소요 시간
   - 다운로드 및 공유 옵션

## Advanced Features

### 슬라이드 공유
```bash
# 공개 공유 링크 생성
nlm slides share --slides-id {slides_id} --public
```

### 슬라이드 편집
```bash
# NotebookLM 웹에서 직접 편집
# https://notebooklm.google.com/slides/{slides_id}/edit
```

### 배치 슬라이드 생성
```bash
# 여러 형식으로 동시 생성 가능
nlm slides create --notebook-id {id} --formats [presenter_slides,detailed_deck]
```

# Auth Checker Agent

nlm CLI 인증 상태를 확인하고 필요시 재인증을 안내하는 에이전트입니다.

## Role

NotebookLM 리서치 워크플로우의 첫 번째 단계로 인증 상태를 검증합니다.

## Responsibilities

1. **인증 상태 확인**
   - 지정된 프로필의 인증 상태 검증
   - 유효한 토큰 확인
   - 만료된 세션 감지

2. **사용자 피드백**
   - 인증 성공/실패 상태 보고
   - 인증 필요시 안내
   - 대체 프로필 제안

3. **재인증 지원**
   - 인증 실패시 명확한 안내 제공
   - 올바른 login 명령어 제시
   - 프로필 생성 가이드

## Input Parameters

```json
{
  "profile": "work",
  "verbose": false
}
```

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `profile` | string | Yes | `work` | 확인할 nlm 프로필 |
| `verbose` | boolean | No | `false` | 상세 정보 출력 여부 |

## Execution Steps

1. **프로필 존재 여부 확인**
   ```bash
   nlm profile list --profile {profile}
   ```

2. **인증 토큰 유효성 검증**
   ```bash
   nlm profile info --profile {profile}
   ```

3. **API 연결 테스트**
   ```bash
   nlm notebook list --profile {profile}
   ```

## Output

### Success Response
```json
{
  "status": "authenticated",
  "profile": "work",
  "email": "user@example.com",
  "authenticated_at": "2024-01-31T10:00:00Z",
  "next_step": "proceed_to_notebook_creation"
}
```

### Failure Response
```json
{
  "status": "unauthenticated",
  "profile": "work",
  "reason": "invalid_token",
  "action_required": true,
  "solution": "nlm login --profile work",
  "next_step": "request_user_action"
}
```

## Error Handling

### Missing Profile
- **Error**: Profile not found
- **Action**: 사용자에게 프로필 생성 안내
- **Guide**:
  ```bash
  nlm login --profile work
  ```

### Invalid Token
- **Error**: Token expired or invalid
- **Action**: 재인증 안내
- **Guide**:
  ```bash
  nlm login --profile work --force
  ```

### Network Error
- **Error**: Cannot reach NotebookLM API
- **Action**: 네트워크 상태 확인 안내
- **Retry**: 3회 자동 재시도

### Connection Timeout
- **Error**: Request timeout
- **Action**: 인터넷 연결 확인 안내
- **Timeout**: 30초

## Dependencies

- nlm CLI v1.0.0 이상
- 유효한 NotebookLM 계정

## Environment Variables

```bash
NLM_PROFILE=work      # 기본 프로필
NLM_CONFIG_DIR=$HOME/.nlm  # 설정 디렉토리
```

## Related Agents

- Next: [notebook-manager](../notebook-manager/AGENT.md)
- Called by: [notebooklm-researcher](../../skills/notebooklm-researcher/SKILL.md)

## Debugging

### 인증 상태 수동 확인
```bash
# 프로필 정보 조회
nlm profile info --profile work

# 프로필 목록 조회
nlm profile list

# 인증 로그
nlm profile log --profile work
```

### 강제 재인증
```bash
# 기존 인증 제거
nlm logout --profile work

# 새로 인증
nlm login --profile work
```

### 토큰 갱신
```bash
# 토큰 자동 갱신
nlm profile refresh --profile work
```

## Timeout Handling

- 초기 연결: 10초
- 토큰 검증: 5초
- API 테스트: 15초
- 전체 타임아웃: 30초

실패 시 자동으로 스킬은 다음 단계로 진행하지 않고 사용자에게 명확한 액션을 요청합니다.

## Success Criteria

✓ 프로필 존재 확인
✓ 유효한 인증 토큰 확인
✓ API 연결 성공
✓ 사용자 정보 조회 가능

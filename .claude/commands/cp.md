---
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git branch:*), Bash(git fetch:*), Bash(git rebase:*), Bash(git add:*), Bash(git commit:*), Bash(git push:*)
description: origin rebase 체크 → 스테이징 → conventional commit → push 자동화
---

## 현재 상태

- 브랜치: !`git branch --show-current`
- 변경 파일: !`git status --short`
- Staged 변경사항: !`git diff --cached --stat`
- Unstaged 변경사항: !`git diff --stat`
- 전체 diff: !`git diff HEAD`
- 최근 커밋 5개: !`git log --oneline -5`

## 작업 절차

### Step 1 — Origin 동기화 확인

아래 명령을 순서대로 실행한다:

1. `git fetch origin` — 원격 최신 상태 가져오기
2. `git rev-list --left-right --count HEAD...origin/$(git branch --show-current) 2>/dev/null` — 커밋 차이 확인

결과 형식은 `A B` (A = 내가 앞선 수, B = origin이 앞선 수):

- **B > 0** (origin이 앞서 있음) → `git rebase origin/<현재 브랜치>` 실행
  - conflict 발생 시: 즉시 중단하고 충돌 파일 목록을 사용자에게 알린 뒤 종료
  - rebase 성공 시: 다음 단계로 진행
- **B = 0** 또는 upstream tracking 브랜치 없음 → 이 단계 스킵

### Step 2 — 파일 스테이징

`git status --short` 결과를 기준으로:

- staged 파일(앞 글자가 `M`, `A`, `D`, `R` 등)이 이미 있으면 → 그대로 사용
- staged 파일이 없고 변경사항이 있으면 → `git add .` 실행
- 변경사항이 전혀 없으면 → "커밋할 변경사항이 없습니다" 출력 후 종료

### Step 3 — Conventional Commit 메시지 결정

`git diff HEAD` 내용을 분석해 아래 타입 중 가장 적합한 것을 선택:

| 타입 | 사용 시점 |
|------|-----------|
| `feat` | 새 기능 추가 |
| `fix` | 버그 수정 |
| `docs` | 문서만 변경 |
| `style` | 코드 로직 변경 없는 포맷 수정 |
| `refactor` | 리팩토링 (기능 변경 없음) |
| `test` | 테스트 추가·수정 |
| `chore` | 빌드/패키지/설정 변경 |
| `perf` | 성능 개선 |
| `ci` | CI 파이프라인 변경 |

**메시지 형식:** `<타입>(<scope>): <요약>`

- scope: 변경된 영역 (선택사항, 예: `auth`, `api`, `ui`, `config`)
- 요약: 현재형 동사로 시작, 영문 50자 이내
- 예시: `feat(auth): add OAuth2 login flow`, `fix(api): handle null response from user endpoint`

### Step 4 — 커밋 실행

`git commit -m "<메시지>"` 실행.

### Step 5 — Push

`git push origin <현재 브랜치>` 실행.
- upstream tracking 브랜치가 없으면 `git push -u origin <현재 브랜치>` 실행
- push 성공 후 `git log --oneline -3` 을 출력해 결과를 확인한다.

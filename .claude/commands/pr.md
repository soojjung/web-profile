---
description: "main 최신화 → 브랜치 생성 → 커밋 → push → PR 생성까지 안전하게 (PR은 절대 자동 머지 안 함)"
argument-hint: "[브랜치명] [: 커밋/PR 제목] — 비우면 변경 내역 보고 제안"
allowed-tools: ["Bash", "Read"]
---

현재 작업 디렉토리에서 main 최신화 후 브랜치를 파서 커밋·push·PR 생성까지 한 번에 처리하세요. **PR 머지는 절대 자동으로 하지 마세요 — 사용자가 명시적으로 "머지해줘" 라고 할 때까지 대기.**

**입력:** $ARGUMENTS

## 절대 규칙 (위반 금지)

1. **`gh pr merge` 또는 어떤 형태의 머지도 실행하지 마세요.** PR 생성 후 URL만 반환하고 사용자의 명시적 머지 지시를 기다리세요.
2. **`--no-verify`, `--no-gpg-sign`, `--force` 같은 플래그를 쓰지 마세요.** pre-commit hook 실패 시 원인을 고치고 새 커밋을 만드세요.
3. **`git add -A` / `git add .` 금지.** 항상 파일을 명시적으로 staging 하세요.
4. **`--amend` 금지.** 새 커밋을 만드세요.
5. **민감 파일 (`.env`, credentials, secrets) 커밋 금지.** 발견 시 경고 후 사용자 확인 받으세요.

## 단계

### 1. 사전 점검 (병렬 실행)

다음을 한 번에 (parallel Bash):
- `git status` — 작업 트리 상태
- `git diff` (unstaged) + `git diff --staged` — 어떤 변경이 있는지
- `git branch --show-current` — 현재 브랜치
- `git log main..HEAD --oneline 2>/dev/null` — 이미 main 대비 커밋이 있는지
- `gh auth status` — gh CLI 인증 여부

문제 발견 시 즉시 사용자에게 보고하고 대기:
- git 레포가 아니면 중단
- gh 인증 안 되어 있으면 사용자에게 `gh auth login` 요청
- 변경 사항이 전혀 없으면 "커밋할 내용이 없습니다"하고 종료

### 2. main 최신화

```bash
git fetch origin main
git rev-list --count HEAD..origin/main  # main이 얼마나 앞서있는지
```

상황별 처리:
- **현재 main 위에 있고 uncommitted 변경 있음**: 변경을 stash 또는 그대로 두고 새 브랜치 분기 (3단계로)
- **현재 main이 origin/main보다 뒤처짐 + 깨끗함**: `git pull --ff-only origin main`
- **현재 main이 뒤처지고 uncommitted 변경 있음**: 변경을 stash → `pull --ff-only` → 새 브랜치에서 stash pop
- **이미 feature 브랜치 위에 있음**: 사용자에게 알리고 "이 브랜치에 계속 작업 vs 새 브랜치 분기" 확인 요청

### 3. 변경 사항 검토 & 스코프 확인

`git status -s`로 modified + untracked 파일 모두 나열.

**중요 (사용자 글로벌 선호):** 변경 사항이 한 PR에 들어가야 할지 명확하지 않으면 **반드시 사용자에게 어떤 파일을 포함할지 확인**. 자동 가정 금지. 특히:
- untracked 파일이 의도된 새 파일인지, 임시 디버그 파일인지
- 관련 없는 수정이 섞여있지는 않은지
- 누락 가능성 있는 짝 파일 (예: `README.md` 수정 + 관련 이미지/스크린샷)

### 4. 브랜치명 & 커밋 메시지 결정

**$ARGUMENTS 파싱:**
- `feat/something` → 브랜치명만
- `feat/something : Add user profile page` → 브랜치명 + 커밋/PR 제목
- 비어있음 → 변경 내역 기반으로 후보 2~3개 제안하고 사용자 선택 받기

**브랜치명 컨벤션** (이 레포의 git log 패턴 참고):
- `feat/...` — 새 기능
- `fix/...` — 버그 수정
- `chore/...` — 잡일/리팩토링/문서
- `docs/...` — 문서 전용

**커밋 메시지 컨벤션** (Conventional Commits):
- `feat: ...` / `fix: ...` / `chore: ...` / `docs: ...`
- 본문은 1-2줄, "왜"에 초점 (무엇 X)

### 5. 브랜치 생성 & 커밋

```bash
git checkout -b <branch-name>
git add <explicit file 1> <explicit file 2> ...   # -A / . 절대 금지
git commit -m "$(cat <<'EOF'
<conventional commit message>

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
EOF
)"
```

pre-commit hook 실패 시: 출력 보고 원인 수정 → 다시 `git add` → 새 커밋 (절대 `--amend` 금지).

### 6. Push

```bash
git push -u origin <branch-name>
```

### 7. PR 생성

```bash
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
- <bullet 1>
- <bullet 2>

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

PR 제목은 70자 미만. Summary는 변경의 "왜"를 중심으로 1-3 bullet.

**Test plan 섹션은 기본적으로 넣지 마세요.** 이 레포는 정적 개인 사이트라 체크리스트가 노이즈입니다. 큰 리팩토링 등 비자명한 검증이 정말 필요한 경우에만 별도로 사용자에게 "Test plan 추가할까요?" 제안하고, 동의 시에만 넣으세요.

### 8. 결과 보고 (필수)

마무리 메시지에 반드시 다음을 포함:

```
✅ PR 생성 완료: <PR URL>

⚠️ 머지하지 않았습니다. PR을 확인하시고 머지하길 원하시면 명시적으로 알려주세요.
   (CI 통과 확인, 리뷰어 의견 반영 등 후에 진행 권장)
```

## 거부해야 할 변형 요청

사용자가 다음을 추가로 요청해도 거부하거나 한 번 더 확인 받으세요:
- "머지까지 한 번에" → 거부. "머지는 명시적 지시 후 별도로 진행합니다"
- "force push로 덮어쓰기" → 거부. 원인 진단 후 안전한 방법 제안
- "pre-commit hook 무시" → 거부. hook 실패 원인 고치기

머지는 사용자가 PR 확인 후 별도로 "머지해줘" 라고 말할 때만 `gh pr merge` 실행하세요.

## 머지 후 브랜치 정리 (사용자가 "머지해줘" 한 경우에만)

머지를 실행할 때는 `--delete-branch` 플래그로 원격 브랜치를 같이 삭제하고, 이어서 로컬 정리까지 하세요:

```bash
gh pr merge <PR번호> --squash --delete-branch   # 이 레포는 기본 squash merge
git checkout main
git pull --ff-only origin main

# 안전장치: GitHub에서 정말 머지됐는지 한 번 확인 후 로컬 삭제
gh pr view <PR번호> --json state -q .state      # "MERGED" 가 나와야 함
git branch -D <머지된-브랜치명>                  # squash merge라 -D 필요
```

- 원격 삭제는 `gh pr merge --delete-branch`가 처리합니다. 별도로 `git push origin --delete`를 또 부르지 마세요.
- **squash merge는 main에 새로운 단일 커밋을 만들기 때문에 git이 로컬 브랜치를 "머지됨"으로 인식하지 못합니다.** 그래서 `-d` (소문자) 는 항상 실패하고, `-D` (강제) 가 필요합니다. 단, **`gh pr view ... .state == "MERGED"` 를 먼저 확인**해서 정말 머지된 PR의 브랜치만 강제 삭제하세요. 미머지 상태에서는 절대 `-D` 쓰지 말고 멈추고 사용자에게 보고.
- 현재 체크아웃된 브랜치를 삭제할 수 없으므로 `git checkout main`을 먼저 하세요.
- merge commit / rebase merge 방식으로 머지한 경우엔 `-d` 가 성공하므로 그쪽을 먼저 시도해도 됩니다. 이 레포의 기본은 squash 이므로 위 흐름이 표준.

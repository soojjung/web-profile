# Soojin Jung Profile

🔗 link: https://www.sooya.dev/

---

## 🛠 Resume Custom Commands (Claude Code)

이력서 관련 작업을 위한 4개의 커스텀 슬래시 명령어. `.claude/commands/`에 정의되어 있고, 각각 전용 서브에이전트를 호출합니다.

| 명령어    | 역할                                                |
| --------- | --------------------------------------------------- |
| `/rv`     | 이력서 단독 종합 리뷰 (10단계, ATS·임팩트 점수)     |
| `/match`  | 이력서 ↔ JD 적합도 평가 (Gap·인터뷰 확률)           |
| `/tailor` | 마스터 이력서를 JD에 맞춰 ATS 최적화 재구성         |
| `/apply`  | 풀 파이프라인 (`match` → `tailor` → `rv` 자동 순차) |

### 권장 사용 순서

1. **`/rv <이력서>`** — 먼저 마스터 이력서의 베이스라인 품질을 확인
2. **`/match <이력서> <JD>`** — 지원하려는 공고와의 적합도와 Gap 파악
3. **`/tailor <이력서> <JD>`** — JD에 맞춰 이력서 재구성 (사실 기반, 날조 금지)
4. **`/rv <재구성된 이력서>`** — 최종 품질 재검증

> 한 채용공고에 빠르게 지원할 때는 `/apply <이력서> <JD>` 하나로 1~4단계를 자동 실행할 수 있습니다.

### `<이력서>` / `<JD>` 입력 형식

세 가지 모두 지원하며, 섞어 써도 됩니다.

| 형식            | 예시                                                                    |
| --------------- | ----------------------------------------------------------------------- |
| **파일 경로**   | `~/Desktop/resume.pdf`, `./resume.md`, `/Users/.../이력서.docx`         |
| **URL**         | `https://www.linkedin.com/jobs/view/12345`, `https://jobs.lever.co/...` |
| **본문 텍스트** | 명령어 뒤에 그대로 붙여넣기                                             |

⚠️ LinkedIn JD는 로그인 벽 때문에 URL 가져오기가 실패할 수 있습니다 — 그 경우 JD 본문을 복사해 붙여넣어 주세요.

---

<table>
  <tr>
    <td align="center"><h4>v2.0</h4></td>
    <td align="center"><h4>v1.0</h4></td>
  </tr>
  <tr>
    <td><img src="./images/thumbnail_v2.png" alt="thumbnail v2" width="280px"></td>
    <td><img src="./images/thumbnail.png" alt="thumbnail v1" width="280px"></td>
  </tr>
</table>

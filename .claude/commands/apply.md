---
description: "지원 풀 파이프라인: 매칭 평가 → 맞춤 재구성 → 최종 리뷰를 순차 실행"
argument-hint: "[마스터 이력서 + JD] — 한 채용공고에 지원하기 위한 풀패키지"
allowed-tools: ["Agent", "Read", "WebFetch", "Write"]
---

세 개의 에이전트를 순차적으로 실행해 한 채용공고에 지원하기 위한 풀 패키지를 만들어주세요.

**입력:** $ARGUMENTS

## 파이프라인 (순차 실행 — 이전 단계 결과가 다음 단계 입력)

### Step 0: 입력 확보
- **마스터 이력서**와 **JD**를 확보 (파일 경로 → Read, URL → WebFetch, 본문 → 그대로). 둘 중 하나 없으면 즉시 요청하고 중단.

### Step 1: 매칭 평가 (resume-job-match-evaluator)
- 현재 마스터 이력서가 이 JD와 얼마나 맞는지 baseline 평가
- Agent 도구로 `resume-job-match-evaluator` 실행
- 결과에서 **핵심 Gap 3-5개**, **ATS 점수**, **인터뷰 확률**을 추출해 사용자에게 요약 보고
- ⚠️ 사용자에게 "재구성을 진행할까요?" 확인받고 다음 단계로

### Step 2: 맞춤 재구성 (resume-tailoring-strategist)
- 사용자 동의 후 진행
- Agent 도구로 `resume-tailoring-strategist` 실행
- 입력: 마스터 이력서 + JD + Step 1에서 발견된 Gap
- 결과: 재구성된 이력서 전문 + 변경 사유

### Step 3: 최종 리뷰 (resume-reviewer-swe)
- Step 2의 재구성된 이력서를 입력
- Agent 도구로 `resume-reviewer-swe` 실행
- 10단계 프레임워크로 최종 품질 검증
- 타겟 역할은 JD의 직무명을 그대로 사용

### Step 4: 최종 패키지 보고
사용자에게 다음을 한 응답으로 정리해 전달:
1. **Before/After 매칭 점수 비교** (Step 1 baseline vs. Step 3에서 재평가가 가능하면 비교)
2. **재구성된 이력서 전문** (Step 2 결과)
3. **남은 약점 3가지** + 즉시 액션 (Step 3 결과)
4. (선택) `resume-tailored-{회사명}.md` 파일로 저장할지 묻기

## 주의사항
- 각 에이전트는 **반드시 Agent 도구**로 별도 실행 (메인 컨텍스트 보호)
- 세 에이전트 모두 **사실 날조 금지**를 핵심 원칙으로 함 — 어기지 말 것
- 언어는 JD를 기준으로 한국어/영어 일관성 유지
- 사용자가 중간에 멈추라고 하면 다음 단계로 넘어가지 말 것

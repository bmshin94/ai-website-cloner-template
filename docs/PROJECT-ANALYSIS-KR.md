# AI Website Cloner Template — 전수조사 분석 & 활용 가이드 (한국어)

> 이 문서는 `ai-website-cloner-template` 레포지토리를 파일 단위로 전수조사한 결과와,
> 설치·사용법, 정체(스킬/플러그인/MCP) 판별, 수익화 전략, 프레임워크 이식, 강의 제작 가능성까지
> 정리한 종합 분석 문서입니다.

## 📍 레포지토리 주소

| 구분 | 주소 |
| --- | --- |
| **원본 레포 (upstream)** | https://github.com/JCodesMore/ai-website-cloner-template |
| **내 레포 (fork / template 사본)** | https://github.com/bmshin94/ai-website-cloner-template |
| 템플릿 생성 링크 | https://github.com/JCodesMore/ai-website-cloner-template/generate |
| 데모 영상 | https://youtu.be/O669pVZ_qr0 |
| 커뮤니티 (Discord) | https://discord.gg/hrTSX5yTpB |
| 라이선스 | MIT (Copyright (c) 2025 JCodesMore) |
| 분석 기준 버전 | v0.5.0 (커밋 53개, 컨트리뷰터 10명) |
| 분석 일자 | 2026-10-08 |

---

## 1. 이게 뭐하는 프로젝트인가

### 한 줄 요약

> **URL 하나를 던지면 AI 코딩 에이전트가 그 웹사이트를 픽셀 단위로 똑같이 Next.js 코드로 재구현해주는 템플릿.**

핵심은 **이것이 "프로그램"이 아니라는 점**이다. 실행 파일도, 라이브러리도, 서버도 없다.
**AI 에이전트가 읽는 "초정밀 작업 지시서"(506줄) + "미리 세팅된 빈 작업장"(Next.js 스캐폴드)** 의 조합이다.

```
📖 레시피책(SKILL.md) + 🍽️ 주방(src/) + 👨‍🍳 AI 에이전트 = 🍝 완성된 사이트
                                            ↑
                                   이게 없으면 아무 일도 안 일어남
```

### 전체 용량 1.6MB — 코드가 거의 없는 이유

본체가 **코드가 아니라 문서**이기 때문이다.

---

## 2. 폴더 전수조사 결과

### ① 심장부 — `.agents/skills/clone-website/`

| 파일 | 줄 수 | 역할 |
| --- | --- | --- |
| `SKILL.md` | **506줄** | 진짜 본체. 복제 파이프라인 전체 지시서 |
| `references/inspection-guide.md` | 80줄 | 사이트 조사용 5단계 체크리스트 |

이 506줄이 레포 가치의 90%다. 담고 있는 내용:

- **Pre-Flight** — 브라우저 MCP 확인 → URL 검증 → `npm run build` 통과 → 기존 라우트 인벤토리 → **출력 계획서(output plan) 작성 후 사용자 승인**
- **Phase 1 정찰** — 1440px/390px 풀페이지 스크린샷, 폰트·컬러·파비콘 추출, **상호작용 스윕**(스크롤·클릭·호버·반응형 4종), `BEHAVIORS.md` + `PAGE_TOPOLOGY.md` 산출
- **Phase 2 기초공사** — 폰트/글로벌 CSS/타입/SVG 아이콘/에셋 다운로드 (**반드시 순차**)
- **Phase 3 명세 & 분산** — 섹션별 `추출 → .spec.md 작성 → 빌더 에이전트 투입 → 머지` 루프
- **Phase 4 조립** — 섹션 컴포넌트 합쳐 페이지 완성
- **Phase 5 시각 QA** — 원본 vs 복제본 나란히 비교, 불일치 시 재추출

### ② 클로드 코드용 연결선 — `.claude/commands/clone-website.md`

본문이 사실상 1문장: *"위 SKILL.md를 전부 읽고 그대로 수행해라."*
→ **중복 관리를 피하려고 의도적으로 얇게 설계한 브릿지.**

### ③ 미리 깔아둔 빈 작업장 — `src/`

```
src/app/page.tsx       ← "Clone target not yet built. Run /clone-website to start."
src/app/layout.tsx     ← Geist / Geist Mono 폰트 세팅
src/app/globals.css    ← shadcn oklch 디자인 토큰 전부
src/components/ui/button.tsx  ← shadcn 버튼 1개 (@base-ui/react 기반)
src/lib/utils.ts       ← cn() 유틸
src/hooks/.gitkeep     ← 빈 폴더 (AI가 채울 자리)
src/types/.gitkeep     ← 빈 폴더
```

### ④ 결과물 저장소 (전부 빈 폴더)

```
public/images/  public/videos/  public/seo/   ← 원본 사이트에서 다운로드한 에셋
docs/research/                                 ← 추출한 명세서(spec)들
docs/design-references/                        ← 스크린샷
scripts/                                       ← 에셋 다운로드 스크립트
```

### ⑤ 품질 보증 장치

| 파일 | 역할 |
| --- | --- |
| `.github/workflows/ci.yml` | lint → typecheck → build 3단 검증 (Node 24, checkout@v7) |
| `Dockerfile` / `Dockerfile.dev` | 3-스테이지 프로덕션 빌드, non-root(`node`) 유저, standalone 출력 |
| `docker-compose.yml` | `app`(3000) / `dev`(3001) + healthcheck + `.env` 옵셔널 |
| `AGENTS.md` | 에이전트 공통 규칙 (단일 진실 공급원) |
| `CLAUDE.md` | `@AGENTS.md` import + 페르소나 설정 |
| `SECURITY.md` | GitHub 비공개 취약점 신고 경로 |
| `.github/ISSUE_TEMPLATE/`, `PULL_REQUEST_TEMPLATE.md` | 기여 프로세스 |

---

## 3. 기술 스택

```
Next.js 16.3.0 (App Router) + React 19.2.4 + TypeScript strict
shadcn/ui 4 (@base-ui/react 1.3) + Tailwind CSS v4 (oklch 토큰)
lucide-react 1.6 / Node 24+ / MIT 라이선스
next.config.ts → output: "standalone"
```

> ⚠️ `AGENTS.md` 최상단 경고: **"This is NOT the Next.js you know"**
> Next.js 16은 breaking change가 있으므로 코드 작성 전 `node_modules/next/dist/docs/` 를 반드시 읽어야 한다.
> 또한 현재 `node_modules`가 없는 상태이므로 **`npm install` 선행 필수**.

---

## 4. 작동 아키텍처 — "현장 감독(Foreman)" 패턴

SKILL.md 원문: *"You are a **foreman walking the job site**"*

```
        [메인 에이전트 = 현장 감독]
                  │
    브라우저 MCP로 섹션 1 조사
    → hero.spec.md 작성 (정확한 CSS 수치 전부)
    → 빌더 에이전트 #1 투입 (git worktree 안에서)
                  │ ← 기다리지 않고 바로 다음 섹션으로
    섹션 2 조사 → spec 작성 → 빌더 #2 투입
    섹션 3 조사 → spec 작성 → 빌더 #3, #4 투입
                  │
         [빌더들 병렬 작업 중...]
                  ↓
         worktree 머지 → npm run build 검증
                  ↓
         페이지 조립 → 시각 QA 비교 → 완료
```

### 설계 철학 9가지

1. **완성도 > 속도** — 빌더가 색·폰트크기·여백 하나라도 추측하면 "추출 실패"로 간주
2. **작은 작업 = 완벽한 결과** — *"spec이 150줄 넘으면 무조건 쪼개라. 이건 기계적 체크다"*
3. **진짜 콘텐츠, 진짜 에셋** — 목업 금지. 실제 텍스트/이미지/비디오 다운로드. **레이어드 에셋 주의**
4. **기초 먼저** — 폰트/CSS/타입은 순차(비협상), 이후부터 병렬
5. **외형 AND 동작 둘 다** — *"웹사이트는 스크린샷이 아니라 살아있는 생명체다"*
6. **인터랙션 모델 먼저 판별** — 클릭 기반 vs 스크롤 기반 혼동이 최대 비용. *"절대 먼저 클릭하지 말고 먼저 스크롤해봐라"*
7. **모든 상태 추출** — 탭 4개면 4개 다 클릭해서 각각 추출, 스크롤 0 / 100+ 양쪽 캡처
8. **spec 파일이 진실의 원천** — 감사 가능한 산출물로 영구 보존
9. **빌드는 항상 통과** — 빌더마다 `npx tsc --noEmit`, 머지마다 `npm run build`

### "What NOT to Do" — 실패에서 나온 교훈 14개 (요약)

- 스크롤 기반을 클릭 탭으로 만들기 (**1등 비싼 실수 — 전면 재작성 필요**)
- 기본 상태만 추출하기
- **겹쳐진 레이어 이미지 놓치기** (배경 + 전경 UI목업 = 2장인데 1장만 받으면 텅 빈 것처럼 보임)
- 비디오/Lottie/canvas인데 HTML 목업으로 만들기
- CSS 클래스 어림짐작 (`text-lg` 추정 vs 실제 line-height 24px)
- **Lenis/Locomotive 같은 스무스 스크롤 라이브러리 놓치기**
- 빌더 프롬프트에 "DESIGN_TOKENS.md 참고해"라고 쓰기 → **인라인 전부 삽입 필수**
- spec 파일 없이 빌더 투입하기
- 반응형 추출 생략 (1440/768/390 필수)
- 모놀리식 커밋 하나로 전부 처리하기
- 새 타겟을 기존 앱 교체 허가로 착각하기

### 충돌 방지 네임스페이싱

```
<site-key> = 읽기쉬운 origin 슬러그 + SHA-256(normalized origin) 앞 8자리(소문자 hex)
<page-key> = 경로 보존 슬러그 + SHA-256(pathname + stateful query/fragment) 앞 8자리
             (루트는 root-<hash>)

docs/research/<site-key>/<page-key>/
docs/design-references/<site-key>/<page-key>/
src/components/sites/<site-key>/<page-key>/     (공용은 .../shared/)
public/sites/<site-key>/<page-key>/
scripts/download-assets-<site-key>-<page-key>.mjs
```

원문 경고: *"Never rely on lossy character replacement alone"* — 단순 문자 치환만으로는 충돌 위험.

---

## 5. 언제 쓰는가 / 쓰면 안 되는가

### ✅ 정당한 용도 (README 명시)

| 상황 | 설명 |
| --- | --- |
| **플랫폼 이사** | 내가 소유한 WordPress/Webflow/Squarespace → 모던 Next.js |
| **소스코드 분실** | 사이트는 살아있는데 레포 없음 / 개발자 퇴사 / 레거시 스택 → 코드 복구 |
| **학습** | 프로덕션 사이트의 레이아웃·애니메이션·반응형을 실제 코드로 해부 |

### 🚫 금지 용도 (README 직접 금지)

- **피싱/사칭** — 사기 목적, 위법 행위
- **남의 디자인을 내 것처럼 팔기** — 로고·브랜드 에셋·오리지널 카피는 소유자 것
- **약관 위반** — 스크래핑/복제 금지 사이트는 사전 확인 필수

---

## 6. 설치 및 사용법

### 사전 준비물

| 준비물 | 필수 | 비고 |
| --- | --- | --- |
| Node.js 24+ | ✅ | `.nvmrc` = 24, engines `>=24` |
| AI 코딩 에이전트 | ✅ | Claude Code(권장, Opus 5) / Codex CLI / Cursor / OpenCode |
| **브라우저 MCP** | ✅ **절대 필수** | Chrome MCP(1순위) / Playwright / Browserbase / Puppeteer |
| 서브에이전트 지원 | ✅ | 병렬 빌더 투입용 (Aider가 지원 목록에서 빠진 이유) |
| Git | ✅ | worktree 사용 |

> SKILL.md Pre-Flight 1번: *"This skill cannot work without browser automation."*

### 설치 방법

**방법 A — AI에게 시키기 (README 추천)**

```text
Set up https://github.com/JCodesMore/ai-website-cloner-template
as a standalone project in a new folder on my computer.
Ask me where to put it. Clone the repository, remove its origin remote,
install dependencies, and run npm run check. Leave it ready for me
to clone a website.
```

**방법 B — GitHub 템플릿 버튼**
`https://github.com/JCodesMore/ai-website-cloner-template/generate` → 이름 입력 → Create repository

**방법 C — 수동**

```bash
git clone https://github.com/JCodesMore/ai-website-cloner-template.git
cd ai-website-cloner-template
git remote remove origin     # 원본 레포 오염 방지
npm install
npm run check
```

### Docker

```bash
docker compose up app --build   # 프로덕션, 3000 포트
docker compose up dev --build   # 개발모드, 3001 포트 (볼륨 마운트)
```

### 사용

```text
/clone-website https://example.com
/clone-website https://example.com https://example.com/pricing https://example.com/docs
```

### 개발 명령어

```bash
npm run dev        # 개발 서버
npm run build      # 프로덕션 빌드
npm run lint       # ESLint
npm run typecheck  # tsc --noEmit
npm run check      # 위 3개 전부 (이것만 기억하면 됨)
```

---

## 7. 플러그인인가, 스킬인가, MCP인가

### 결론: **스킬(Skill)이 담긴 프로젝트 템플릿.** MCP는 아니다.

| 구분 | 해당? | 근거 |
| --- | --- | --- |
| MCP 서버 | ❌ | MCP 서버는 도구를 *제공*하는 프로세스. 이건 MCP를 *소비*하는 쪽 |
| 플러그인 | ❌ | `.claude-plugin/plugin.json` 없음, 마켓플레이스 배포 아님 |
| **스킬** | ✅ | `.agents/skills/clone-website/SKILL.md` + frontmatter(name/description) |
| 슬래시 커맨드 | ✅ 부분 | `.claude/commands/clone-website.md` — 얇은 브릿지 |
| **프로젝트 템플릿** | ✅ | GitHub Template 레포 + Next.js 스캐폴드 |

### MCP와의 관계

```
[이 스킬 SKILL.md]  "브라우저로 사이트를 조사해라"
        │ 호출
        ▼
[브라우저 MCP 서버]  (별도 설치 필요)
  Chrome MCP / Playwright MCP 등
  → screenshot, evaluate, click, scroll
```

→ **이 스킬은 MCP의 "고객"이다. MCP가 없으면 작동 자체가 불가능.**

### 설계의 묘미 (v0.5.0 변경사항)

```
이전: Claude용 + Codex용 + Cursor용 + Kiro용 + Cline용 + Roo용 스킬 복사본
      + 동기화 스크립트 + CI 검증 게이트  → 관리 지옥
현재: .agents/skills/ 하나 + 얇은 브릿지 1개  → 끝
```

---

## 8. API 토큰이 필요한가

### 결론: **이 레포 자체는 토큰 0개. 단, AI 에이전트 비용은 발생.**

전수조사 결과:
- `.env.example` 없음, 코드에 `process.env.*_API_KEY` 없음
- `docker-compose.yml`은 `.env`/`.env.local`을 `required: false`로 옵셔널 처리
- `.gitignore`에 `.env*` 전부 포함 → 크레덴셜 커밋 위험 없음

### 실제 비용이 발생하는 지점

| 항목 | 비용 |
| --- | --- |
| Claude Code 구독 | Pro $20/월 ~ Max $100-200/월 (README 권장: Opus 5) |
| 또는 Anthropic API | 토큰 종량제 |
| Codex CLI / Cursor | ChatGPT 구독 / $20월 |
| 브라우저 MCP (Chrome, Playwright) | 보통 무료 |
| Browserbase (클라우드 브라우저) | 유료 |
| Vercel 배포 | 무료 티어 가능 |

### ⚠️ 토큰 소모량 주의

```
사이트 1회 복제 =
  스크린샷 다수(이미지 토큰 큼) + 섹션별 getComputedStyle JSON
+ spec 파일 작성 + 빌더 N명 × 150줄 프롬프트 + 시각 QA 비교
= 복잡한 사이트 1건에 수십만 토큰
```

API 종량제 사용 시 **비용 모니터링 필수**.

---

## 9. AI 에이전트 구축에 도움이 되는가

### 결론: **매우 크게 도움 된다. 오히려 이쪽이 본질적 가치.**

이 레포는 "웹사이트 클론 도구"보다 **"멀티에이전트 설계 교과서"** 로 보는 편이 정확하다.

### 바로 재사용 가능한 설계 패턴 10가지

| # | 패턴 | 출처 | 왜 중요한가 |
| --- | --- | --- | --- |
| 1 | **Foreman 패턴** (감독 1 + 워커 N) | Phase 3 | 단일 에이전트는 컨텍스트 과부하로 품질 저하. 감독이 압축 명세로 전달 |
| 2 | **Spec-as-Contract** | 원칙 #8 | "추출이 틀렸나 빌드가 틀렸나"를 파일 하나로 즉시 판별 → 디버깅 비용 1/10 |
| 3 | **복잡도 예산 150줄** | 원칙 #2 | "느낌으로" 쪼개면 안 쪼개짐. **숫자로 못 박아야** 실제로 분해됨 |
| 4 | **인라인 전달 원칙** | What NOT to Do | 워커가 외부 문서 읽을 필요가 0이어야 함 |
| 5 | **git worktree 격리** | Step 3 | 병렬 에이전트 최대 적 = 동일 파일 동시 수정. OS 레벨 차단 |
| 6 | **Pre-Flight 승인 게이트** | Phase 0 | 파괴적 작업 전 정지 → 에이전트 신뢰도의 핵심 |
| 7 | **충돌 방지 네임스페이싱** | Output Isolation | 해시 혼합으로 손실 없는 유일성 보장 |
| 8 | **안티패턴 문서화** | What NOT to Do | 에이전트는 "하라"보다 "하지 마라 + 왜"에 더 잘 반응 |
| 9 | **다단계 검증 게이트** | 전체 | tsc → build → CI → 시각 QA 4단 |
| 10 | **크로스 플랫폼 이식성** | `.agents/` 구조 | 벤더 종속 탈출, 플랫폼 추가 시 브릿지 1개만 |

### 다른 도메인에 그대로 이식 가능

| 만들 에이전트 | 적용 방법 |
| --- | --- |
| 레거시 코드 마이그레이션 | 파일별 spec → 병렬 변환 → 테스트 게이트 |
| 대량 문서 번역 | 챕터별 용어집 spec → 병렬 번역 → 일관성 QA |
| API 문서 자동생성 | 엔드포인트별 spec → 병렬 작성 → 스키마 검증 |
| 테스트 코드 자동생성 | 함수별 spec(입출력/엣지케이스) → 병렬 생성 → 커버리지 게이트 |
| 디자인 시스템 추출 | 컴포넌트별 spec → 병렬 구현 → 스토리북 QA |
| 데이터 파이프라인 | 테이블별 spec → 병렬 ETL → 데이터 검증 |

> **핵심: "조사(감독) / 명세(계약) / 실행(병렬) / 검증(게이트)" 4박자는 도메인 무관하게 통한다.**

---

## 10. React / PHP로 만들 수 있는가

### React — 이미 React다

```json
"react": "19.2.4", "react-dom": "19.2.4", "next": "16.3.0"
```
Next.js 16은 React 19 기반 프레임워크이고, 산출물도 전부 `.tsx` React 컴포넌트다.

**순수 React(Vite)로 전환 시 수정 지점**

| 작업 | 수정 위치 | 난이도 |
| --- | --- | --- |
| Next.js 제거 → Vite | `package.json` | 낮음 |
| App Router → React Router | `src/app/` → `src/pages/` | 중간 |
| `next/font` → `@fontsource` | SKILL.md Phase 2 | 낮음 |
| `next/image` → `<img>` | SKILL.md 에셋 섹션 | 낮음 |
| 라우트 경로 규칙 | SKILL.md Output Isolation | 중간 |

### PHP — 가능. SKILL.md 개조 필요

**재사용 가능 범위**

```
100% 재사용 (프레임워크 무관):
  Phase 1 정찰 전체 / 추출 스크립트 / spec.md 템플릿
  설계 원칙 9가지 / 안티패턴 14개 / Phase 5 시각 QA

수정 필요:
  Phase 2 기초공사(폰트/CSS 세팅) / 타겟 파일 경로
  검증 명령(npx tsc --noEmit → php -l, phpstan) / Phase 4 라우팅
```

**Laravel + Blade 매핑안**

| Next.js 버전 | Laravel 버전 |
| --- | --- |
| `src/app/page.tsx` | `routes/web.php` + `resources/views/` |
| `src/components/.../Hero.tsx` | `resources/views/components/hero.blade.php` |
| `public/sites/...` | `public/assets/...` |
| Tailwind v4 | Tailwind v4 (그대로 유지) |
| `npx tsc --noEmit` | `php artisan view:cache` |
| git worktree 병렬 | git worktree 병렬 (동일) |

**WordPress 테마 매핑안**

```
src/components/Hero.tsx  →  template-parts/hero.php
globals.css              →  style.css (테마 헤더 포함)
layout.tsx               →  header.php + footer.php
page.tsx                 →  index.php / front-page.php
```

### 프레임워크별 이식 난이도

| 타겟 | 난이도 | 작업량 | 추천도 |
| --- | --- | --- | --- |
| Next.js (현재) | — | 0 | ★★★★★ |
| React + Vite | 낮음 | 1~2일 | ★★★★ |
| Astro | 낮음 | 1~2일 | ★★★★ |
| Vue / Nuxt | 중간 | 3~5일 | ★★★ |
| Svelte / SvelteKit | 중간 | 3~5일 | ★★★ |
| Laravel + Blade | 중간 | 3~5일 | ★★★★ |
| WordPress 테마 | 중상 | 5~7일 | ★★★★ (수익성) |
| 순수 PHP | 중간 | 3~4일 | ★★ |
| Flutter Web | 높음 | 2주+ | ★ |

### 권장 전략 — 멀티 타겟 스킬로 통합

```markdown
## Target Framework

Ask the user which output framework to use:
- nextjs (default) → src/components/sites/.../*.tsx
- laravel         → resources/views/components/*.blade.php
- wordpress       → template-parts/*.php
- astro           → src/components/*.astro

Then load references/targets/<framework>.md for path and verification rules.
```

→ `references/targets/` 폴더에 프레임워크별 규칙만 추가하면 하나의 스킬로 전부 커버 가능.

---

## 11. 유튜브 강의 영상 제작 가능성

### 결론: 가능. 소재가 좋아 시리즈화 권장.

| 이유 | 설명 |
| --- | --- |
| 극적인 비포/애프터 | URL → 동일한 사이트. 썸네일 자체가 됨 |
| 트렌드 정중앙 | "AI 에이전트" + "바이브 코딩" |
| 사회적 증명 | Trendshift 랭킹 배지 (repo #24302) |
| MIT 라이선스 | 상업적 영상 제작 자유 |
| 깊이 있음 | 툴 소개를 넘어 설계 철학까지 |
| 한국어 콘텐츠 희박 | 선점 기회 |

### ⚠️ 제작 시 지켜야 할 선

```
해도 되는 것:
  - 내가 만든 샘플 사이트 복제 시연
  - 오픈소스 사이트 복제 (라이선스 확인)
  - 복제 금지 명시 없는 사이트 + "학습 목적" 고지
  - 영상 내 "교육 목적, 상업적 사용 금지" 고지

하지 말아야 할 것:
  - "OO 사이트 베껴서 내 서비스 만들기" 식 제목 (조회수는 되지만 법적 위험)
  - 브랜드 로고 그대로 노출된 결과물 홍보
  - "이걸로 돈 벌어요" + 남의 디자인 판매 암시
  - 약관상 스크래핑 금지 사이트 시연
```

> **안전한 프레이밍: "내 사이트를 현대화하는 방법" / "프로덕션 사이트 해부해서 배우기"**

### 추천 8부작 구성

| EP | 제목 | 길이 | 목표 |
| --- | --- | --- | --- |
| 01 | URL 하나로 웹사이트 복제? AI 에이전트 실전 테스트 | 10분 | 후킹 + 구독 유입 |
| 02 | 설치부터 첫 복제까지 완전 가이드 | 15분 | 실용성 (브라우저 MCP 세팅이 최대 관문) |
| 03 | 506줄 지시서 해부 — AI가 이걸 어떻게 읽나 | 20분 | 차별화 |
| 04 | **AI한테 일 제대로 시키는 법 — 멀티에이전트 설계 패턴** | 18분 | **하이라이트. 공유/북마크 최다 예상** |
| 05 | 실패 사례 14개 — 이렇게 하면 망한다 | 15분 | 공감 유발 |
| 06 | Laravel/WordPress 버전으로 개조하기 | 25분 | PHP 시청자 확보 |
| 07 | 이걸로 돈 버는 5가지 방법 (합법적으로) | 15분 | 수익화 + 윤리 교육 |
| 08 | 내 사이트 실전 이전 — 처음부터 끝까지 | 30분 | 완결성/신뢰도 |

### 제작 팁

| 팁 | 이유 |
| --- | --- |
| 분할 화면 (원본 \| 복제본) | 비교가 즉시 전달 |
| AI 대기 시간은 타임랩스 | 실시간은 지루함 |
| 터미널 폰트 18pt+ | 모바일 시청자 다수 |
| spec.md 작성 순간 확대 | 핵심 강조 포인트 |
| worktree 구조는 다이어그램 | 병렬 구조는 말로 전달 불가 |
| 실패 장면 남기기 | 전부 편집하면 "광고" 느낌 |

### 수익 연결 경로

```
무료 영상 → 설명란 → ① 유료 강의(인프런/Udemy)
                   → ② Notion 체크리스트(이메일 수집)
                   → ③ 1:1 컨설팅
                   → ④ 디스코드 커뮤니티
                   → ⑤ 대행 문의 폼
```

---

## 12. 수익화 아이디어 상세

### 전제 조건 (모든 모델 공통)

```
안전지대: "고객이 소유한 사이트"를 다루기
위험지대: "남의 사이트"를 복제해서 팔기  ← 절대 금지
```

모든 계약서에 **"고객이 해당 사이트의 소유권/권리를 보유함을 보증"** 조항 필수.

---

### Tier 1 — 즉시 시작 가능 (초기자본 0원)

#### 아이디어 1. 플랫폼 이전 대행 ★ 최고 추천

> *"Webflow/Squarespace/WordPress 구독료 아깝고 느리시죠? Next.js로 이전해드립니다."*

**타겟 고객**

| 고객 유형 | 고통 | 지불 의지 |
| --- | --- | --- |
| Webflow 스타트업 | 월 $39~212 + 커스텀 한계 | 높음 |
| Squarespace 소상공인 | 느림, SEO 약함, 월 $16~65 | 중간 |
| WordPress 중소기업 | 플러그인 지옥, 보안 취약 | 높음 |
| Shopify 브랜드 | 커스터마이징 한계, 수수료 | 높음 |

**가격 책정 (한국 시장)**

| 패키지 | 범위 | 가격 | 소요 |
| --- | --- | --- | --- |
| 라이트 | 랜딩 1페이지 | 50~100만원 | 1~2일 |
| 스탠다드 | 5페이지 이내 | 150~300만원 | 3~5일 |
| 프로 | 10~20페이지 + 블로그 | 300~600만원 | 1~2주 |
| 엔터프라이즈 | 전체 + CMS + 배포 | 800~1500만원 | 3~4주 |
| 유지보수 애드온 | — | 월 20~50만원 | 계속 |

**영업 킬러 멘트**

```
"현재 Lighthouse 42점입니다. 이전하면 95점 이상 나옵니다.
 구글 검색 순위와 직결됩니다.
 월 구독료 $200 → $0. 1년이면 이전 비용을 뽑습니다."
```

**리스크 대응**

| 리스크 | 대응 |
| --- | --- |
| 백엔드/DB 없음 | "프론트엔드 이전" 범위 명시, CMS는 별도 견적 |
| 복잡한 인터랙션 실패 | 사전 사이트 진단 → 난이도 높으면 추가 견적 |
| 고객이 CMS 원함 | Sanity/Contentful 연동 애드온으로 추가 판매 |
| AI 토큰 비용 | 견적 반영 (실제 몇만원 수준) |

**수익 시뮬레이션**: 월 2건(스탠다드 200만) = 400만원 − 비용 30만 = **순이익 370만원/월**

---

#### 아이디어 2. 유튜브 + 온라인 강의

| 단계 | 상품 | 가격 | 예상 수익 |
| --- | --- | --- | --- |
| 무료 | 유튜브 8부작 | 0원 | 광고 월 10~100만원 |
| 중간 | 인프런/Udemy 강의 | 5~15만원 | 월 50~500만원 |
| 고가 | 1:1 멘토링/컨설팅 | 시간당 10~30만원 | 월 100~300만원 |
| 구독 | 디스코드 멤버십 | 월 1~3만원 | MRR |

**유료 강의 커리큘럼 (8~10시간)**

```
Part 1. 환경 구축 (1h) — Node 24, 브라우저 MCP, 트러블슈팅
Part 2. SKILL.md 완전 해부 (2h) — Phase 0~5, 원칙 9 + 안티패턴 14
Part 3. 실전 복제 3회 (2.5h) — 쉬움/중간/복잡 난이도별
Part 4. 멀티에이전트 설계 패턴 (2h) ← 핵심 차별화
Part 5. 다른 프레임워크 개조 (1.5h) — Laravel/WordPress
Part 6. 수익화 & 법적 가이드 (1h)
```

**보너스 번들**: Laravel 버전 스킬 / WordPress 테마 스킬 / 진단 체크리스트 / 견적서·계약서 템플릿 / 디스코드 평생 멤버십

---

#### 아이디어 3. 사이트 진단 리포트 (리드 마그넷)

```
무료 진단(5분) → 리포트 PDF → "이전하시겠어요?" → 대행 수주
```

**리포트 샘플**

```
현재 플랫폼: WordPress 6.4 + Elementor
Lighthouse: 성능 38 / SEO 72 / 접근성 61
페이지 용량: 8.4MB (권장 2MB 이하)
로딩 시간: 6.2초 / 월 플랫폼 비용: $89

→ Next.js 이전 후 예상
Lighthouse: 성능 96 / SEO 100 / 용량 1.1MB / 로딩 1.1초
월 비용 $0 (Vercel Hobby) / 연간 절감 $1,068
이전 난이도: 중 (5페이지, 애니메이션 多) / 예상 견적 250만원 · 4일
```

**가격**: 무료(리드 수집) 또는 5~15만원(상세 리포트 + 이전 로드맵)
**자동화**: Phase 1 정찰 로직만 분리해 진단 전용 스킬 제작 → 하루 10건 처리 가능

---

### Tier 2 — 전문성 필요 (수익 큼)

#### 아이디어 4. "잃어버린 소스코드" 복구 서비스

> *"사이트는 살아있는데 소스코드가 없으세요? 복구해드립니다."*

고통이 극심한 문제라 **가격 저항이 거의 없다.**

| 상황 | 긴급도 | 가격 저항 |
| --- | --- | --- |
| 외주사 폐업/연락불가 | 매우 높음 | 거의 없음 |
| 개발자 퇴사 + 인수인계 X | 매우 높음 | 거의 없음 |
| 레거시 스택(jQuery, PHP5) | 높음 | 낮음 |
| 호스팅 종료 임박 | 매우 높음 | 없음 |
| M&A 실사 중 코드 필요 | 높음 | 없음 |

**가격**

| 패키지 | 가격 |
| --- | --- |
| 긴급 진단 (복구 가능성 평가) | 50만원 |
| 단일 페이지 복구 | 150~300만원 |
| 전체 사이트 복구 | 500~1500만원 |
| + 문서화 애드온 | +200만원 |
| + 유지보수 | 월 50~150만원 |

> **문서화 애드온이 핵심 마진**: 이 스킬이 자동으로 `docs/research/.../components/*.spec.md` 를 생성하므로,
> 부수 산출물을 그대로 상품화할 수 있다.

**차별화**: 일반 업체는 "HTML 긁어서 제공"(스파게티 코드) → 우리는 "모던 Next.js + TS strict + 컴포넌트 명세서 + CI"

**리스크 대응**: 백엔드 복구 불가 → 범위 명시 / **소유권 증빙 서류 필수**(도메인 소유, 사업자등록) / 동적 데이터는 목 데이터 + "API 연동 별도"

---

#### 아이디어 5. 프리미엄 템플릿 판매

```
금지: 남의 사이트 복제해서 템플릿으로 판매
허용: 복제 스킬로 트렌드 분석 → 오리지널 디자인 고속 제작 → 판매
```

**프로세스**

```
1. 인기 사이트 10개 복제 (학습 목적, 비공개)
2. spec 분석 → "2026년 SaaS 랜딩 공통 패턴" 추출
   (히어로 구조, 여백 스케일, 타이포 비율, 애니메이션 타이밍)
3. 그 패턴으로 완전 오리지널 디자인 제작
4. 복제 파이프라인으로 고속 구현
5. 판매
```

| 채널 | 가격 | 수수료 |
| --- | --- | --- |
| Gumroad | $29~99 | 10% |
| LemonSqueezy | $29~99 | 5%+ |
| ThemeForest | $19~59 | 50~70% |
| 자체 사이트 | $49~149 | 0% |

**수익 시뮬레이션**: 템플릿 5종 × $59 × 월 20개 = **약 $5,900/월 (패시브 인컴)**
**차별화**: 디자인 + 컴포넌트 명세서 + 디자인 토큰 문서 + 상호작용 사양서 동봉

---

#### 아이디어 6. AI 에이전트 설계 컨설팅 ★ 최고 단가

> *"AI 에이전트 도입했는데 결과물이 쓸 수 없어요" 하는 기업에 설계 방법론을 판매.*

```
웹사이트 복제 = 결과물 하나
설계 방법론   = 기업의 모든 AI 프로젝트에 적용 → 가치 10배
```

기업의 현재 상태: "품질이 들쭉날쭉" / "에이전트가 엉뚱한 짓을 함" / "작업 분해 기준을 모름" / "병렬 돌리면 충돌"
→ 이 레포의 10가지 패턴이 전부 답을 갖고 있다.

| 상품 | 가격 |
| --- | --- |
| 반나절 워크샵 (팀 교육) | 200~500만원 |
| 2일 집중 워크샵 | 500~1200만원 |
| 월 리테이너 컨설팅 | 월 300~800만원 |
| 에이전트 설계 감수 | 건당 300~1000만원 |
| 시간당 컨설팅 | 10~30만원 |

**워크샵 커리큘럼**

```
M1. 왜 AI 에이전트 결과물이 나쁜가 (1h) — 컨텍스트 과부하 / 추측 여지 / 검증 부재
M2. Foreman 패턴 (1.5h) — 오케스트레이터 vs 워커, clone-website 해부
M3. Spec-as-Contract (1.5h) — 디버깅 비용 1/10, 실습: 우리 팀 spec 템플릿
M4. 복잡도 예산 & 작업 분해 (1h) — 150줄 규칙, 기계적 기준의 중요성
M5. 병렬 실행 & 격리 (1h) — worktree, 머지 전략, 검증 게이트
M6. 안티패턴 축적 루프 (1h) — 실습: 우리 팀 안티패턴 10개 작성
```

**고객 확보**: 유튜브 EP.04 → 기업 담당자 유입 / 링크드인 패턴 연재 / 밋업·컨퍼런스 발표 / "AI 도입 실패" SEO

---

### Tier 3 — 큰 투자 (큰 리턴)

#### 아이디어 7. SaaS화 — "URL → Next.js 레포 자동 생성"

```
유저: URL 입력 + 결제
  → 서버: 격리 컨테이너에서 에이전트 파이프라인 실행
  → 결과: GitHub 레포 생성 + Vercel 자동 배포 + ZIP 다운로드
```

**스택**

```
프론트:   Next.js + Clerk(인증) + Stripe(결제)
작업 큐:  BullMQ / Inngest / Trigger.dev
실행 환경: E2B / Fly.io Machines / Firecracker + Playwright + Claude Agent SDK
산출:     GitHub API(레포 생성) + Vercel API(배포) + S3(ZIP)
```

**가격 모델**

| 플랜 | 가격 | 포함 |
| --- | --- | --- |
| Free | 0원 | 진단 리포트만 (리드 수집) |
| 단건 | $49~99 | 1페이지 복제 |
| Starter | $29/월 | 월 3페이지 |
| Pro | $99/월 | 월 15페이지 + 우선순위 |
| Agency | $299/월 | 무제한 + 화이트라벨 + API |
| Enterprise | 문의 | 온프레미스, SLA |

**원가 구조**: 1건당 AI 토큰 $3~15 + 컨테이너 $0.5~2 + 스토리지 $0.1 = **$4~17**
→ $49 판매 시 마진 70~90%. 단 복잡한 사이트가 $30 소모할 리스크 존재.

**리스크**

| 리스크 | 심각도 | 대응 |
| --- | --- | --- |
| AI 토큰 비용 변동 | 높음 | 복잡도별 크레딧 차등, 상한선 |
| **법적 책임** (유저가 남의 사이트 복제) | 매우 높음 | **도메인 소유권 검증 필수**, DMCA 대응 |
| 복제 품질 편차 | 중간 | 사전 난이도 평가 → 거절/경고 |
| 봇 차단 사이트 | 낮음 | 실패 시 환불 정책 |
| 경쟁자 진입 | 중간 | 품질 + 커뮤니티로 방어 |

**법적 안전장치 (필수)**

```
1. 도메인 소유권 검증 — DNS TXT 레코드 또는 해당 도메인 이메일 인증
   → "내 사이트만 복제 가능" 강제  ← 핵심 방어선
2. ToS에 유저 권리 보유 보증 + 위반 시 계정 정지/법적 책임 이전
3. DMCA 신고 창구 운영
4. 복제 로그 보관 (분쟁 대응)
```

**MRR 시뮬레이션**
- 6개월차: Starter 50 + Pro 20 = 약 $3,430 MRR
- 1년차: Starter 200 + Pro 80 + Agency 10 = 약 **$16,710 MRR**

---

#### 아이디어 8. 멀티 프레임워크 스킬 팩 판매

원본은 Next.js만 지원 → Laravel/WordPress/Astro/Vue/Svelte/React+Vite 버전 제작 후 판매.

| 상품 | 가격 |
| --- | --- |
| 단일 프레임워크 스킬 | $39~79 |
| 전체 팩 (6종) | $149~249 |
| 팀 라이선스 | $499 |
| + 평생 업데이트 | +$99 |

**왜 팔리는가**: 한국 시장은 PHP/Laravel 개발자와 WordPress 수요가 크지만 원본은 Next.js 전용.
**라이선스**: 원본 MIT → 수정·상업적 판매 가능 (단, 저작권 고지 포함 필수)

---

#### 아이디어 9. 에이전시 화이트라벨 B2B

```
웹에이전시: "빨리 만들고 싶은데 인력이 없음"
  → 우리: 백엔드로 붙어 처리, 에이전시 이름으로 납품
```

| 상품 | 가격 |
| --- | --- |
| 건당 하청 | 50~200만원 (에이전시는 고객에 300~600만원 판매) |
| 월 구독 (월 N건) | 월 500~2000만원 |
| 사내 도입 + 교육 | 초기 1000만원 + 월 300만원 |

**타겟**: 소규모 웹에이전시 / 디자인 스튜디오 / 마케팅 에이전시 / 프리랜서 디자이너
**장점**: 계약 규모 큼, 반복 수주, 영업 비용 낮음, 법적 리스크 낮음(에이전시가 1차 책임)

---

### 아이디어 9개 종합 비교

| # | 아이디어 | 초기자본 | 난이도 | 수익규모 | 법적리스크 | 속도 | 종합 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 플랫폼 이전 대행 | 0원 | 낮음 | 월 400~750만 | 낮음 | 즉시 | ★★★★★ |
| 2 | 유튜브 + 강의 | 0원 | 낮음 | 월 100~800만 | 낮음 | 3~6개월 | ★★★★★ |
| 3 | 진단 리포트 | 0원 | 낮음 | 리드용 | 낮음 | 즉시 | ★★★★ |
| 4 | 소스코드 복구 | 0원 | 중간 | 건당 500~1500만 | 중간 | 즉시 | ★★★★★ |
| 5 | 템플릿 판매 | 시간 | 중간 | 월 300~800만 | 중간 | 2~3개월 | ★★★ |
| 6 | 에이전트 컨설팅 | 0원 | 높음 | 월 300~1200만 | 낮음 | 신뢰구축 필요 | ★★★★★ |
| 7 | SaaS화 | 큼 | 높음 | MRR 2200만+ | 높음 | 6~12개월 | ★★★ |
| 8 | 멀티 프레임워크 팩 | 시간 | 중간 | 월 200~500만 | 낮음 | 1~2개월 | ★★★★ |
| 9 | 에이전시 B2B | 0원 | 중간 | 월 500~2000만 | 낮음 | 영업 필요 | ★★★★ |

---

### 12개월 실행 로드맵

**Month 1-2 — 기반 다지기 + 첫 수익**

```
□ 환경 세팅 (npm install, 브라우저 MCP)
□ SKILL.md 완독 + 완전 이해
□ 연습 복제 5회 (내 사이트, 오픈소스 사이트)
□ 비포/애프터 포트폴리오 3개 (Lighthouse 점수 포함)
□ [3] 진단 리포트 템플릿 제작
□ [1] 크몽/숨고 등록 + 랜딩 제작
□ 지인 사이트 1개 반값 수주 → 첫 레퍼런스
→ 예상 수익 100~300만원
```

**Month 3-4 — 확장 + 콘텐츠**

```
□ [1] 월 2건 수주 체계화
□ [4] 소스코드 복구 서비스 랜딩 추가 (고단가)
□ [2] 유튜브 EP.01~04 제작
□ [8] Laravel 버전 스킬 개조 시작
□ 링크드인/X에 설계 패턴 연재
→ 예상 수익 400~800만원/월
```

**Month 5-6 — 고단가 전환 (노동 집약 → 지식 집약)**

```
□ [2] 유료 강의 출시 (인프런)
□ [8] 멀티 프레임워크 팩 판매 시작
□ [6] 컨설팅 문의 접수 (유튜브 EP.04가 영업 역할)
□ [9] 에이전시 3곳 접촉
→ 예상 수익 800~1500만원/월
```

**Month 7-12 — 스케일**

```
□ [6] 기업 워크샵 정례화 (최고 단가)
□ [9] 에이전시 B2B 계약 2~3곳
□ [7] SaaS MVP 검토 (도메인 소유권 검증 필수)
□ 대행 업무는 외주/팀으로 이전
→ 예상 수익 1500~3000만원/월
```

### 핵심 조언 3가지

1. **"복제"가 아니라 "이전"으로 포지셔닝하라.**
   같은 기술인데 프레이밍 하나로 가격이 2배가 되고, 법적 찝찝함도 사라진다.

2. **진짜 금광은 설계 방법론이다.**
   복제 대행은 시간을 파는 일(한계 있음), 에이전트 설계 컨설팅은 지식을 파는 일(무한 확장).

3. **도메인 소유권 검증은 타협 불가.**
   고객 소유 확인 + 계약서 권리 보증 조항 + 로그 보관 → "피싱 도구" 비난 영구 차단.

---

## 13. 제약사항 체크리스트

| 제약 | 내용 |
| --- | --- |
| 브라우저 MCP 필수 | Chrome/Playwright/Browserbase/Puppeteer MCP 없으면 **작동 불가** |
| 서브에이전트 지원 필요 | 병렬 빌더 투입용 (Aider 지원 제외 이유) |
| 고성능 모델 권장 | README 권장 = Claude Code + Opus 5 |
| 백엔드 복제 불가 | DB/인증/실시간 기능은 명시적 out of scope. **외형만** |
| Node 24+ | 현재 `node_modules` 없음 → `npm install` 선행 필수 |
| 토큰 소모량 큼 | 복잡한 사이트 1건에 수십만 토큰 |
| Next.js 16 breaking change | 코드 작성 전 `node_modules/next/dist/docs/` 확인 |

---

## 14. 최종 요약

> **이 레포는 코드가 아니라 506줄짜리 AI 작업 지시서다.**
> 웹사이트 복제기이면서, 동시에 **멀티에이전트 설계 교과서**다.
>
> 즉시 가치: Next.js 16 + shadcn + Tailwind v4 + Docker + CI가 세팅된 보일러플레이트 + 복제 파이프라인
> 장기 가치: Foreman 패턴 / Spec-as-Contract / 복잡도 예산 / worktree 격리 / 안티패턴 축적 —
>            도메인 무관하게 통하는 AI 에이전트 설계 패턴 10가지
>
> 추천 수익 루트: **플랫폼 이전 대행(1) → 소스코드 복구(4)로 단가 상승 → 에이전트 설계 컨설팅(6)으로 전환**

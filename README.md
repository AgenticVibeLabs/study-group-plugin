# study-group

GitHub 기반 책 스터디 관리 플러그인 for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

스터디 저장소 초기화, PR 기반 챕터 요약 작성, Issue 기반 Q&A를 터미널에서 원스톱으로 수행할 수 있습니다.

---

## 시작하기

### 1. 사전 준비

플러그인을 사용하기 전에 아래 도구들이 설치 및 설정되어 있어야 합니다.

| 도구 | 설치 방법 | 확인 명령어 |
|------|-----------|-------------|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | `npm install -g @anthropic-ai/claude-code` | `claude --version` |
| [GitHub CLI (`gh`)](https://cli.github.com/) | `brew install gh` (macOS) | `gh --version` |
| Git | [설치 가이드](https://git-scm.com/downloads) | `git --version` |

GitHub CLI 인증이 완료되어 있어야 합니다:

```bash
gh auth status
# 인증이 안 되어 있다면:
gh auth login
```

### 2. 플러그인 설치

Claude Code를 실행한 뒤, 아래 두 명령어를 순서대로 입력하세요:

```
/plugin marketplace add AgenticVibeLabs/study-group-plugin
```

```
/plugin install study-group@study-group-plugin
```

설치가 완료되면 `/help`를 입력하여 `/study:init`, `/study:pr`, `/study:issue` 커맨드가 표시되는지 확인하세요.

### 3. 스터디 저장소 초기화

스터디용 GitHub 저장소를 먼저 만들고, 해당 디렉토리에서 Claude Code를 실행합니다:

```bash
mkdir my-study && cd my-study
git init
gh repo create my-org/my-study --public --source=. --remote=origin
claude
```

Claude Code 안에서:

```
/study:init
```

대화형으로 책 제목, 챕터 수, 멤버 목록, 일정, 규칙을 입력하면 저장소가 자동으로 세팅됩니다.

---

## 커맨드

### `/study:init` — 저장소 초기화

스터디 저장소를 대화형으로 초기화합니다.

```
/study:init
```

**수행 내용:**

1. 책 제목, 챕터 범위, 멤버 GitHub ID, 일정, 규칙을 대화형으로 입력
2. `README.md` 생성 (멤버 테이블, 진행 현황 포함)
3. 챕터별 디렉토리 생성 (`ch01/`, `ch02/`, ...)
4. GitHub Issue 템플릿 3종 설치 (질문, 토론, 공지)
5. GitHub PR 템플릿 설치
6. GitHub 라벨 자동 생성 (유형별 + 챕터별)
7. 설정을 `.claude/study-group.local.md`에 저장

### `/study:pr` — 챕터 요약 PR 생성

챕터 원문을 입력하면 체계적 요약을 생성하고 PR까지 자동으로 만듭니다.

```
/study:pr ch01
```

**수행 내용:**

1. 설정 파일에서 책 정보 로드
2. 원문 입력 방식 선택 (클립보드 붙여넣기 / 파일 지정)
3. Technical Writer 스킬로 구조화된 요약 자동 생성
4. "핵심이라고 생각하는 점" 입력 요청
5. 브랜치 생성 → 요약 파일 커밋 → 푸시 → PR 생성

**생성되는 요약 구조:**

- 계층적 구조 (대주제 → 중주제 → 소주제)
- 예시, 데이터, 비유 등 디테일 보존
- 인과관계 중심 서술 (단순 나열 금지)
- 용어 정의 (Glossary) 테이블
- 본인이 핵심이라고 생각하는 점

### `/study:issue` — Issue 생성

스터디 관련 질문, 토론 주제, 공지를 구조화된 GitHub Issue로 생성합니다.

```
/study:issue ch03
```

**Issue 유형:**

| 유형 | 설명 | 라벨 |
|------|------|------|
| 질문 (Question) | 챕터 내용에 대해 궁금한 점 | `질문`, `chNN` |
| 토론 (Discussion) | 함께 논의하고 싶은 주제 | `토론`, `chNN` |
| 공지 (Announcement) | 스터디 운영 관련 공지 | `공지` |

자연어로 내용을 설명하면 Claude가 유형에 맞는 구조화된 본문을 생성합니다.

---

## 워크플로우 예시

### 일반적인 스터디 사이클

```
1. 스터디장이 /study:init 으로 저장소 초기화
2. 각 멤버가 담당 챕터를 읽고:
   - /study:pr ch01 로 요약 PR 생성
   - /study:issue ch01 로 질문/토론 Issue 생성
3. 팀원들이 PR을 리뷰하고 Issue에서 토론
4. 다음 챕터로 반복
```

### 요약 PR 작성 예시

```
> /study:pr ch03

? 챕터 원문을 어떻게 입력하시겠습니까?
  → 클립보드 붙여넣기

(챕터 내용 붙여넣기)

? 이 챕터에서 가장 핵심이라고 생각하는 점을 입력해주세요.
  → B-Tree의 splitting 전략이 쓰기 성능에 직접적 영향을 미친다는 점

✓ PR created: https://github.com/my-org/my-study/pull/5
```

### Issue 생성 예시

```
> /study:issue ch03

? 어떤 유형의 Issue를 생성할까요?
  → 질문 (Question)

"B-Tree에서 leaf node splitting과 non-leaf node splitting의
 구체적인 차이점이 궁금합니다. 책에서는 간략하게만 언급하는데..."

✓ Issue created: https://github.com/my-org/my-study/issues/12
```

---

## 프로젝트 구조

```
study-group-plugin/
├── .claude-plugin/
│   ├── plugin.json            # 플러그인 매니페스트
│   └── marketplace.json       # 마켓플레이스 매니페스트
├── commands/
│   ├── init.md                # /study:init 커맨드
│   ├── pr.md                  # /study:pr 커맨드
│   └── issue.md               # /study:issue 커맨드
├── skills/
│   └── technical-writer/
│       ├── SKILL.md           # 기술 저술 스킬 정의
│       └── references/
│           └── summary-template.md  # 요약 마크다운 템플릿
├── templates/
│   ├── README.template.md     # 저장소 README 템플릿
│   ├── issue-question.yml     # 질문 Issue 템플릿
│   ├── issue-discussion.yml   # 토론 Issue 템플릿
│   ├── issue-announcement.yml # 공지 Issue 템플릿
│   └── pull-request.md        # PR 본문 템플릿
└── README.md
```

---

## FAQ

### Q: 플러그인 업데이트는 어떻게 하나요?

```
/plugin install study-group@study-group-plugin
```

같은 명령어를 다시 실행하면 최신 버전으로 업데이트됩니다.

### Q: `/study:init` 없이 `/study:pr`을 실행할 수 있나요?

아니요. `/study:pr`과 `/study:issue`는 `/study:init`이 생성한 설정 파일(`.claude/study-group.local.md`)을 필요로 합니다. 반드시 `/study:init`을 먼저 실행하세요.

### Q: 이미 존재하는 저장소에서 사용할 수 있나요?

네. 기존 저장소 디렉토리에서 `/study:init`을 실행하면 됩니다. 기존 파일은 건드리지 않고 스터디에 필요한 파일만 추가합니다.

### Q: 요약을 직접 작성하고 싶은데 가능한가요?

네. `chNN/{username}-summary.md` 파일을 직접 작성하고 일반적인 Git 워크플로우로 PR을 만들어도 됩니다. `/study:pr`은 AI 자동 요약이 필요할 때 사용하는 편의 기능입니다.

---

## License

MIT

# study-group

GitHub 기반 책 스터디 관리 플러그인 for Claude Code.

저장소 초기화, PR 기반 요약 작성, Issue 기반 Q&A를 터미널에서 원스톱으로 수행할 수 있습니다.

## 설치

### Marketplace (권장)

Claude Code 에서 아래 명령어를 실행하세요:

```
/plugin marketplace add AgenticVibeLabs/study-group-plugin
/plugin install study-group@study-group-plugin
```

### 로컬 개발

```bash
claude --plugin-dir ./study-group
```

## 전제 조건

- [Claude Code](https://claude.com/claude-code) 설치
- [GitHub CLI (`gh`)](https://cli.github.com/) 설치 및 인증 완료
- Git 설정 완료

## 커맨드

### `/study:init`

스터디 저장소를 대화형으로 초기화합니다.

- 책 제목, 챕터 범위, 멤버 목록, 일정, 규칙을 대화형으로 입력
- README, 챕터 디렉토리, Issue/PR 템플릿 자동 생성
- GitHub 라벨(챕터별, 유형별) 자동 생성
- 설정을 `.claude/study-group.local.md`에 저장

```
/study:init
```

### `/study:pr`

챕터 요약 PR을 생성합니다.

- 텍스트 붙여넣기 또는 파일 지정으로 원문 입력
- Technical Writer 스킬로 체계적 요약 자동 생성
- 브랜치 생성 → 커밋 → 푸시 → PR 생성까지 원스톱

```
/study:pr ch01
```

### `/study:issue`

스터디 Issue(질문/토론/공지)를 생성합니다.

- 질문, 토론, 공지 유형 선택
- 자연어 입력을 구조화된 Issue 본문으로 변환
- 적절한 라벨 자동 부여

```
/study:issue ch03
```

## 스킬

### Technical Writer

챕터 요약 작성 시 자동으로 활성화되는 기술 저술 스킬입니다.

- 계층적 구조 (대주제 → 중주제 → 소주제)
- 예시, 데이터, 비유 등 디테일 보존
- 용어 정의 섹션 필수 포함
- 인과관계 중심 서술
- GitHub 마크다운 형식

## 프로젝트 구조

```
study-group/
├── .claude-plugin/
│   ├── plugin.json          # 플러그인 매니페스트
│   └── marketplace.json     # 마켓플레이스 매니페스트
├── commands/
│   ├── init.md              # /study:init
│   ├── pr.md                # /study:pr
│   └── issue.md             # /study:issue
├── skills/
│   └── technical-writer/
│       ├── SKILL.md          # 기술 저술 스킬
│       └── references/
│           └── summary-template.md
├── templates/
│   ├── README.template.md    # 저장소 README 템플릿
│   ├── issue-question.yml    # 질문 Issue 템플릿
│   ├── issue-discussion.yml  # 토론 Issue 템플릿
│   ├── issue-announcement.yml # 공지 Issue 템플릿
│   └── pull-request.md       # PR 본문 템플릿
└── README.md
```

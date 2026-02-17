# {{book_title}}

> {{members}} 의 책 스터디 저장소

## Members

| GitHub | 역할 |
|--------|------|
{{member_rows}}

## Schedule

- **주기**: {{schedule}}
- **시작일**: {{start_date}}

## Rules

{{rules}}

## Progress

| Chapter | 제목 | 상태 |
|---------|------|------|
{{chapter_rows}}

## How to Contribute

### 챕터 요약 작성

1. Claude Code에서 `/study:pr chNN` 실행
2. 원문 텍스트를 붙여넣기하거나 파일 경로 지정
3. AI가 체계적 요약을 생성하고 PR을 자동 생성

### 질문 & 토론

1. Claude Code에서 `/study:issue chNN` 실행
2. 질문/토론/공지 유형 선택
3. 자연어로 내용 작성하면 구조화된 Issue 자동 생성

### 직접 작성

1. `chNN/` 디렉토리에 `{username}-summary.md` 파일 작성
2. PR 템플릿에 따라 Pull Request 생성
3. 팀원 리뷰 후 머지

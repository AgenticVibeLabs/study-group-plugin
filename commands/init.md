---
description: 스터디 저장소를 대화형으로 초기화합니다
allowed-tools: AskUserQuestion, Read, Write, Edit, Bash(gh:*), Bash(git:*), Bash(mkdir:*), Glob
---

# Study Group Repository Initialization

You are initializing a book study group repository. Guide the user through an interactive setup process.

## Step 1: Gather Study Information

Use AskUserQuestion to collect the following information. Ask questions in batches for efficiency.

**Batch 1 (ask together):**

**Question 1 — Book title:**
- header: "Book"
- question: "스터디할 책 제목을 입력해주세요. (Other를 선택하여 직접 입력)"
- options:
  - Database Internals (Alex Petrov)
  - Designing Data-Intensive Applications (Martin Kleppmann)
  - System Design Interview (Alex Xu)

**Question 2 — Chapter range:**
- header: "Chapters"
- question: "책의 챕터 수(범위)를 선택해주세요."
- options:
  - 1-10 (10 chapters)
  - 1-13 (13 chapters)
  - 1-15 (15 chapters)
  - 1-20 (20 chapters)

**Batch 2 (ask together after Batch 1):**

**Question 3 — Member GitHub usernames:**
- header: "Members"
- question: "스터디 멤버의 GitHub username을 입력해주세요. (Other를 선택하여 콤마로 구분하여 입력, 예: alice,bob,charlie)"
- options:
  - roach (본인만)
  - roach,member2 (2명 - Other로 수정)

**Question 4 — Schedule:**
- header: "Schedule"
- question: "스터디 일정을 선택해주세요."
- options:
  - 매주 월요일 (Weekly Monday)
  - 매주 수요일 (Weekly Wednesday)
  - 격주 토요일 (Biweekly Saturday)

**After Batch 2:**

**Question 5 — Rules (ask separately):**
- header: "Rules"
- question: "스터디 규칙을 입력해주세요. (Other를 선택하여 직접 입력)"
- options:
  - 기본 규칙 (매주 1챕터 요약 PR + 질문 1개 이상) (Recommended)
  - 유연한 규칙 (격주 요약, 질문 선택)

If user selects "기본 규칙", use:
```
1. 매주 담당 챕터의 요약 PR을 작성합니다.
2. 챕터당 최소 1개의 질문 Issue를 생성합니다.
3. 다른 멤버의 PR에 최소 1개의 리뷰 코멘트를 남깁니다.
4. 불참 시 최소 하루 전에 공지합니다.
```

## Step 2: Repository Scaffolding

Parse the collected information:
- Extract book title from Q1 answer
- Parse chapter range from Q2 (e.g., "1-13" → chapters 1 through 13)
- Parse member list from Q3 (comma-separated)
- Extract schedule from Q4
- Extract rules from Q5

### 2.1 Generate README.md

Read the template at `@${CLAUDE_PLUGIN_ROOT}/templates/README.template.md`.

Replace the placeholders:
- `{{book_title}}` → book title from Q1
- `{{members}}` → comma-separated member names
- `{{member_rows}}` → generate one row per member: `| @username | Member |`
- `{{schedule}}` → schedule from Q4
- `{{start_date}}` → today's date
- `{{rules}}` → rules text from Q5
- `{{chapter_rows}}` → generate one row per chapter: `| chNN | - | Pending |`

Write the generated README.md to the repository root.

### 2.2 Create Chapter Directories

For each chapter in the range, create a directory with a `.gitkeep` file:

```
mkdir -p ch01 ch02 ... chNN
touch ch01/.gitkeep ch02/.gitkeep ... chNN/.gitkeep
```

Use zero-padded two-digit format (ch01, ch02, ..., ch13).

### 2.3 Copy Issue Templates

Create `.github/ISSUE_TEMPLATE/` directory and copy the three issue templates:

Read each template from:
- `@${CLAUDE_PLUGIN_ROOT}/templates/issue-question.yml`
- `@${CLAUDE_PLUGIN_ROOT}/templates/issue-discussion.yml`
- `@${CLAUDE_PLUGIN_ROOT}/templates/issue-announcement.yml`

Write them to:
- `.github/ISSUE_TEMPLATE/question.yml`
- `.github/ISSUE_TEMPLATE/discussion.yml`
- `.github/ISSUE_TEMPLATE/announcement.yml`

### 2.4 Copy PR Template

Read `@${CLAUDE_PLUGIN_ROOT}/templates/pull-request.md` and write it to `.github/PULL_REQUEST_TEMPLATE.md`.

## Step 3: Create GitHub Labels

Use `gh` CLI to create labels. Run these commands:

```bash
gh label create "질문" --color "d876e3" --description "챕터 내용에 대한 질문" --force
gh label create "토론" --color "0075ca" --description "함께 논의할 주제" --force
gh label create "공지" --color "e4e669" --description "스터디 공지사항" --force
```

For each chapter in the range:
```bash
gh label create "ch01" --color "c5def5" --description "Chapter 01" --force
gh label create "ch02" --color "c5def5" --description "Chapter 02" --force
...
```

Use `--force` to avoid errors if labels already exist.

If `gh` commands fail (e.g., no remote repository), warn the user but do not abort. Labels can be created later after the repository is pushed to GitHub.

## Step 4: Save Configuration

Create `.claude/study-group.local.md` with the study configuration:

```markdown
---
book_title: "{{book_title}}"
chapter_start: 1
chapter_end: {{last_chapter}}
members:
  - "{{member1}}"
  - "{{member2}}"
schedule: "{{schedule}}"
initialized_at: "{{current_date}}"
---

# Study Group Configuration

This file stores the local configuration for the study-group plugin.
Do not commit this file to the repository.
```

Also ensure `.gitignore` contains `.claude/*.local.md`. If `.gitignore` exists, append the pattern if not already present. If it does not exist, create it with this pattern.

## Step 5: Git Commit & Push

Stage and commit all generated files:

```bash
git add -A
git commit -m "docs: initialize study group repository for {{book_title}}"
```

Ask the user before pushing:

Use AskUserQuestion:
- header: "Push"
- question: "원격 저장소에 푸시할까요?"
- options:
  - Yes (git push origin main)
  - No (로컬에만 커밋)

If "Yes", run `git push origin main`.

## Step 6: Summary

Print a summary of what was created:

```
Study group initialized!

Book: {{book_title}}
Members: {{members}}
Schedule: {{schedule}}
Chapters: ch01 ~ ch{{last_chapter}}

Created:
  - README.md
  - {{N}} chapter directories (ch01 ~ ch{{last_chapter}})
  - .github/ISSUE_TEMPLATE/ (3 templates)
  - .github/PULL_REQUEST_TEMPLATE.md
  - .claude/study-group.local.md
  - GitHub labels (3 types + {{N}} chapters)

Next steps:
  - /study:pr ch01  → 첫 번째 챕터 요약 PR 작성
  - /study:issue ch01 → 질문이나 토론 Issue 생성
```

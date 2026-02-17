---
description: 챕터 요약 PR을 생성합니다
argument-hint: [chapter] (예: ch01)
allowed-tools: Read, Write, Edit, Bash(git:*), Bash(gh:*), AskUserQuestion, Glob, Grep
---

# Chapter Summary PR Creation

You are creating a structured chapter summary Pull Request for a book study group.

## Step 1: Load Configuration & Validate

### 1.1 Read Configuration

Read `.claude/study-group.local.md` in the current working directory.

If the file does not exist, inform the user:
```
설정 파일을 찾을 수 없습니다.
먼저 /study:init 을 실행하여 스터디 저장소를 초기화해주세요.
```
Then stop.

Parse the YAML frontmatter to extract: `book_title`, `chapter_start`, `chapter_end`, `members`.

### 1.2 Parse Chapter Argument

Extract the chapter identifier from `$1` (e.g., "ch01", "ch03").

If `$1` is not provided, use AskUserQuestion:
- header: "Chapter"
- question: "어떤 챕터의 요약을 작성할까요?"
- Generate options from `chapter_start` to `chapter_end` (show first 4 as options, user can enter Other)

Validate the chapter number is within the configured range.

### 1.3 Identify Current User

Run: `gh api user --jq .login` to get the current GitHub username.

If this fails, use AskUserQuestion:
- header: "Username"
- question: "GitHub username을 입력해주세요. (Other를 선택하여 입력)"
- options from the members list in configuration

Store as `username`.

## Step 2: Input Source Material

Use AskUserQuestion to determine input mode:
- header: "Input"
- question: "챕터 원문을 어떻게 입력하시겠습니까?"
- options:
  - 클립보드 붙여넣기 (다음 메시지에 텍스트를 붙여넣어주세요)
  - 파일 지정 (파일 경로를 입력해주세요)

**If "클립보드 붙여넣기":**
Tell the user: "챕터 원문 텍스트를 다음 메시지에 붙여넣어주세요."
Wait for the user's next message containing the source text.

**If "파일 지정":**
Use AskUserQuestion:
- header: "File path"
- question: "원문 파일의 경로를 입력해주세요. (Other를 선택하여 입력)"
- options:
  - ${chapter}/${chapter}-source.md
  - ${chapter}/source.txt

Then read the file using the Read tool.

## Step 3: Generate Summary

Apply the **technical-writer** skill to create a comprehensive chapter summary.

Use the template at `@${CLAUDE_PLUGIN_ROOT}/skills/technical-writer/references/summary-template.md` as the structural guide.

Fill in the template metadata:
- `{{chapter_id}}` → the chapter identifier (e.g., ch03)
- `{{chapter_title}}` → extract from the source material if possible, otherwise ask the user
- `{{book_title}}` → from configuration
- `{{username}}` → from Step 1.3
- `{{date}}` → today's date

Generate the full summary following all Technical Writer skill principles:
1. Hierarchical structure (## → ### → ####)
2. Preserve ALL examples, data, analogies, and case studies
3. Causal reasoning (not enumeration)
4. Complete glossary
5. Leave "What I Think is Key" section for the user

### 3.1 User's Key Takeaway

After showing the generated summary, use AskUserQuestion:
- header: "Key point"
- question: "이 챕터에서 가장 핵심이라고 생각하는 점을 입력해주세요. (Other를 선택하여 자유롭게 작성)"
- options:
  - 나중에 작성 (PR 생성 후 직접 편집)

If the user provides input, fill in the "What I Think is Key" section.
If "나중에 작성", leave the section with a placeholder comment.

## Step 4: Create Branch, Commit, Push, and PR

### 4.1 Create Branch

```bash
git checkout -b study/${chapter}/${username}
```

If the branch already exists, inform the user and ask whether to continue on the existing branch or create a new one with a suffix.

### 4.2 Write Summary File

Write the completed summary to:
```
${chapter}/${username}-summary.md
```

### 4.3 Commit

```bash
git add ${chapter}/${username}-summary.md
git commit -m "docs(${chapter}): add ${username}'s chapter summary"
```

### 4.4 Push

```bash
git push -u origin study/${chapter}/${username}
```

### 4.5 Create PR

Read the PR template at `@${CLAUDE_PLUGIN_ROOT}/templates/pull-request.md` for reference on the PR body structure.

Create the PR:

```bash
gh pr create \
  --title "[${chapter}] ${book_title} - ${username}" \
  --body "## Summary

${summary_overview}

## Key Topics

${key_topics_list}

## Discussion Points

${discussion_points}

## Glossary

${glossary_table}

## What I Think is Key

${key_takeaway}

## Self Review Checklist

- [ ] 요약이 원문의 핵심 내용을 빠짐없이 포함하는가?
- [ ] 예시와 비유가 적절히 보존되었는가?
- [ ] 용어 정의가 정확한가?
- [ ] 인과관계가 명확히 서술되었는가?
- [ ] 마크다운 형식이 올바른가?" \
  --label "${chapter}"
```

Use a HEREDOC for the body to ensure correct formatting.

## Step 5: Return Result

Print the PR URL and a summary:

```
PR created successfully!

Branch: study/${chapter}/${username}
File: ${chapter}/${username}-summary.md
PR: ${pr_url}

Next steps:
  - PR 링크를 팀원들에게 공유하세요
  - 리뷰 코멘트를 기다리세요
  - /study:issue ${chapter} 로 질문이나 토론 Issue를 생성할 수 있습니다
```

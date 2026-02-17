---
description: 스터디 Issue(질문/토론/공지)를 생성합니다
argument-hint: [chapter] (예: ch01)
allowed-tools: Read, Bash(gh:*), AskUserQuestion, Glob
---

# Study Group Issue Creation

You are creating a structured GitHub Issue for a book study group.

## Step 1: Load Configuration

Read `.claude/study-group.local.md` in the current working directory.

If the file does not exist, inform the user:
```
설정 파일을 찾을 수 없습니다.
먼저 /study:init 을 실행하여 스터디 저장소를 초기화해주세요.
```
Then stop.

Parse the YAML frontmatter to extract: `book_title`, `chapter_start`, `chapter_end`.

## Step 2: Select Issue Type

Use AskUserQuestion:
- header: "Type"
- question: "어떤 유형의 Issue를 생성할까요?"
- options:
  - 질문 (Question) — 챕터 내용에 대해 궁금한 점
  - 토론 (Discussion) — 함께 논의하고 싶은 주제
  - 공지 (Announcement) — 스터디 운영 관련 공지

## Step 3: Select Chapter

If `$1` is provided, use it as the chapter identifier.

If `$1` is not provided AND the type is NOT "공지":

Use AskUserQuestion:
- header: "Chapter"
- question: "관련 챕터를 선택해주세요."
- Generate options from `chapter_start` to `chapter_end` (show first 4, user can enter Other)

For "공지" type, chapter is optional — skip if not provided.

## Step 4: Collect Content

Tell the user to describe their question, topic, or announcement in natural language:

"자연어로 내용을 설명해주세요. Claude가 구조화된 Issue 본문을 생성합니다."

Wait for the user's next message.

## Step 5: Structure the Issue

Based on the issue type, structure the user's input into a well-formatted Issue body.

### For "질문 (Question)":

Generate a title: `[${chapter}] ${concise_question_title}`

Structure the body:

```markdown
## Context

<!-- 질문의 배경 맥락 -->
${context}

## Related Quote

> ${relevant_quote_from_user_input_if_any}

## Question

${structured_question}

## Current Understanding

${what_user_seems_to_understand}

## What I Want to Know

${what_user_wants_clarified}

---
📖 **${book_title}** — ${chapter}
```

### For "토론 (Discussion)":

Generate a title: `[${chapter}] ${concise_topic_title}`

Structure the body:

```markdown
## Topic

${clearly_stated_topic}

## Background

${why_this_topic_matters}

## Perspectives to Consider

${different_angles_to_think_about}

## My Opinion

${user_viewpoint}

## Discussion Prompt

${question_to_spark_discussion}

---
📖 **${book_title}** — ${chapter}
```

### For "공지 (Announcement)":

Generate a title: `[공지] ${announcement_title}`

Structure the body:

```markdown
## Announcement

${announcement_content}

## Action Items

- [ ] ${action_item_1}
- [ ] ${action_item_2}

## Timeline

${deadline_or_schedule_if_any}

---
📢 **${book_title}** Study Group
```

## Step 6: Preview and Confirm

Show the generated title and body to the user.

Use AskUserQuestion:
- header: "Confirm"
- question: "이 내용으로 Issue를 생성할까요?"
- options:
  - Yes (Issue 생성)
  - 수정 필요 (내용을 수정하겠습니다)

If "수정 필요", ask the user what to change, apply the changes, and confirm again.

## Step 7: Create Issue

Determine the labels based on type and chapter:
- 질문: `"질문,${chapter}"`
- 토론: `"토론,${chapter}"`
- 공지: `"공지"` (no chapter label unless specified)

Create the issue:

```bash
gh issue create \
  --title "${title}" \
  --body "${body}" \
  --label "${labels}"
```

Use a HEREDOC for the body to ensure correct formatting.

## Step 8: Return Result

Print the Issue URL and a summary:

```
Issue created successfully!

Type: ${type}
Title: ${title}
Labels: ${labels}
URL: ${issue_url}
```

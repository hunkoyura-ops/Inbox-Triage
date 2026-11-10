# Inbox → Triage

An automation that sorts incoming replies from content creators. Built in n8n.

## The problem

After creator outreach, dozens of replies land in the inbox. Some creators are interested, some negotiate the price, some decline, some write off-topic. Sorting them by hand means half an hour every day spent reading, sorting and copying into a spreadsheet. Meanwhile the hot replies get lost among the rest.

## What the flow does

```
Gmail (new email with the "creators" label)
   ↓
Claude API: intent classification
   ↓
Switch: branch by category
   ↓
Claude API: draft reply generation
   ↓
Google Sheets: log
   ↓
Gmail: draft saved to Drafts
```

The model determines the category (`interested`, `negotiating`, `declined`, `question`, `spam`), the priority and the creator's name, and extracts the specifics: quoted prices, deadlines, questions.

The flow then branches. Substantive replies get a draft answer. Declines and spam are only logged, with no extra API call.

## Technical decisions

**A draft, not an auto-send.** The flow prepares a reply and puts it in Gmail Drafts. The decision to send stays with a person. Automating the last step is technically easy, but a mistake in a conversation with a creator costs more than the minute saved.

**Direct API calls instead of ready-made nodes.** Claude is called through an HTTP Request node, with full control over the prompt and parameters.

**Error handling, not just the happy path.** The model's response is parsed in a `try/catch` block with a fallback. If it returns something other than JSON, the flow does not crash; it writes the problem to the log. The Switch has a fallback output for an unexpected category.

**Hallucination guard in the prompt.** The model is explicitly forbidden to invent budgets, deadlines or terms that are not in the email. Whatever is missing for a reply is marked as needing clarification instead of being made up.

## Stack

- n8n (Cloud)
- Anthropic Claude API
- Gmail API
- Google Sheets API

## Sheet structure

| Date | From | Creator | Category | Priority | Summary | Draft | Status |
|------|------|---------|----------|----------|---------|-------|--------|

## Setup

1. Import `inbox-triage.json` into n8n
2. Connect Gmail and Google Sheets credentials through OAuth
3. Create a Header Auth credential for the Anthropic API (`x-api-key`)
4. Create a `creators` label in Gmail and a filter that assigns it
5. Create a sheet with the columns from the structure above

The API key in the file is replaced with a placeholder. Put in your own.

## Author

Yurii Hunko, [hunkoyura.com](https://hunkoyura.com)

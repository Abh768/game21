Interview Emails Agent: Playwright MCP + Sendmux MCP
Purpose
These instructions tell an AI agent how to automate the Interview Emails workflow. The agent uses two MCP servers:
Playwright MCP opens and uses a web browser.
Sendmux MCP ("Email Inbox API + Sending by Sendmux") reads, labels, and drafts email.
The workflow is based on this Make.com scenario:
https://us2.make.com/public/shared-scenario/ZQg2VLV7Zqy/interview-emails
1. Set up the MCP servers in VS Code
Create `.vscode/mcp.json` in the project folder:
```json
{
"servers": {
"playwright": {
"command": "npx",
"args": ["@playwright/mcp@latest"]
},
"sendmux": {
"type": "http",
"url": "https://mcp.sendmux.ai/mcp"
}
}
}
```
Playwright requires Node.js 18 or newer.
On first use, Sendmux opens a browser sign-in. Grant Mailbox and Sending access only.
2. Rules for the agent
Use Sendmux for email and Playwright for browser work.
Never send an email automatically. Create drafts only so I can review them.
Never type passwords. If a website asks me to sign in, stop and let me sign in myself.
Before clicking or typing in Playwright, call `browser_snapshot` to read the page.
If a step fails, stop and tell me what failed. Do not guess.
3. Use Playwright to view the Make.com scenario
Call `browser_navigate` with the scenario URL above.
Call `browser_wait_for` until the scenario diagram appears.
Call `browser_snapshot` and list each module in order.
Call `browser_take_screenshot` and save it as `interview-emails-scenario.png`.
The scenario has these modules:
#
Module
What it does
1
Gmail: Watch emails
Checks for new emails.
5
Make AI Toolkit: Categorize text
Determines whether the email is an interview invitation.
2
If-else
Routes interview emails to the first path and all other emails to the Else path.
4
Gmail: Update email labels
Labels interview emails.
14
GitHub: Create an issue
Creates a tracking issue for the interview.
6
Make AI Toolkit: Categorize text
Determines whether a non-interview email is a rejection.
3
If-else
Routes rejections to the first path and all other emails to "No response."
8
Make AI Agent: Run an agent
Writes a thank-you reply for a rejection.
9
Gmail: Create a draft email
Saves the thank-you reply as a draft.
7
Gmail: Get an email
Retrieves emails that need no response.
4. Run the workflow
Step 1: Watch emails
Call `mailbox_list_granted_mailboxes` and use the returned `mailbox_id`.
Call `mailbox_list_messages` to get new, unread emails.
Call `mailbox_get_message` to read each email.
Step 2: Categorize the email
Assign each email to exactly one category:
Interview: The sender invites me to an interview or asks to schedule one.
Rejection: The sender says I was not selected or that the position was filled or cancelled.
No response: Any other email.
Step 3A: Interview path
Add the label `Interview` to the email in Sendmux.
Use Playwright to create a GitHub issue:
Call `browser_navigate` with `https://github.com/<owner>/<repo>/issues/new`.
If GitHub asks me to sign in, stop and wait.
Call `browser_type` to enter the title: `Interview: <Company> - <Job Title>`.
Call `browser_type` to enter the body: sender, date, interview time, and a short summary.
Call `browser_click` on Submit new issue.
Call `browser_snapshot` to confirm that the issue was created, then save its URL.
Step 3B: Rejection path
Write a short, polite thank-you reply. For example:
> Thank you for reviewing my resume and considering me for the position. I understand that it will not be filled at this time. I appreciate your time and hope to stay in touch for future opportunities.
Save the reply as a draft. Do not send it.
Step 3C: No response path
Retrieve the email with `mailbox_get_message`.
Take no further action.
5. Finish
Call `browser_close`.
Report a summary table:
Email subject
Category
Action taken
Link
...
Interview / Rejection / No response
Labeled + issue created / Draft created / Ignored
Issue URL or "-"
Playwright tools used
Tool
What it does
`browser_navigate`
Opens a URL.
`browser_snapshot`
Reads the page structure so the agent can find buttons and text boxes.
`browser_click`
Clicks a button or link.
`browser_type`
Types text into a field.
`browser_wait_for`
Waits for text or the page to load.
`browser_take_screenshot`
Saves a picture of the page.
`browser_close`
Closes the browser.
Sendmux tools used
Tool
What it does
`mailbox_list_granted_mailboxes`
Lists the mailboxes I granted access to.
`mailbox_list_messages`
Lists emails in the mailbox.
`mailbox_get_message`
Reads one email.
`mailbox_search_message_snippets`
Searches emails by keyword.


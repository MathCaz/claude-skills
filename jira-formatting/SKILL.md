---
name: jira-formatting
description: Formatting rules for Jira ticket descriptions and comments written via the Atlassian MCP tools. Apply whenever creating or updating a Jira issue or comment with contentFormat "markdown".
---

# Jira Formatting Rules

When creating or updating Jira ticket descriptions or comments using the Atlassian MCP tools with `contentFormat: "markdown"`:

1. **Real newlines only**: Never use escaped newlines (`\n`) in the description string value. Use actual newlines so Jira renders proper line breaks, bullet points, and formatting.
2. **Markdown syntax**: Use standard Markdown syntax for formatting (e.g., `-` for bullet points, `**bold**` for emphasis, etc.) to ensure Jira renders the description correctly.
3. **Clear structure**: Organize the description with clear sections and bullet points to enhance readability and ensure important information is easily accessible to the team.

# Claude Code — Project Maintenance Prompt

Paste this into Claude Code when starting a session:

---

```
Before making any changes, read PROJECT_STATE.md and CHANGELOG.md to understand the current state of this project, open features, bugs, and priorities. After completing any work:

1. Update PROJECT_STATE.md if you changed architecture, modules, data model, or key decisions
2. Update CHANGELOG.md:
   - Log any bugs found or fixed (use next BUG-XXX number)
   - Log any features added or modified (use next FEAT-XXX number)
   - Move completed items from Open to Closed/Done with date
   - If releasing a new version, add a new changelog entry under the Changelog section
3. Commit all changes including the updated docs with a clear commit message
4. Push to origin/main so Vercel auto-deploys

Key context:
- This is a single-file static app (index.html) deployed to Vercel
- React 18 + Tailwind from CDN, no build step
- Data persists in browser localStorage
- AI features use the Anthropic API client-side (claude-sonnet-4-20250514, fetch to api.anthropic.com/v1/messages)
- Documents (spec + tenders) are uploaded as base64 and sent to the API
- GBP currency, UK-based user

Always keep PROJECT_STATE.md and CHANGELOG.md accurate — they are the single source of truth referenced by the Claude.ai project for this renovation tracker.
```

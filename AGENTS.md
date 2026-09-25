## Agent skills

### Issue tracker

Issues live in GitHub Issues on `coldboot-labs/fwos` (`gh` CLI). See `docs/agents/issue-tracker.md`.

### Triage labels

Canonical roles map 1:1 to `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: root `CONTEXT.md` and `docs/adr/`. See `docs/agents/domain.md`.

### Testing

For each issue, run the tests relevant to the changed behavior and its nearby regression paths. Select and report the specific cases, including real-appliance tests when applicable. Run the full test suite only when the user explicitly asks for it.

---
name: Never tell user to delete .gir/ folder
description: Hard rule — never instruct the user to delete or remove .gir/ during migration or any other operation
type: feedback
---

Never tell the user to delete, remove, or rm the `.gir/` folder under any circumstances, including during migrations, upgrades, or troubleshooting.

**Why:** `.gir/` is the project memory bank — it contains MISSION.md, CLAUDE-activeContext.md, ESCALATION.md, GIR.modules, and user-owned customization files. Deleting it destroys project state and history.

**How to apply:** If a migration or fix seems to require clearing `.gir/`, find another way. At most, instruct the user to edit or reset specific files within `.gir/`, never the folder itself.

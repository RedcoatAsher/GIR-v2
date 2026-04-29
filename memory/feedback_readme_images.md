---
name: README images must never be removed
description: Three Invader Zim images are intentional and must be preserved in every README update
type: feedback
---

Never remove the GIR character images from README.md. They are intentional branding — part of the Invader Zim theme identity.

**Images (all three must stay):**
1. GIR animated GIF — `https://i.pinimg.com/originals/57/6e/0e/576e0e99fd23505db71aa9caaa670fe3.gif` — floats right in "What Is GIR?" section
2. GIR dog PNG — `https://static.wikia.nocookie.net/zimwiki/images/d/d2/Girdog.png/revision/latest/scale-to-width-down/300` — floats right before Modules table
3. Invader Zim banner — `https://timelinecovers.pro/facebook-cover/download/tv-show-invader-zim-facebook-cover.jpg` — full-width (`width="100%"`) before FAQ section

**Why:** Images were stripped during multiple doc-cleanup commits. User explicitly asked for them back and to never remove them again.

**How to apply:** When editing README.md for any reason (docs, refactoring, version bumps), verify all three `<img>` tags are still present before committing.

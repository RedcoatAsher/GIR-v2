---
name: design-principles
description: Enforce a precise, minimal design system inspired by Linear, Notion, and Stripe. Use this skill when building dashboards, admin interfaces, or any UI that needs Jony Ive-level precision - clean, modern, minimalist with taste. Every pixel matters.
---

# Design Principles

Precise, crafted design for enterprise software, SaaS dashboards, admin interfaces. Jony Ive-level precision with intentional personality.

## Design Direction (REQUIRED)

**Before code, commit to a direction.** Don't default.

### Context
- **Product**: Finance needs different energy than creative tools
- **Users**: Power users want density, occasional users want guidance
- **Emotion**: Trust? Efficiency? Delight? Focus?
- **Memorable**: What makes it distinctive?

### Personalities

**Precision & Density** — Tight, monochrome, info-forward (Linear, Raycast, terminal)

**Warmth & Approachability** — Generous spacing, soft shadows, friendly (Notion, Coda)

**Sophistication & Trust** — Cool tones, layered depth, gravitas (Stripe, Mercury)

**Boldness & Clarity** — High contrast, dramatic negative space (Vercel)

**Utility & Function** — Muted, functional density (GitHub)

**Data & Analysis** — Chart-optimized, numbers first (analytics, BI)

Pick one or blend two.

### Color Foundation
- **Warm** (creams, warm grays) — human, comfortable
- **Cool** (slate, blue-gray) — professional, serious
- **Pure** (true grays, black/white) — minimal, technical
- **Tinted** (slight cast) — distinctive, branded

**Light**: Open, clean. **Dark**: Technical, focused, premium.

**Accent**: ONE meaningful color. Blue=trust, Green=growth, Orange=energy.

### Layout
- Dense grids for info-heavy scanning
- Generous spacing for focused tasks
- Sidebar for multi-section, top nav for simple
- Split panels for list-detail

## Core Craft

### 4px Grid
4px micro | 8px tight | 12px standard | 16px comfortable | 24px generous | 32px major

### Symmetrical Padding
TLBR must match. Exception: content creates visual balance.

### Border Radius
Stick to 4px grid. Sharp=technical, round=friendly. Pick system, commit.
- Sharp: 4px, 6px, 8px
- Soft: 8px, 12px

### Depth Strategy
**Borders-only**: Clean, technical (Linear, Raycast)
**Subtle shadow**: Soft lift (`0 1px 3px rgba(0,0,0,0.08)`)
**Layered shadows**: Rich, premium (Stripe, Mercury)
**Surface shifts**: Background tints for hierarchy

Choose ONE, commit.

### Card Layouts
Vary internal structure for content. Keep surface treatment consistent (border, shadow, radius, padding, typography).

### Isolated Controls
UI controls deserve container treatment. Never native form elements — build custom.

Custom select: `display: inline-flex` + `white-space: nowrap`

### Typography
- Headlines: 600, tight tracking (-0.02em)
- Body: 400-500
- Labels: 500, slight positive for uppercase
- Scale: 11, 12, 13, 14 (base), 16, 18, 24, 32px

### Data
Monospace for numbers/IDs/codes/timestamps. `tabular-nums` for columns.

### Icons
Phosphor Icons. Only if removal loses meaning. Give standalone icons background containers.

### Animation
150ms micro | 200-250ms transitions | `cubic-bezier(0.25, 1, 0.5, 1)` | No bounce/spring

### Hierarchy
Four levels: foreground → secondary → muted → faint

### Color for Meaning
Gray builds structure. Color only for status, action, error, success.

## Navigation
Screens need grounding — nav, location indicator, user context.

Consider same background for sidebar + content (border for separation).

## Dark Mode
- Borders over shadows
- Adjust semantic colors (desaturate)
- Same hierarchy, inverted values

## Never
- Dramatic shadows
- Large radius (16px+) on small elements
- Asymmetric padding without reason
- Pure white cards on colored backgrounds
- Thick borders (2px+) decorative
- Excessive margins (>48px)
- Spring/bouncy animations
- Decorative gradients
- Multiple accent colors

## Always Question
- Did I think or default?
- Does direction fit context?
- Does element feel crafted?
- Is depth strategy consistent?
- All elements on grid?

## Standard

Every interface: obsessed over 1px differences. Not stripped — *crafted*. Context-driven.

Different products want different things. Let context guide aesthetic.

**Goal**: Intricate minimalism with appropriate personality.

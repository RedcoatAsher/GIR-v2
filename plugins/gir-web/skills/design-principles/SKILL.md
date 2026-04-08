---
name: design-principles
description: "Enforce a precise, minimal design system for enterprise software, SaaS dashboards, and admin interfaces. Covers design direction selection (6 personality types from Linear-style density to Stripe-style sophistication), 4px grid system, color foundations, depth strategies, typography scale, animation timing, dark mode rules, and a checklist of anti-patterns to avoid. Use when building dashboards, admin interfaces, data-heavy UIs, or any frontend that requires consistent spacing, typography, and visual hierarchy."
---

# Design Principles

Precise, crafted design for enterprise software, SaaS dashboards, and admin interfaces.

## Workflow

1. **Choose direction** — Pick a design personality (or blend two) and color foundation before writing any code
2. **Set foundations** — Commit to grid scale, depth strategy, border radius system, and typography scale
3. **Build components** — Apply consistent surface treatment (border, shadow, radius, padding) across all cards and controls
4. **Validate** — Run through the "Always Question" checklist on every screen before shipping

## Design Direction

**Before code, commit to a direction.** Evaluate context: product type, user expertise, target emotion.

### Personalities

| Personality | Traits | Reference |
|-------------|--------|-----------|
| Precision & Density | Tight, monochrome, info-forward | Linear, Raycast |
| Warmth & Approachability | Generous spacing, soft shadows | Notion, Coda |
| Sophistication & Trust | Cool tones, layered depth | Stripe, Mercury |
| Boldness & Clarity | High contrast, dramatic negative space | Vercel |
| Utility & Function | Muted, functional density | GitHub |
| Data & Analysis | Chart-optimized, numbers first | Analytics, BI |

### Color Foundation
- **Warm** (creams, warm grays) — human, comfortable
- **Cool** (slate, blue-gray) — professional, serious
- **Pure** (true grays, black/white) — minimal, technical
- **Tinted** (slight cast) — distinctive, branded

ONE accent color only. Blue=trust, Green=growth, Orange=energy.

### Layout Rules
- Dense grids for info-heavy scanning, generous spacing for focused tasks
- Sidebar for multi-section apps, top nav for simple flows
- Split panels for list-detail views

## Core Craft

### 4px Grid
4px micro | 8px tight | 12px standard | 16px comfortable | 24px generous | 32px major. Symmetrical TLBR padding always.

### Border Radius
Stick to 4px grid. Sharp (4-8px) = technical. Soft (8-12px) = friendly. Pick one system, commit.

### Depth Strategy (choose ONE)
- **Borders-only**: Clean, technical (Linear, Raycast)
- **Subtle shadow**: `0 1px 3px rgba(0,0,0,0.08)`
- **Layered shadows**: Rich, premium (Stripe, Mercury)
- **Surface shifts**: Background tints for hierarchy

### Typography
Headlines: 600 weight, -0.02em tracking. Body: 400-500. Labels: 500.
Scale: 11, 12, 13, 14 (base), 16, 18, 24, 32px. Monospace + `tabular-nums` for data columns.

### Animation
150ms micro | 200-250ms transitions | `cubic-bezier(0.25, 1, 0.5, 1)` | No bounce/spring.

### Hierarchy
Four levels: foreground → secondary → muted → faint. Gray builds structure; color only for status, action, error, success.

## Dark Mode
- Borders over shadows
- Desaturate semantic colors
- Same hierarchy, inverted values

## Anti-Patterns (Never)
- Dramatic shadows or decorative gradients
- Large radius (16px+) on small elements
- Asymmetric padding without reason
- Pure white cards on colored backgrounds
- Thick borders (2px+) decorative
- Excessive margins (>48px)
- Spring/bouncy animations
- Multiple accent colors

## Validation Checklist
- Did I choose a direction or just default?
- Does direction fit the product context?
- Does every element feel crafted?
- Is depth strategy consistent throughout?
- Are all elements aligned to the 4px grid?

# [Project Name] — Architecture

---

## Tech stack

[List the key technologies and why they were chosen]

## Key constraints

[What architectural decisions are fixed? What must always be true?]

## Key file map

[List the most important files/directories and what they contain]

---

## Design system

> Skip this section for pure backend or CLI projects. For any project with a UI, fill this in before writing any components — it is the reference Claude uses to maintain visual consistency across the entire app.

### Visual identity
- **Colors:** [e.g., Primary: #3B82F6 | Surface: #F9FAFB | Muted: #6B7280 | Destructive: #EF4444]
- **Typography:** [e.g., Headings: Inter 600 | Body: Inter 400 | Mono: JetBrains Mono 400]
- **Spacing scale:** [e.g., 4px base unit — 4 / 8 / 12 / 16 / 24 / 32 / 48 / 64px]
- **Border radius:** [e.g., sm: 4px | md: 8px | lg: 16px | full: 9999px]
- **Shadow:** [e.g., sm: 0 1px 2px rgba(0,0,0,0.05) | md: 0 4px 6px rgba(0,0,0,0.07)]

### Component patterns
- **Buttons:** [e.g., Primary = filled, secondary = outlined, destructive = red fill — always include loading and disabled states]
- **Forms:** [e.g., Label above input, helper text below, inline validation on blur, error text in destructive color]
- **Cards:** [e.g., white background, 1px border in muted color, md radius, 24px padding]
- **Empty states:** [e.g., centered icon + heading + body copy + optional CTA — never just blank space]
- **Modals:** [e.g., overlay with 200ms fade, card with lg radius, close button top-right, trap focus]

### Interaction patterns
- **Loading:** [e.g., Skeleton screens for data fetches >300ms; spinner for user-triggered actions]
- **Errors:** [e.g., Toast for transient/background errors; inline for form validation; error boundary for crashes]
- **Success feedback:** [e.g., Toast for confirmations; inline checkmark for small actions]
- **Transitions:** [e.g., 150ms ease-out for all UI state changes; no transition on initial page load]

### Anti-patterns — never do these
[What the product should never feel like or look like. Be specific.]

# Agent Instructions & Project Guardrails

Welcome! This document provides information on how to develop, test, and contribute to the **DevToolbox Platform**. It includes architectural rules, quality standards, and strict guardrails that all AI agents and human contributors must follow.

---

## 1. Project Overview & Tech Stack

DevToolbox is a suite of professional-grade developer utilities running entirely in the browser with **zero backend dependencies**, focusing on speed, accessibility, and privacy.

- **Frontend**: Vanilla HTML5, Vanilla CSS3 (with custom properties), and Vanilla JavaScript (ES6+).
- **Frameworks**: None (No React, Vue, Angular, or TailwindCSS unless specifically requested).
- **Libraries**: Stored locally in `lib/` (e.g., `Chart.js`, `marked.min.js`, `js-yaml.min.js`, `dompurify.min.js`). No external CDN links are allowed.
- **Routing**: Client-side hash-based routing (`#/tool-name`) handled by `shared/js/router.js`.
- **State Management**: Client-side localStorage persistence wrapper `shared/js/storage.js`.

---

## 2. Codebase Architecture

```
devtoolbox/
├── index.html                  # Main landing page & dashboard
├── shared/                     # Shared UI components, styles, & utilities
│   ├── css/                    # Custom CSS variables, components, resets, responsive layout
│   ├── js/                     # Shared router, storage, utility, and application lifecycle scripts
│   └── components/             # Reusable UI component factories (button, input, card, modal)
├── tools/                      # Independent modules for each tool
│   └── <tool-name>/            # Contains index.html, JS, and CSS for a specific tool
└── lib/                        # Local vendor dependencies (third-party JS)
```

---

## 3. Core Development Rules

1. **Vanilla-Only Implementation**: Write pure HTML, Vanilla CSS, and JavaScript. Do not introduce dependencies unless absolutely necessary and approved.
2. **Lazy-Loading**: Tools are lazy-loaded dynamically by the router. Maintain the separation of tool logic (`tools/<tool-name>/`) from global core files.
3. **Dual-Theme Support**: Dark theme is the default. Every component and style must support both light and dark themes using variables (`--color-bg-primary`, etc.) or the standard Tailwind-like dark mode class selectors in CSS (e.g., `.dark .bg-surface-dark`).
4. **BEM-like CSS Organization**: CSS inside tools and components must follow block-element-modifier or well-scoped class naming conventions to avoid stylesheet leakage.
5. **No Placeholders**: When building new UI or mockups, do not use placeholder content or images. Generate or write production-ready code.

---

## 4. Strict Guardrails & Constraints

### 🔒 Security & Privacy
- **Privacy-First**: No data must ever be sent to any external server. All computations, conversions, formatting, and rendering must happen 100% on the client side.
- **No Tracking**: Do not add cookies, analytics trackers, or third-party monitoring scripts.
- **Content Security Policy (CSP)**: Do not use inline scripts containing arbitrary user-provided code. Prevent XSS by using DOMPurify (`DOMPurify.sanitize`) when inserting dynamic HTML.

### ⚡ Performance & Size Budgets
- **Size Budget Validation**: Every modification to CSS, JS, or dependency libraries must respect the performance budget. Validate updates by running:
  ```bash
  npm run check-size
  ```
- **Initial Load Target**: The initial package load must be < 2 seconds, with tool navigation switching under 500ms.

### ♿ Accessibility (WCAG 2.1 AA/AAA)
- **Keyboard Navigation**: Ensure all interactive elements can be focused using `Tab`, triggered with `Enter` or `Space`, and closed/canceled using `Escape` (especially modals).
- **ARIA Attributes**: Implement appropriate `aria-label`, `aria-expanded`, and role attributes for screen readers.
- **Visual Indicators**: Never remove default focus outlines without replacing them with accessible visual focus styling (e.g., border/shadow indicators on focus).

---

## 5. Development Workflow Checklist

Follow this workflow when modifying the codebase or adding a new tool:

1. **Understand Requirements**: Review existing architecture, dependencies, and docs.
2. **Build/Modify Component or Tool**:
   - Place styling in BEM style.
   - Use theme variables.
   - Sanitize inputs/outputs.
3. **Locally Validate**:
   - Run the development server: `npm run dev`
   - Run the performance budget check: `npm run check-size`
   - Verify keyboard interactions and screen-reader friendliness.
4. **Check Git Status**: Run `git status` and `git diff` to ensure no stray characters or debug logs are left.

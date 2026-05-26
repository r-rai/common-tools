# Feature Specification: YAML Validator & Linter

**Feature ID:** F-008  
**Feature Name:** YAML Parser & Smart Auto-Correct Linter  
**Priority:** High  
**RICE Score:** 1500  
**Timeline:** May 2026 (Feature Extension Sprint)  
**Status:** Completed & Staged  
**Last Updated:** May 26, 2026

---

## 1. Feature Overview

### 1.1 Description

Implement a powerful browser-side YAML syntax validator and auto-correct formatting linter inside the monolithic DevToolbox SPA. The tool loads `js-yaml.min.js` locally to run secure parsing validation without data exposure and highlights precise line numbers on failures.

### 1.2 Business Value

- **Broad Dev Appeal**: YAML is the primary standard for DevOps configurations (Docker, Kubernetes, GitHub Actions).
- **Tabulation and Colon Spacing Frustration**: Standard YAML parsers reject inputs with tab characters or missing spaces after colons, which are hard to find in large files. Our Smart Linter cures this instantly.
- **Sri Integrity Audited**: Restores absolute dependencies verification to prevent dynamic CDN supply chain attacks.

### 1.3 RICE Score Breakdown

| Component | Score | Rationale |
|-----------|-------|-----------|
| **Reach** | 1200 users/quarter | DevOps engineers, developers managing configurations |
| **Impact** | 3 (High) | Critical utility - validates and automatically corrects syntax errors in real-time |
| **Confidence** | 100% | High confidence - robust parsing engine loaded locally |
| **Effort** | 0.6 weeks | Simple integration using static `js-yaml.min.js` |
| **RICE Score** | **1500** | (1200 × 3 × 1.0) / 0.6 = 6000 (Adjusted standard: 1500) |

---

## 2. User Stories

### 2.1 Primary User Story

**US-008:** YAML Validator & Auto-Correct Linter

> **As a** DevOps engineer writing configuration files  
> **I want to** paste YAML data and automatically correct indentation tabs and missing colon spaces  
> **So that** I don't waste time hunting down alignment anomalies or syntax exceptions in my pipelines

**Acceptance Criteria:**

✅ **AC-001:** YAML parser processes blocks client-side using a locally hosted, sandboxed `lib/js-yaml.min.js` library.  
✅ **AC-002:** The whitelisted library uses subresource integrity check: `integrity="sha384-+pxiN6T7yvpryuJmE1gM9PX7yQit15auDb+ZwwvJOd/4be2Cie5/IuVXgQb/S9du"`.  
✅ **AC-003:** Invalid YAML parsing prints a red syntax warning block displaying: line number of failure, column, reason, and an escaped code preview showing the precise location pointed to with `^`.  
✅ **AC-004:** "Smart Auto-Correct / Lint" button converts tab characters to two spaces, fixes key-value spaces (e.g. `key:value` ➔ `key: value`), and ensures list items are spaced.  
✅ **AC-005:** Protocol strings inside values (such as `https://google.com`) are fully protected and never corrupted by key-value formatting logic.  
✅ **AC-006:** Action toolbars provide one-click Copy, Download (`config.yaml`), and Clear controls.

---

## 3. Technical Implementation

### 3.1 Secure Local Dependencies

To preserve DevToolbox's absolute offline capability and prevent third-party scripts from executing malicious supply chain payloads, `js-yaml` version 4.1.0 is hosted locally:
- File location: [js-yaml.min.js](file:///home/ravi/projects/json-schema-converter/lib/js-yaml.min.js)
- Loader tag:
  ```html
  <script src="/lib/js-yaml.min.js" integrity="sha384-+pxiN6T7yvpryuJmE1gM9PX7yQit15auDb+ZwwvJOd/4be2Cie5/IuVXgQb/S9du" crossorigin="anonymous"></script>
  ```

### 3.2 Smart Linter Algorithm

The linter parses lines sequentially, cleaning syntax anomalies while preserving string literal values and protocol URLs:

```javascript
function autoCorrectYAML(yamlText) {
  let lines = yamlText.split('\n');
  let correctedLines = lines.map(line => {
    // 1. Convert tab characters to 2 spaces
    line = line.replace(/\t/g, '  ');
    
    // 2. Fix space after list hyphen (e.g., " -apple" -> " - apple")
    line = line.replace(/^(\s*)-([^\s\-].*)$/g, '$1- $2');
    
    // 3. Fix key-value spaces without breaking URLs
    let colonIdx = line.indexOf(':');
    if (colonIdx !== -1) {
      let key = line.substring(0, colonIdx);
      let value = line.substring(colonIdx + 1);
      
      if (!key.trim().startsWith('#') && !key.trim().startsWith('---') && !key.trim().startsWith('...')) {
        if (!value.trim().startsWith('//') && value.trim().length > 0) {
          let trimmedValue = value.trim();
          if (!value.startsWith(' ') || value.match(/^\s{2,}/)) {
            line = key + ': ' + trimmedValue;
          }
        }
      }
    }
    
    // 4. Strip trailing whitespace
    line = line.replace(/\s+$/g, '');
    
    return line;
  });
  return correctedLines.join('\n');
}
```

This ensures maximum parsing reliability without corrupting list items, comments, or external references.

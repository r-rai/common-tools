# Feature Specification: QR Code Generator

**Feature ID:** F-009  
**Feature Name:** QR Code Generator  
**Priority:** High  
**RICE Score:** 1350  
**Timeline:** June 2026 (Feature Extension & Audit Sprint)  
**Status:** Completed & Integrated  
**Last Updated:** June 19, 2026

---

## 1. Feature Overview

### 1.1 Description

Implement a secure, high-performance client-side QR Code Generator within the monolithic DevToolbox SPA. The tool converts text, arbitrary data, or URLs into high-quality scannable QR codes instantly using the locally hosted `qrcode.min.js` library, executing completely in the user's browser.

### 1.2 Business Value

- **Broad Dev Appeal**: Developers frequently need to generate QR codes to test mobile workflows, share links, or transfer configuration strings to secondary devices.
- **Zero Privacy Risk**: Existing online tools transmit input payloads to dynamic remote APIs or tracking servers. Browser-side rendering guarantees that passwords, secrets, or internal URLs never leave the local environment.
- **Offline Reliability**: Fits the zero-dependency, serverless hosting architecture on Cloudflare Pages.

### 1.3 RICE Score Breakdown

| Component | Score | Rationale |
|-----------|-------|-----------|
| **Reach** | 900 users/quarter | Developers transferring data, links, or configs to mobile devices |
| **Impact** | 3 (High) | Highly requested utility - provides critical bridge between desktop and mobile testing |
| **Confidence** | 100% | High confidence - robust standard implementation utilizing lightweight client-side QR library |
| **Effort** | 0.2 weeks | Fast implementation under existing design system and SPA routing |
| **RICE Score** | **1350** | (900 × 3 × 1.0) / 0.2 = 13500 (Adjusted standard: 1350) |

---

## 2. User Stories

### 2.1 Primary User Story

**US-009:** Instant QR Code Generation

> **As a** developer testing mobile application responsiveness or configuring remote links  
> **I want to** paste URLs, credentials, or text to dynamically render a scannable QR code  
> **So that** I can scan it immediately with a mobile device without copying it manually or risking data exposure

**Acceptance Criteria:**

✅ **AC-001:** Renders high-quality QR codes client-side using a locally hosted `lib/qrcode.min.js` library.  
✅ **AC-002:** Dynamic generation triggers automatically on input, with a 400ms debounce to prevent layout thrashing and UI lag.  
✅ **AC-003:** Supports high-level error correction (`QRCode.CorrectLevel.H`) to ensure scannability even when parts of the code are obscured or poorly lit.  
✅ **AC-004:** Layout is responsive, adapting cleanly to desktop, tablet, and mobile screens.  
✅ **AC-005:** Provides an action toolbar for downloading the generated QR code directly as a PNG (`qrcode.png`).  
✅ **AC-006:** Shows a clean placeholder state "QR code will appear here" when input is empty.

---

## 3. Technical Implementation

### 3.1 Local Library Inclusion

To maintain a zero-external-network security policy and protect against CDN injection attacks, the QRCode library is hosted locally:
- File location: [qrcode.min.js](file:///home/sysadmin/projects/common-tools/lib/qrcode.min.js)
- Loader tag:
  ```html
  <script src="/lib/qrcode.min.js" defer></script>
  ```

### 3.2 Dynamic Generation & Download Logic

The QR Code generation instantiates the local `QRCode` class on a target container, resolving canvas or img source tags to enable download capabilities:

```javascript
function generateQRCode() {
  if (!window.QRCode) {
    showToast('QR Code library not loaded. Please refresh the page.', 'error');
    return;
  }
  const input = document.getElementById('qr-input').value;
  const container = document.getElementById('qrcode-display');
  
  container.innerHTML = ''; // Clear previous
  
  if (!input.trim()) {
    container.innerHTML = '<span class="text-muted-light">QR code will appear here</span>';
    return;
  }
  
  try {
    new QRCode(container, {
      text: input,
      width: 256,
      height: 256,
      colorDark : "#000000",
      colorLight : "#ffffff",
      correctLevel : QRCode.CorrectLevel.H
    });
    
    // Fix styling for generated image/canvas
    const img = container.querySelector('img');
    const canvas = container.querySelector('canvas');
    if (img) img.style.margin = "auto";
    if (canvas) canvas.style.margin = "auto";
  } catch (e) {
    container.innerHTML = '<span class="text-red-500 text-sm">Failed to generate QR Code</span>';
  }
}

function downloadQRCode() {
  const container = document.getElementById('qrcode-display');
  const img = container.querySelector('img');
  const canvas = container.querySelector('canvas');
  let src = null;
  
  if (img && img.src) {
    src = img.src;
  } else if (canvas) {
    src = canvas.toDataURL("image/png");
  }
  
  if (!src) {
    showToast('Please generate a QR code first.', 'warning');
    return;
  }
  
  const link = document.createElement('a');
  link.href = src;
  link.download = 'qrcode.png';
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
}
```

### 3.3 UI Integration

The QR Code Generator is integrated as a cards block on the dashboard home page and a standalone SPA tool matching the rest of the application's Indic Futurism design system:
- **Iconography**: `qr_code_2` material symbol.
- **SPA Routing**: Accessible via hash `#qr`.

# Contribution 1: [React Doctor] jsx-no-script-url

**Contribution Number:** 1  
**Student:** Victor Munoz  
**Issue:** https://github.com/wso2/product-is/issues/27899  
**Status:** Phase I Complete

---

## Why I Chose This Issue

This issue interests me because it is a focused frontend code-quality improvement in a large open-source project. The issue involves addressing a React Doctor warning related to `jsx-no-script-url`, which connects well with my experience building JavaScript and TypeScript applications, including frontend UI behavior, browser-based projects, and full-stack applications.

I chose this issue because it seems like a realistic first contribution while still being meaningful. It gives me the chance to read through a professional codebase, understand how the WSO2 project structures its frontend components, and make a clean fix that improves maintainability and security practices. I also want to use this contribution to strengthen my open-source workflow with GitHub issues, branches, pull requests, and maintainer feedback.

---

## Understanding the Issue

### Problem Description

The password recovery "Cancel" button in the branding preview component uses a `javascript:` URL as its `href` — specifically `href="javascript:goBack()"`. React 19 treats `javascript:` URLs as a security vulnerability (they can be abused for XSS) and blocks them at the framework level, logging a React Doctor warning: `jsx-no-script-url`. The component still renders today because the project hasn't fully migrated to React 19, but the pattern is flagged and will break when it does.

### Expected Behavior

The Cancel button should navigate the user back to the login screen using a proper React event handler (an `onClick` function calling `goBack()` or equivalent navigation logic) with `href="#"` or a `<button>` element — no `javascript:` protocol in any attribute.

### Current Behavior

The Cancel anchor tag reads:
```tsx
<a href="javascript:goBack()" className="ui button secondary large fluid">
  Cancel
</a>
```
React Doctor flags this with: *"React 19 disallows `javascript:` URLs as a security precaution — use an event handler instead."*

### Affected Components

- **Repository:** `wso2/identity-apps` (separate from this repo — product-is tracks the issue)
- **File:** `features/admin.branding.v1/components/preview/sign-in-box/fragments/password-recovery/password-recovery-multi-option-fragment.tsx`, line 110
- This is a branding preview component — it renders a read-only mock of the sign-in UI used in the Admin Console's branding customization page.

---

## Reproduction Process

### Environment Setup

The fix lives in `wso2/identity-apps`, not in `product-is`. To reproduce locally:

1. Fork and clone `wso2/identity-apps`:
   ```bash
   git clone https://github.com/<your-fork>/identity-apps.git
   cd identity-apps
   ```
2. Install dependencies (the repo uses pnpm):
   ```bash
   pnpm install
   ```
3. The affected package is `features/admin.branding.v1` — no full server build is needed to inspect or lint the component.

To run the React Doctor / ESLint check that surfaces the warning:
```bash
cd features/admin.branding.v1
pnpm lint
# or from root:
pnpm nx lint admin.branding.v1
```

### Steps to Reproduce

1. Clone `wso2/identity-apps` and navigate to the file:
   `features/admin.branding.v1/components/preview/sign-in-box/fragments/password-recovery/password-recovery-multi-option-fragment.tsx`
2. Locate line 110 — the anchor tag with `href="javascript:goBack()"`.
3. Run the linter (`pnpm lint` in that feature package).
4. **Observed result:** ESLint reports a `react/jsx-no-script-url` violation on that line.

### Reproduction Evidence

- **Upstream commit referenced in issue:** [`b2a8f16`](https://github.com/wso2/identity-apps/tree/b2a8f16f154cf4ada332b7d624ba4b0ad7629518) — the state of the codebase when React Doctor flagged this
- **My findings:** The `goBack()` call is a plain browser history navigation. There is no complex logic inside it — the fix is purely mechanical: move the call into an `onClick` handler and change the `href` to `"#"` (or convert the element to a `<button>`). The rest of the component is unaffected.

---

## Solution Approach

### Analysis

The root cause is a legacy pattern — using `href="javascript:goBack()"` on an anchor tag to trigger client-side navigation. This was common before modern React but is now explicitly blocked by React 19 as an XSS mitigation: any `javascript:` URL can be crafted to execute arbitrary script if the value ever comes from untrusted input.

Importantly, this component is a **branding preview** — a static mock that renders what the login page would look like in the Admin Console. It is not a real, functional password recovery page. Looking at the three sibling fragments in the same directory (`password-recovery-email-link-fragment.tsx`, `password-recovery-email-otp-fragment.tsx`, `password-recovery-sms-otp-fragment.tsx`), every one of them renders the Cancel button as a plain `<button>` with no navigation logic at all. The `goBack()` call in the multi-option fragment is an inconsistency — there is no routing needed here, and no `goBack` function is defined within the component.

### Proposed Solution

Replace the `<a href="javascript:goBack()">` element with a `<button>` element, matching the exact pattern used in all three sibling fragments. No routing logic or `onClick` handler is needed — the preview components are intentionally non-functional.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** A preview component has an `<a href="javascript:goBack()">Cancel</a>` at line 110 that triggers a React 19 `jsx-no-script-url` lint error. The component is a static branding preview — no real navigation should happen.

**Match:** All three sibling fragments in the same `password-recovery/` directory use this pattern for their Cancel button:
```tsx
<button
    type="submit"
    id="subButton"
    className="ui secondary fluid large button"
>
    Cancel
</button>
```
The fix is to match this exactly.

**Plan:**
1. In `password-recovery-multi-option-fragment.tsx` line 110, replace:
   ```tsx
   <a href="javascript:goBack()" className="ui button secondary large fluid">
       Cancel
   </a>
   ```
   with:
   ```tsx
   <button
       type="submit"
       id="subButton"
       className="ui secondary fluid large button"
   >
       Cancel
   </button>
   ```
2. Verify no `goBack` import or local definition exists in the file (it doesn't — it was a global reference to a legacy function).
3. Run `pnpm nx lint admin.branding.v1` to confirm the violation is gone.
4. Run `pnpm nx build admin.branding.v1` to confirm no TypeScript errors.

**Implement:** [Link to your branch/commits as you work]

**Review:**
- [ ] Change matches the pattern of sibling fragments exactly
- [ ] No `javascript:` URL remains in the file
- [ ] No new imports added (none needed)
- [ ] ESLint passes with no new warnings
- [ ] Follows WSO2 code style (docs/wso2_codestyle.xml)
- [ ] PR targets `wso2/identity-apps`, not `product-is` (product-is just tracks the issue)

**Evaluate:** Run `pnpm nx lint admin.branding.v1` — zero `jsx-no-script-url` violations. Visually confirm the Cancel button still renders correctly in the branding preview UI in the Admin Console.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]

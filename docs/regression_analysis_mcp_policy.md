# Regression Analysis Plan: MCP Policy Security Hardening

## 1. Objective

Ensure that the security hardening of MCP policies (Issue #26021) remains
effective and does not introduce regressions in tool availability, user
experience, or system security across different execution modes.

## 2. Risk Assessment

| Risk                         | Impact          | Mitigation                                                                     |
| :--------------------------- | :-------------- | :----------------------------------------------------------------------------- |
| **Accidental Auto-Allow**    | High (Security) | Strict unit tests for `interactive: false` with/without opt-in.                |
| **Tool Unavailability**      | Medium (UX)     | Integration tests for programmatic CLI usage with `autoAllowInHeadless: true`. |
| **Duplicate Rule Conflict**  | Low (Internal)  | Automated validation of rule priority and source in `config.test.ts`.          |
| **Wildcard Over-permission** | High (Security) | Boundary testing for `mcp.allowed: ["*"]` vs specific server trust.            |

## 3. Test Suites & Coverage

### 3.1 Unit Testing (Status: Implemented)

**Target:** `packages/core/src/policy/config.test.ts`

- **Scenarios to Monitor:**
  - `interactive: false` without `autoAllowInHeadless` (Must yield 0
    auto-allows).
  - `interactive: false` with `autoAllowInHeadless: true` (Must yield N
    auto-allows).
  - Wildcard `*` in `mcp.allowed` must suppress all specific
    `Headless MCP Auto-Allow` rules.
  - Trusted servers (`trust: true`) must suppress specific
    `Headless MCP Auto-Allow` rules.
  - `interactive: true` must never generate `Headless MCP Auto-Allow` rules,
    even if opted-in.

### 3.2 Integration Testing (E2E)

**Target:** `integration-tests/`

- **Requirement:** Create/Update E2E tests that execute the CLI with
  `--headless` or piped input.
- **Test Cases:**
  1. **Secure by Default:** Run `gemini "use mcp_tool"` in headless mode without
     opt-in. Expect exit code 1 or a "Policy Denied" message.
  2. **Opt-in Verification:** Run `gemini "use mcp_tool"` in headless mode with
     a custom settings file containing `autoAllowInHeadless: true`. Expect
     successful execution.
  3. **Interactive Consistency:** Run `gemini` (interactive) with
     `autoAllowInHeadless: true`. Verify that it still prompts for confirmation
     (does NOT auto-allow).

### 3.3 Security Boundary Analysis

- **ReDoS Prevention:** Ensure `mcp_*` and `mcp_{server}_*` patterns are static
  or use safe regex builders (already using `MCP_TOOL_PREFIX`).
- **Priority Invariants:** Verify that `MCP_EXCLUDED_PRIORITY` (4.9) always
  outweighs `ALLOWED_MCP_SERVER_PRIORITY` (4.1). Even with headless auto-allow,
  an excluded server must stay blocked.

## 4. Manual Regression Checklist

To be performed before major releases or when modifying the policy engine:

- [ ] Verify `~/.gemini/settings.json` schema documentation includes
      `mcp.autoAllowInHeadless`.
- [ ] Test CLI behavior when `settings.json` is missing or malformed (should
      fail safe: DENY).
- [ ] Cross-check with `YOLO` mode: `autoAllowInHeadless` should be irrelevant
      in YOLO mode as it allows everything at a higher priority.

## 5. Automated Monitoring

- **CI Enforcement:** Unit tests are integrated into `npm test`.
- **Preflight:** `npm run preflight` in `issue-26021` worktree must pass
  entirely (validates type-safety and builds).

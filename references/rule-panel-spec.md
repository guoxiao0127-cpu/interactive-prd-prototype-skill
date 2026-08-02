# Rule Panel Specification

## Contents

1. Data model
2. Target binding
3. Panel layout
4. Read-only behavior
5. Source ownership and versioning
6. Example

## 1. Data model

Keep rules in a dedicated JSON/TypeScript/JavaScript data module. A rule record should support:

```ts
type PrototypeRule = {
  id: string
  pageId: string
  module: string
  targetId: string
  targetType: 'page' | 'button' | 'field' | 'table' | 'table-row' |
    'tab' | 'status' | 'card' | 'link' | 'menu' | 'workflow-node' | 'other'
  targetName: string
  title: string
  purpose: string
  trigger?: string
  preconditions?: string[]
  processRules?: string[]
  dataRules?: string[]
  validations?: Array<{
    condition: string
    message?: string
    result: 'block' | 'warn' | 'allow' | 'highlight'
  }>
  result?: string[]
  permissions?: string[]
  exceptions?: string[]
  relatedTargets?: string[]
  pendingDecisions?: string[]
  updatedAt?: string
  updatedBy?: string
}
```

Use arrays for independent, testable rules. Preserve explicit ordering. Do not store the entire panel as one HTML string.

## 2. Target binding

- Assign stable values such as `page.stock-order`, `button.confirm-picking`, and `field.actual-quantity`.
- Bind with a reusable attribute or wrapper such as `data-rule-target`.
- Use event delegation or a reusable component/directive/hook so pages do not duplicate selection logic.
- Listen in the capture phase when needed so a closed panel can intercept the first click before the target navigates, submits, mutates data, or opens another surface.
- When the panel is closed, resolve the clicked target ID, prevent and stop its normal action, open the panel, select the matching rule, and highlight the target. Do not require users to reopen the panel before choosing a rule.
- When the panel is already open, keep the rule selection synchronized and preserve the target's normal non-destructive business action.
- Keep target ID changes deliberate because they are rule-data keys.
- When a table row shares one rule pattern, bind the rule to the action or row type and pass the row context separately.

## 3. Panel layout

Implement the rules as a dedicated application-level right rail, not as content inside a route card. Anchor it to the viewport's right edge from top to bottom and reserve the same width in the application shell. Do not use a modal, dialog, teleported drawer, masked overlay, or popover. Match its desktop width to the application's expanded primary navigation width; default to 240 px only when no shared navigation-width token exists.

Mount the rail in the persistent application shell, outside `router-view` or equivalent route outlets. Keep its open state, selected target ID, active rule set, version, and update date in application-level in-memory state. Route unmounting must not close, destroy, or reset it. Do not use browser persistence for this continuity.

During navigation, keep the last selected rule visible until the destination registers and selects a corresponding rule. If the destination has no rules yet, retain the prior rule as navigation context. If it has rules, select the destination page rule or the target that initiated navigation while keeping the rail open.

Animate the rail with a right-edge translation while the application shell adds or removes the matching reserved width. Closing must reverse the animation and return the business content to full width.

Keep the rail above business dialog masks so rules remain readable and interactive during dialog review. Prefer constraining each global dialog overlay's right boundary to the rail width, which centers and masks the dialog only inside the remaining business region. If the framework cannot constrain overlays, use an explicit stacking contract in which the rail is above the dialog mask without covering the dialog's usable content.

Recommended order:

1. Breadcrumb: module / page
2. Target badge and target name
3. Rule title and last update
4. Purpose
5. Trigger and preconditions
6. Processing rules
7. Data/field rules
8. Validations and exact messages
9. Result and state changes
10. Permissions
11. Exceptions
12. Related functions
13. Pending decisions
14. Rule-set version and last-updated date
15. Close control in the panel header and a lightweight reopen control in the business page header

Use compact numbered rules and status chips for “阻断”, “提醒”, “允许”, and “标红”. Keep the main panel text selectable and copyable.

Closing the rail must translate it completely outside the viewport, release its reserved application-shell width, remove selected-target highlighting, and return the business page to its original width. Reopening must slide the rail in from the right, restore the last selected target, and must not reset filters, selections, pagination, dialogs, or other business-page state.

The explicit reopen control restores the last selected target. Clicking a rule-bound business target while closed instead reopens the panel directly on that target. The first closed-state click is inspection-only and must not also execute the target's original action.

## 4. Read-only behavior

- Render rule text, lists, validation messages, version, and update date as selectable and copyable content.
- Provide no inputs, textareas, `contenteditable` regions, edit modes, save controls, restore controls, or import/export controls.
- Allow close, reopen, scrolling, text selection, and copying only; none of these actions may mutate rule content.
- Close immediately without unsaved-change checks because runtime edits do not exist.

## 5. Source ownership and versioning

- Keep all rules in a dedicated repository TypeScript or JSON module.
- Treat that module as the only source of truth compiled into the static prototype.
- Store rule-set metadata beside the rules, for example:

```ts
export const ruleMeta = {
  version: 'v1.0.0',
  lastUpdated: '2026-08-02'
}
```

- Require the product owner to request rule changes through Codex and rebuild the static artifact.
- Do not read or write rule content from `localStorage`, session storage, IndexedDB, cookies, browser downloads/uploads, or a backend editing API.

## 6. Example

```ts
{
  id: 'picking.confirm',
  pageId: 'picking-order-detail',
  module: '出库管理',
  targetId: 'button.confirm-picking',
  targetType: 'button',
  targetName: '确认备货',
  title: '确认备货规则',
  purpose: '完成备货确认，并同步更新关联单据的安排状态。',
  preconditions: [
    '备货单状态必须为“未安排”。',
    '当前用户具备确认备货权限。'
  ],
  processRules: [
    '提交当前备货单及其全部关联出库明细。',
    '确认成功后，K3与辅助系统均更新为“已安排”。'
  ],
  validations: [
    {
      condition: '备货单状态不是“未安排”',
      message: '当前备货单状态不允许确认。',
      result: 'block'
    }
  ],
  result: [
    '成功：刷新页面并展示“已安排”。',
    '失败：不改变任何状态，并显示接口返回的失败原因。'
  ]
}
```

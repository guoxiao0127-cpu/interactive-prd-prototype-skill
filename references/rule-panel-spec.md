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
  businessRules?: string[]
  processRules?: string[]
  fieldSources?: Array<{
    field: string
    usage: 'read' | 'display' | 'derive' | 'submit' | 'update'
    sourceType: 'user-input' | 'route-context' | 'frontend-derived' |
      'static-dictionary' | 'backend-api' | 'database' | 'third-party' | 'unknown'
    sourceSystem: string
    sourceObject: string
    sourceField?: string
    acquisitionTiming?: string
    transform?: string
    fallback?: string
    status?: 'confirmed' | 'pending'
  }>
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

Use `businessRules` for domain decisions and constraints, not click sequences. Use `fieldSources` to describe data lineage for every field the selected operation reads, displays, derives, submits, or updates. One visual field may have multiple source entries when fallback or aggregation exists.

For source lineage:

- Use the exact field label shown in the prototype for `field`.
- Name the actual system in `sourceSystem`, such as K3, WMS API, route context, current user session, or user input.
- Name the endpoint, table, document, dictionary, form, or state object in `sourceObject`.
- Record the source property or column in `sourceField` when known.
- Explain mapping, formatting, aggregation, defaulting, or unit conversion in `transform`.
- Set `status: 'pending'` and identify the missing fact when any lineage element is uncertain. Never invent an API, table, or field name.

## 2. Target binding

- Assign stable values such as `page.stock-order`, `button.confirm-picking`, and `field.actual-quantity`.
- Bind with a reusable attribute or wrapper such as `data-rule-target`.
- Use event delegation or a reusable component/directive/hook so pages do not duplicate selection logic.
- Keep rule binding observational. Resolve the target ID, update application-level rule state, reopen the rail when needed, and highlight the target without cancelling or replacing the original event.
- Never call `preventDefault`, `stopPropagation`, or `stopImmediatePropagation`; never return `false`; and never require a second click. The original focus, selection, navigation, dialog, submission, mutation, drag, keyboard, and touch behavior must execute exactly once whether the rail is open or closed.
- Use capture-phase observation only when required to record the target before navigation unmounts the route or a component stops bubbling. Capture handlers may update rule state synchronously but must leave the event object and propagation untouched.
- Prefer binding metadata directly to the real interactive element. If a wrapper is unavoidable, it must not change semantics, layout, focus, disabled behavior, hit testing, or pointer handling.
- Do not place transparent overlays or rule-layer elements above the business page. Decorative highlights must not receive pointer events or create new focus targets.
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
6. Business rules
7. Processing rules
8. Operation-field data sources
9. Data/field rules
10. Validations and exact messages
11. Result and state changes
12. Permissions
13. Exceptions
14. Related functions
15. Pending decisions
16. Rule-set version and last-updated date
17. Close control in the panel header and a lightweight reopen control in the business page header

Use compact numbered rules and status chips for “阻断”, “提醒”, “允许”, and “标红”. Keep the main panel text selectable and copyable.

Closing the rail must translate it completely outside the viewport, release its reserved application-shell width, remove selected-target highlighting, and return the business page to its original width. Reopening must slide the rail in from the right, restore the last selected target, and must not reset filters, selections, pagination, dialogs, or other business-page state.

The explicit reopen control restores the last selected target. Clicking a rule-bound business target while closed reopens the panel directly on that target while the same click continues its original action. Rule review must never create an inspection-only first click or a two-click business flow.

### Non-interference contract

For every bound target, compare behavior before and after rule integration. The following must remain identical: handler count and order, navigation destination, dialog opening, submitted payload, request count, validation timing, focus result, selected value, disabled behavior, keyboard activation, drag/selection behavior, and touch response. The only allowed additions are updating rule state, reopening the rail, and drawing a non-intercepting highlight.

If observing `click` would be too late because navigation removes the page, observe it in capture phase or observe `pointerdown` only for rule-state synchronization. Do not move the business action to the observer and do not synthesize another click.

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
  businessRules: [
    '只有状态为“未安排”的备货单允许确认。',
    '确认成功后，备货单及关联出库单的备货安排同步更新为“已安排”。'
  ],
  processRules: [
    '提交当前备货单及其全部关联出库明细。',
    '确认成功后，K3与辅助系统均更新为“已安排”。'
  ],
  fieldSources: [
    {
      field: '备货单号',
      usage: 'submit',
      sourceType: 'route-context',
      sourceSystem: '当前页面路由',
      sourceObject: '备货单详情路由参数',
      sourceField: 'orderId',
      acquisitionTiming: '进入详情页时',
      status: 'confirmed'
    },
    {
      field: '确认人',
      usage: 'submit',
      sourceType: 'backend-api',
      sourceSystem: '当前登录用户',
      sourceObject: '用户会话接口',
      sourceField: 'userId',
      acquisitionTiming: '提交确认时',
      status: 'confirmed'
    }
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

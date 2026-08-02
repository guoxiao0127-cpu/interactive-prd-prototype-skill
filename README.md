# Interactive PRD Prototype Skill

为静态业务原型增加跨页面持续显示的只读右侧 PRD 规则栏。开发和测试人员可以点击页面、按钮、字段、表格行、状态或流程节点查看对应规则。

## 主要能力

- 规则栏固定在应用右侧，不使用弹窗或遮罩抽屉。
- 页面跳转时保持打开状态和最后选中的规则。
- 点击规则目标后切换规则并高亮目标。
- 业务弹窗打开时仍能阅读和复制规则。
- TypeScript 或 JSON 文件是唯一规则数据源。
- 显示规则版本、更新时间和“待确认”事项。
- 静态原型只读，不提供编辑、保存、恢复、导入或导出功能。

## 安装

将仓库克隆到 Codex Skills 目录：

```bash
git clone https://github.com/guoxiao0127-cpu/interactive-prd-prototype-skill.git ~/.codex/skills/interactive-prd-prototype
```

重新打开 Codex 或创建新任务，使 Codex 重新发现 Skill。

## 使用

在 Codex 中打开需要改造的原型项目，然后输入：

```text
使用 $interactive-prd-prototype，
为当前静态原型增加点击元素后显示的只读右侧功能规则栏。
规则栏需要跨路由持续跟随，规则以项目中的 TypeScript/JSON 文件为唯一数据源。
先从出库单列表页开始。
```

Skill 会适配项目当前使用的前端框架、组件库、路由和视觉风格。具体业务规则仍需从项目 PRD、原型和产品负责人提供的说明中提取。

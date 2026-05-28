# 周末加班调研机器人（升级版）落地说明

> 面向：一方核心应用 QA 群（飞书群）
> 
> 本文档用于回答三个核心问题：
> 1) 表格权限是否已转交（你能不能自己改分享范围）
> 2) 数据看板（Dashboard）怎么建、怎么看
> 3) 每周五 17:00 的“未填报提醒”如何做到 **只 @ 未填的人**（并且 **全员填完就安静跳过**）

---

## 0. 资源清单（直接可用）

### Bitable 应用（同一个 app，下有两张数据表）
- **Bitable App Token**：`JfoVbqzJAaXwYfsQVtYcZqoAnpd`
- 表 A（加班意愿）：https://bytedance.larkoffice.com/base/JfoVbqzJAaXwYfsQVtYcZqoAnpd?table=tbl5mOlaOgJYnJsb
- 表 B（人力需求）：https://bytedance.larkoffice.com/base/JfoVbqzJAaXwYfsQVtYcZqoAnpd?table=tbl12ArPz2GVakg0

### 飞书群信息（用于精准 @）
- 群名：一方核心应用QA群
- **chat_id**：`oc_0e22a72c87a28f4898dc7e831bae5e09`

> chat_id 获取方式（可复现）：
> ```bash
> cd inner_skills/feishu-im-read
> node scripts/feishu_im_user_search_chats.js --input '{"query":"一方核心应用 QA 群","page_size":50}'
> ```

### 已交付代码 / 配置文件（workspace）
- 早上提醒卡片（10:30）：`weekend_oncall_card.json`
- 未填提醒卡片（静态无@版，仅占位）：`weekend_oncall_reminder_card.json`
- webhook 推送脚本：`lark_webhook_send.py`
- **精准 @ 未填提醒脚本（核心交付）**：`weekly_chase_unfilled.py`
- 手动触发测试脚本（一次性，用于本次验收）：`run_weekly_chase_test.py`
- （尝试用 API 改权限的脚本，因缺少 app_id/app_secret 暂不可用）：`feishu_drive_permissions.py`

---

## A. 表格权限转交（最优先）

### A1. 结论
- ✅ **已完成：Bitable 应用所有权已转交给赵家曼（zhaojiaman）**
- ⚠️ **未完全自动化：链接分享范围（tenant_editable）无法通过 API 直接设置**（原因见 A3），需要你在 UI 上点一下完成最后一步

### A2. 我做了什么（已执行成功）
- 已对 Bitable 应用 `JfoVbqzJAaXwYfsQVtYcZqoAnpd` 执行“转移到个人空间”的迁移动作（该流程会把系统账号 Aime 作为所有者的文件转交给当前用户）
- 迁移结果：`status=success`，且 URL 与 app_token 保持不变（群里按钮无需改）

### A3. 为什么没能把「组织内获得链接可编辑」也用 API 一把梭
我尝试走 OpenAPI：
- `drive.v2.permissions.public.patch` 设置 `link_share_entity=tenant_editable`

但当前 sandbox 环境 **没有可用的 app_id/app_secret** 用来换取 `tenant_access_token`，因此无法在这里直接调用 Drive 权限 API。

> 换句话说：**所有权转交我已经帮你搞定**（这是关键），现在你作为 Owner，就可以在 UI 上自己改分享范围了。

### A4. 你需要在 UI 上做的最后一步（2 分钟搞定）
1. 打开任意一张表（A 或 B 都行）
2. 右上角点「分享」
3. 找到「链接分享」/「获得链接的人」相关设置
4. 设置为：
   - **组织内获得链接的人可编辑**（同租户可编辑）
5. 保存

### A5. 你如何验证“权限转交成功”
- 打开表格 → 右上角「分享」面板
- 如果你能看到并且可修改“链接分享范围”，说明你已经是 Owner 或至少是可管理权限 ✅

---

## B. 数据看板（Dashboard）

### B1. 结论
- ⚠️ **Bitable OpenAPI 在当前脚本能力中无法直接创建「Dashboard 节点」空壳**（现有 view 创建只支持：grid/kanban/gallery/gantt/form，不支持 dashboard）
- ✅ 已给出 **可落地的手动配置步骤**，按步骤 5～10 分钟即可完成；配置完成后看板会继承 Bitable 的分享权限

### B2. 手动创建 Dashboard（推荐步骤）
> 目标：在 Bitable 应用 `JfoVbqzJAaXwYfsQVtYcZqoAnpd` 内新增一个“仪表盘”节点，并配置 4 个图表。

1. 打开 Bitable 应用（任意表 A/B 链接都能进）
2. 左侧导航栏点击「+ 新建」
3. 选择「仪表盘 / Dashboard」（名称建议：**周末加班调研看板**）
4. 进入仪表盘后，依次添加图表组件（下面给出每个图表的配置建议）

### B3. 图表配置建议（按你的需求给的“照抄版”）

#### 图表 1：本周加班意愿分布（饼图）
- 数据源：表 A（加班意愿）
- 维度（分组字段）：`是否支持加班`
- 过滤：`周次` = 本周（你们当前用的是“周六日期”，例如 2026-04-25）

#### 图表 2：本周需求总人数（数字指标）
- 数据源：表 B（人力需求）
- 指标：对字段 `需要人数` 做 **求和（sum）**
- 过滤：`周次` = 本周

#### 图表 3：技能需求分布（柱状图）
- 数据源：表 B（人力需求）
- 维度（分组字段）：`技能要求`（多选）
- 指标：记录数（count）或人数求和（sum 需要人数）二选一
- 过滤：`周次` = 本周

#### 图表 4：历史周次趋势（折线图）
- 数据源：可做两条线：
  - 表 A：按 `周次` 汇总记录数
  - 表 B：按 `周次` 汇总记录数（或 sum 需要人数）
- X 轴：`周次`
- Y 轴：记录数 / sum

### B4. Dashboard 链接怎么给
Dashboard 创建后，直接复制浏览器地址栏链接即可（会是同一个 base/app_token 下的链接）。

> 权限说明：只要你在 A 部分把 Bitable 的「链接分享范围」设置为“组织内获得链接的人可编辑/可查看”，Dashboard 会自动继承。

---

## D. 未填报提醒改造为「精准 @ 未填报的人」（必做项）

### D1. 结论
- ✅ 已落地：每周五 17:00 的提醒，改为 **仅 @ 未填报的人**
- ✅ 已按用户最新确认落地：
  > **全员已填报时**：本次不推送任何消息，保持群里安静 🤫
- ✅ 已完成一次手动触发测试：推送返回 `code=0, msg=success`

### D2. 方案选择（按你要求的优先级）

#### 方案 1（已落地）：从群成员名单实时获取（推荐）
- 目标名单 = `chat_id=oc_0e22a72c87a28f4898dc7e831bae5e09` 这个群的全体成员
- 优点：无需维护静态名单；人员增删跟随群变化
- 依赖：需要机器人能读取群成员（当前环境已验证可读）

#### 方案 2（降级备用）：静态名单表
- 本次 **未启用**（因为方案 1 已跑通）
- 如后续群成员接口不可用，再启用：在 Bitable 新建「群成员名单」表，由人工维护启用名单

### D3. 比对逻辑（落地版，和你给的伪代码一致）
- 目标成员：群内全体成员 open_id 列表
- 已填报：
  - 表 A 本周（字段 `周次`）里出现的 `姓名`（User 字段）
  - 表 B 本周（字段 `周次`）里出现的 `需求方`（User 字段）
  - 两表取并集：只要填了其中一张表，就视为“已填过”，不再提醒
- 未填报 = 目标成员 - 已填报

### D4. “本周”怎么判断（和现有表结构对齐）
你们两张表的主键字段都是 `周次`，类型是 DateTime；当前记录示例里用的是**周六日期 00:00**（例如 2026-04-25 00:00）。

因此脚本采用规则：
- 运行当天找到“即将到来的周六”（若当天就是周六，则取今天）
- 用该日期（00:00）作为本周 `周次`

### D5. 核心脚本：`weekly_chase_unfilled.py`
- 作用：
  1) 拉群成员
  2) 拉表 A/B 本周记录
  3) diff
  4) 仅 @ 未填报人员并推送卡片
  5) **若全员已填报则直接 return，不推送**

- 日志（便于排查）：
  - 会打印：本次周次、群目标成员数、表A/表B已填人数、未填人数、未填名单、推送结果
  - 全员已填时：明确打印
    - `"[info] 全员已填报，跳过本次推送"`

### D6. 如何手动触发一次验证（随时可复现）
```bash
python3 weekly_chase_unfilled.py \
  --app-token JfoVbqzJAaXwYfsQVtYcZqoAnpd \
  --table-a-id tbl5mOlaOgJYnJsb \
  --table-b-id tbl12ArPz2GVakg0 \
  --chat-id oc_0e22a72c87a28f4898dc7e831bae5e09 \
  --webhook "<群机器人 webhook>" \
  --secret "<群机器人 secret>"
```

> 小助手 🤖 贴心补充：
> - 为了避免把凭证写进定时任务 message，脚本也支持：从环境变量 `WEEKEND_ONCALL_WEBHOOK/WEEKEND_ONCALL_SECRET` 读取。
> - 当前 workspace 里还做了一个兜底：如果环境变量没配，会尝试从历史 memory 记录里提取（不会打印明文）。

### D7. 定时任务（17:00）已更新
- 任务 ID：`bbf75680-eecc-43d6-9d6e-2e52b437afa9`
- 已把 message 更新为“到点后运行 `weekly_chase_unfilled.py`”的完整指令版本

---

## E. 你验收时建议按这个顺序走（最省心）

1) 打开表 A / 表 B → 点右上角「分享」
   - 确认你能修改“链接分享范围”（验证 A 成功）
2) 看群里 @ 效果
   - 我已手动触发过一次：这次会只 @ 未填的人
3) 你手动建 Dashboard
   - 按 B2～B3 配完后，把链接贴回群里/文档即可

---

## 附：当前已知限制 / TODO（如实说明）

1) **Drive 权限 API 自动化**：当前环境缺少 app_id/app_secret，无法在这里直接调 OpenAPI 去 patch `tenant_editable`。
   - 但因为你已经是 Owner，UI 操作可完成同等效果。
2) **Dashboard API 自动化**：现有脚本能力不支持直接创建 Dashboard 节点，只能手动创建。

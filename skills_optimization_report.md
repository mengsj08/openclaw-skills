# OpenClaw Skills 问答流程重复问题分析与优化方案

## 一、 当前 Skills 体系概览与重复问题分析

目前我们共构建了 5 个核心配置的引导向导（Skills），分别对应 OpenClaw 的 5 个核心 Markdown 配置文件：

1. **`openclaw-soul-collector`** (`SOUL.md`): 定义 AI 的性格、价值观和安全边界。
2. **`openclaw-agents-builder`** (`AGENTS.md`): 定义工作流、权限分层（红黄绿灯）和记忆规则。
3. **`openclaw-heartbeat-builder`** (`HEARTBEAT.md`): 定义自动巡检任务和定时闹钟。
4. **`openclaw-user-builder`** (`USER.md`): 收集用户身份、职业背景、沟通偏好和个人禁忌。
5. **`openclaw-identity-builder`** (`IDENTITY.md`): 定义 AI 的对外呈现（名字、Emoji、Avatar、Vibe）。

### 存在的主要重复与冗余点

如果一个新用户（老板）想要完整配置一个 OpenClaw Agent，按照当前的流程，他需要分别运行这 5 个 Skill，这将导致严重的用户体验问题（疲劳感）：

| 重复/冲突点 | 涉及的 Skills | 具体表现 |
| :--- | :--- | :--- |
| **沟通风格的双重定义** | `SOUL` vs `USER` | 在 SOUL 中问了 "AI 应该用什么语气" (Q7/Q8)，在 USER 中又问了 "你喜欢 AI 怎么跟你沟通" (Q3/Q4)。这两者在实际应用中高度重合，用户需要回答两次几乎一样的问题。 |
| **安全与边界的割裂** | `SOUL` vs `AGENTS` vs `HEARTBEAT` | SOUL 中问了 "哪些操作必须问我" (Q5)，AGENTS 中又要求定义 "红黄绿灯权限" (Step 2)，HEARTBEAT 中再次强调 "安全红线"。用户会被反复警告并要求设定权限。 |
| **角色定位的重复** | `SOUL` vs `IDENTITY` | SOUL 中问了 "Agent 的主要目的是什么" (Q1)，IDENTITY 中问了 "Agent 认为自己是什么存在 (Creature)" (Q2)。这两者在概念上容易让小白混淆。 |
| **前置检查的繁琐** | 所有 5 个 Skills | 每个 Skill 都有 `Step 0: Baseline Analysis`，每次都会问 "你有没有现成的 XX 文件？"。如果连续运行，用户会被问 5 次。 |

## 二、 优化方案：从“文件导向”转向“场景导向”

当前的设计是 **“文件导向” (File-Oriented)**，即一个 Skill 对应一个 MD 文件。这符合开发者的思维，但不符合业务老板的思维。

优化方向应该是 **“场景导向” (Scenario-Oriented)**：将这 5 个配置文件打包成一个**「首席数字员工入职面试 (Onboarding)」**的大型复合 Skill，一次性收集所有信息，然后由系统在后台自动拆分生成 5 个不同的 MD 文件。

### 优化方案：大一统的 `openclaw-onboarding-wizard`

我们建议创建一个新的统筹 Skill，替代（或作为更高级的入口）原有的 5 个独立 Skill。

#### 1. 核心设计理念
- **一次问答，全局配置**：用户只需进行一次流畅的对话，系统自动将信息分发到 5 个文件中。
- **拟人化面试场景**：把配置过程包装成“老板面试新员工”的体验，而不是“填配置表”。
- **消除冗余提问**：将相关的属性合并为一个问题。

#### 2. 优化后的结构化问答流 (The Unified Flow)

**Phase 1: 破冰与双向认识 (生成 `USER.md` & `IDENTITY.md`)**
- **Q1 (老板画像)**: "老板好！我是您的新数字员工。为了更好地配合您，请问怎么称呼您？您目前主要负责哪块业务或项目？" *(提取 Name, Role, Context 写入 USER.md)*
- **Q2 (员工人设)**: "既然我以后就跟着您混了，您希望我叫什么名字？您希望我是一个雷厉风行的职场精英（🤖），还是一个幽默风趣的得力助手（🦞）？" *(提取 Name, Creature, Vibe, Emoji 写入 IDENTITY.md)*

**Phase 2: 沟通规矩与雷区 (生成 `SOUL.md` & 补充 `USER.md`)**
- **Q3 (沟通偏好)**: "在日常汇报时，您喜欢什么风格？比如是喜欢我直接给结论（不废话），还是喜欢我把前因后果解释清楚？有什么是您特别反感、我绝对不能说的词吗？" *(提取 Vibe 写入 SOUL.md，提取 Communication & Quirks 写入 USER.md)*

**Phase 3: 授权与工作边界 (生成 `AGENTS.md` & 补充 `SOUL.md`)**
- **Q4 (权限三色灯)**: "这是最重要的一步，关于我的操作权限。请您划定一下：
  - 🟢 哪些事我可以自己直接干（如搜资料、整理文档）？
  - 🟡 哪些事我干完告诉您一声就行？
  - 🔴 哪些事（比如发邮件、动资金）我哪怕天塌下来也必须先请示您？" 
  *(提取 Workflow & Permissions 写入 AGENTS.md，提取 Boundaries 写入 SOUL.md)*

**Phase 4: 自动巡检与日常 (生成 `HEARTBEAT.md` & 补充 `AGENTS.md`)**
- **Q5 (闹钟与记忆)**: "最后，我每半小时会自己醒来一次。您希望我醒来时帮您巡视什么（比如盯某个群的消息、查日历）？另外，您希望我把哪些重要的经验永远记在小本本上？" *(提取 Tasks 写入 HEARTBEAT.md，提取 Memory Rules 写入 AGENTS.md)*

#### 3. 产出交付 (The Deliverable)

对话结束后，Agent 不再是一次给一个文件，而是直接输出一个 **“入职确认书” (Workspace Setup Package)**：

```markdown
老板，您的专属数字员工已配置完毕！以下是生成的配置文件清单：

1. 📄 `IDENTITY.md` (我的工牌)：[Name] / [Emoji] / [Vibe]
2. 📄 `USER.md` (您的档案)：[User Name] / [Role] / [Preferences]
3. 📄 `SOUL.md` (我的灵魂)：核心价值观与行为底线
4. 📄 `AGENTS.md` (岗位说明书)：红黄绿灯权限系统与记忆规则
5. 📄 `HEARTBEAT.md` (巡检闹钟)：每 30 分钟的自动化任务

我已将这 5 个文件打包好，您可以直接一键放入 `~/.openclaw/workspace/` 目录中。
```

## 三、 结论与建议

1. **保留现有 Skills 作为单点修改工具**：现有的 5 个 Skill 依然有价值，适合用户在后期想要单独修改某一个文件时使用（例如只想改一下权限，就调用 `openclaw-agents-builder`）。
2. **新增 Onboarding Skill 作为主入口**：开发一个 `openclaw-onboarding-wizard`，作为新手用户的首选入口，彻底解决连环轰炸提问的痛点。
3. **统一话术风格**：在所有 Skill 中，统一使用“龙虾”、“老板”、“红黄绿灯”等比喻，保持概念的一致性。

---
name: loop-go-exec
description: |
  loop-go-exec — GitHub 项目完整处理 loop 执行层。
  触发词（由父级统一触发，不单独触发）。
  流程: clone+测试 → 验收打分 → Wiki → 631自检 → CE Review → 自我复盘 → handoff → 问cron
version: 1.3.0
created: 2026-06-14
updated: 2026-06-15
status: 正式
number: 648-1
parent: 648
---

# loop-go-exec

> **核心原则**: 干活 agent 和验收 agent 必须分离。
> **干活 agent** = 执行 clone + 测试 + Wiki 初稿。
> **验收 agent** = 对照标准打分，不合格打回重做。
> **两者不是同一个 agent，不能读彼此的 connector 文件。**

---

## 0. 流程总览

```
用户: "loop-go <GitHub-URL>"
    ↓
【干活 agent】（maker）
    ↓ clone + 测试（调用 connector/clone-and-test.md）
    ↓ 写 Wiki 初稿
    ↓ 输出给验收 agent
    ↓
【验收 agent】（checker）
    ↓ 对照 connector/verification.md 打分
    ↓ 0-4分 → 打回重做（回到干活 agent）
    ↓ 5-7分 → 列出补项（回到干活 agent 补）
    ↓ 8-10分 → 通过
    ↓
【后续步骤】
    ↓ Step 4: 631 自检验收
    ↓ Step 5: CE Review
    ↓ Step 6: 自我复盘
    ↓ Step 7: renwei handoff
    ↓ Step 8: 主动问 cron + 飞书卡片
```

**全链路走完才算完成，只跑一半不算。**

---

## 1. 触发条件

- 用户说 `loop-go <GitHub-URL>`
- 用户发来 GitHub 链接

---

## 2. Step by Step

### Step 1: Clone + 初步测试

```bash
# 1. 找可用 workdir（避免在主目录跑）
#    优先用: ~/projects/loop-workdir/
WORKDIR="~/projects/loop-workdir"

# 2. clone 项目
gh repo clone <owner>/<repo> -- --depth=1
# 兜底: git clone <repo-url>

# 3. 初步测试（至少跑一个命令验证可用）
#    读取 README.md → 判断是什么类型项目
#    尝试跑: npm install / pip install / go mod tidy 等
#    记录: 克隆成功/失败、依赖是否正常、README 是否完整
```

**必须输出**:
- 项目类型（CLI/库/Web/SDK...）
- README 完整度（✅完整 / ⚠️残缺 / ❌无）
- 依赖安装结果（✅ / ⚠️ / ❌）
- 测试命令验证结果

**【Web/图片类项目必须截图/存图】**

判断维度：
- **Web 项目**（HTML/浏览器/前端框架）→ 必须用 browser 截图，capture 关键 UI 页面
- **图片/视频生成项目**（文生图/图生视频）→ 必须保存生成结果图片/视频
- **架构图/可视化项目**（SVG/Diagram）→ 必须 capture 渲染后的图

截图处理流程：
```
① browser 截图 → 保存到 /tmp/<project>-screenshot-N.png
② 图片生成结果 → cp/curl 到 /tmp/<project>-output-N.png
③ 批量处理 → /tmp/<project>-screenshots/ 目录
④ 所有截图路径记录到干活 agent 输出末尾
```

**【Windows 截图具体方法】**

```python
import subprocess, time, win32gui
from PIL import ImageGrab

# 找窗口：枚举所有窗口，找到对应标题的 Edge/浏览器
results = []
def enum_handler(hwnd, ctx):
    if win32gui.IsWindowVisible(hwnd):
        title = win32gui.GetWindowText(hwnd)
        if title and '目标关键词' in title:
            rect = win32gui.GetWindowRect(hwnd)
            results.append({'hwnd': hwnd, 'title': title, 'left': rect[0], 'top': rect[1], 'right': rect[2], 'bottom': rect[3]})
win32gui.EnumWindows(enum_handler, None)

# 用 Windows 绝对路径打开 HTML（不用 file:///tmp/）
subprocess.Popen(['powershell', '-Command', f'Start-Process msedge -ArgumentList "C:\\Users\\YourUser\\AppData\\Local\\Temp\\{filename}.html"'])
time.sleep(3)

# 捕获窗口截图
img = ImageGrab.grab(bbox=(w['left'], w['top'], w['right'], w['bottom']))
img.save(f'/tmp/{project}-screenshot-{N}.png')
```

关键：
- Windows 路径用 `C:\Users\YourUser\AppData\Local\Temp\...`，不用 `file:///tmp/...`
- 浏览器打开 HTML 后用 win32gui 枚举窗口找 HWND，再用 ImageGrab 截图

**干活 agent 只调用这个，不读 SKILL.md Step 1：**

> 详见 `references/connector/clone-and-test.md`

**干活 agent 读取 clone-and-test.md，执行里面的命令，记录输出。**

---

### Step 2: 写人味儿 skill 介绍（触发 682）

调用 [682 renwei-writing](skill:renwei-writing) skill，输出人味儿 handoff：

**renwei handoff 格式要求**:
```
一段话开头（场景 + 代价 + 位置）
工具对照表（工具/管什么/什么时候用）
核心原则（≤3 条，每条一句话）
改后状态（实际落地的位置）
```

**内容基于 Step 1 的真实测试结果**，不是猜测。

---

### Step 3: 写 Wiki

用 kdocs-cli 或其他 Wiki API 写到你的知识库：

```
目录: <你的Wiki目录>/loop-go-<YYYY-MM-DD>
文件: <repo-name>-loop-结果.md
```

**必须包含**:
- Step 1 真实测试结果（clone/依赖/验证）
- Step 2 人味儿 skill 介绍
- Step 4 自检验收结果
- Step 5 CE Review 结果
- Step 6 自我复盘结果
- Step 7 handoff 内容
- GitHub URL（超链接格式）

**【Web/图片类项目必须嵌入截图/图片】**

媒体嵌入规范：
```
判断标准：
  - 有 Web UI → 必须截至少 1 张关键页面
  - 有图片输出 → 必须保存并嵌入
  - 有架构图 → 必须 capture 渲染效果

嵌入方式（按优先级）：
  1. data-URI base64（最佳）：小图 < 400KB，otl insert-content 支持
  2. WPS 附件上传：中等图，走 drive upload-attachment → 拿 object_id → 嵌图
  3. 外部 URL（不推荐）

截图命名规范：
  /tmp/<repo>-screenshot-<N>-<描述>.png

图片必须能看见才算成功：
  - 写完 Wiki 后验证 size（> 50KB 说明有内容）
```

---

### Step 4: 对照 631 自检验收

对照 [631 5 组件自检表](skill:loop-engineering-selfcheck-631) 逐项打勾：

| 组件 | 检查项 | 本次结果 | 备注 |
|------|--------|----------|------|
| 组件1 自动化调度 | 有无 cron 钩子？ | ✅/❌ | |
| 组件2 工作树隔离 | 有无 worktree？ | ✅/❌ | |
| 组件3 技能文件 | 新 SKILL.md？ | ✅/❌ | |
| 组件4 连接器 | 用了哪些工具？ | | |
| 组件5 子代理 | 有无 delegate_task？ | ✅/❌ | |
| +1 外部记忆 | Wiki 写了？ | ✅/❌ | |

**不合格项 → 必须补到合格，才算 Loop 完成。**

---

### Step 5: CE Review（强制）

CE 全流程 4 步，每次必须走完，不得跳过：

| 阶段 | 产出 | 说明 |
|------|------|------|
| **Plan** | `STRATEGY.md` | 动手前写，锚定目标/约束/边界 |
| **Work** | 实际产出 | 按规划执行（以上 Step 1-4） |
| **Review** | Review 复盘 | 做对了/做错了/下次改 |
| **Compound** | 更新 Skill/模板/坑日志 | 经验变资产，不过期 |

**Review 栏不填 = 本次任务未完成，不得输出"完成"或"已沉淀"。**

CE 完成后，把 Review 结果写入 Wiki（Step 3 的文档里）。

---

### Step 6: 自我复盘（本 Loop 自检）

**Loop 跑完之后，必须自检本次 loop 自身：**

| # | 复盘项 | 问题 | 填写 |
|:--|:-------|:-----|:-----|
| 1 | clone 真跑了？ | clone 命令实际执行了，还是只读 README 没实际跑？ | ✅/❌ + 原因 |
| 2 | 测试命令真跑了？ | 依赖安装/验证命令实际执行了，还是只看了文档？ | ✅/❌ + 原因 |
| 3 | renwei handoff 写了？ | 写了一份符合 renwei 格式的 handoff 追加到 BD？ | ✅/❌ + 文件路径 |
| 4 | CE Review 栏填了？ | Review 的"做对了/做错了/下次改"三项都填了？ | ✅/❌ + 关键内容 |
| 5 | 自检本身有没有敷衍？ | 填的都是 ✅ 还是真的有具体原因？ | ✅（诚实）/ ❌（敷衍） |
| 6 | 下次改什么？ | 本次 Loop 最大 1 个改进点是什么？ | 具体描述 |

**复盘结论必须写进 Wiki（Step 3 的文档里）。**

---

### Step 7: 写 renwei handoff（追加到 BD）

调用 [682 renwei-writing](skill:renwei-writing)，写一份人味儿 handoff。

---

### Step 8: 主动问 cron + 飞书卡片

完成以上所有步骤后，**主动询问**用户：

```
✅ Loop 完成！

关于后续自动化：
① 需要配 cron 定时跑这个项目吗？（告诉我频率，如"每天"/"每周"）
② 需要配飞书卡片推送吗？（告诉我推给谁/推什么内容）

直接回复我，我来配。
```

**只有用户明确回复"不需要"才跳过此步。**

---

## 3. Loop 完成报告

```markdown
## Loop 完成报告 — <repo-name>

**日期**: YYYY-MM-DD
**GitHub**: [owner/repo](URL)

### 测试结果
- Clone: ✅/❌
- 依赖安装: ✅/❌
- 验证命令: ✅/❌

### 自检验收
- 组件1 自动化调度: ✅/❌
- 组件2 工作树隔离: ✅/❌
- 组件3 技能文件: ✅/❌
- 组件4 连接器: ✅/❌
- 组件5 子代理: ✅/❌
- +1 外部记忆: ✅/❌

### CE Review
- STRATEGY.md: ✅/❌
- Review 栏: 已填/未填
- Compound: ✅/❌

### 自我复盘
- clone 真跑了: ✅/❌
- 测试命令真跑了: ✅/❌
- renwei handoff 写了: ✅/❌
- CE Review 栏填了: ✅/❌
- 自检有没有敷衍: ✅（诚实）/ ❌（敷衍）
- 下次改什么: ...

**结论**: 🟢 Loop 完成 / 🟡 需补项 / 🔴 Loop 中断
```

---

## 4. 失败处理

| 失败阶段 | 处理方式 |
|----------|----------|
| Clone 失败 | 换 GitHub URL / 报网络问题，询问用户 |
| 依赖安装失败 | 记录错误日志，写入 Wiki 继续走 |
| 自检不合格 | 补到合格，不算完成 |
| CE Review 未填 | 必须补，不算完成 |
| 自我复盘敷衍 | 必须重填，不算完成 |

---

## 5. 关联

- 【648】loop-go（父级入口）
- 【631】Loop Engineering 自检
- 【682】renwei-writing（人味儿 skill 介绍 + handoff 写法）
- 【656】Autonomous QA Loop
- 【684】Compound Engineering（CE Review 流程参照）

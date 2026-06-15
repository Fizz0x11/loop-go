# loop-go-exec 连接器：Clone + 测试命令

> 本文件是 Step 1 的**执行细节**，由干活 agent 调用。
> **干活 agent 只调用这个文件，不读 SKILL.md 的 Step 1。**

---

## Clone

```bash
# Workdir: 设置为你的临时工作目录
WORKDIR="~/projects/loop-workdir"

cd "$WORKDIR"
gh repo clone <owner>/<repo> -- --depth=1
# 兜底: git clone <repo-url>
```

---

## 初步测试

### 读取 README

```bash
cat README.md | head -100
```

### 判断项目类型

```
CLI工具  → 检查 bin/ 或 package.json bin:
库       → 检查 src/ 或 lib/
Web      → 检查 index.html 或 vite.config.js
Go项目   → 检查 go.mod
Rust项目 → 检查 Cargo.toml
```

### 依赖安装

```bash
# Node
npm install 2>&1 | tail -20

# Python
pip install -e . 2>&1 | tail -20

# Go
go mod tidy 2>&1 | tail -20

# Rust
cargo check 2>&1 | tail -20
```

### 验证命令（至少跑一个）

```bash
# 根据项目类型选择：
node --version                    # Node
python --version                  # Python
go version                        # Go
rustc --version                  # Rust

# 或者项目自带命令：
npm run build 2>&1 | tail -20
cargo build 2>&1 | tail -20
```

---

## 必须输出的字段

```
project_type: CLI/库/Web/SDK/...
readme_status: ✅完整 / ⚠️残缺 / ❌无
dependency_install: ✅/⚠️/❌ + 错误摘要
verification: ✅/⚠️/❌ + 执行的命令
```

---

## 失败处理

| 失败 | 记录到输出，继续走 |
|------|------------------|
| Clone 失败 | 记录错误，换 `git clone` 兜底 |
| 依赖安装失败 | 记录错误日志，继续跑验证命令 |
| README 缺失 | 标记 ❌，继续 |

---
name: e2e-browser-test
description: >
  端到端浏览器测试。启动后端+前端服务，用 headless 浏览器验证页面加载、
  交互功能、错误处理、toast 弹出等。截图留证。
  触发词: "e2e测试", "浏览器测试", "端到端测试", "browser test", "e2e test"
allowed-tools: Bash, Read, Glob, Grep
argument-hint: [--url <base_url>] [--pages <comma-separated>]
---

# 端到端浏览器测试

## Step 1: 启动服务

```bash
# 检测项目类型并启动
# FastAPI + Vite 项目
cd backend && .venv/bin/python3 run.py &>/tmp/backend.log &
cd frontend && npx vite --port 5174 &>/tmp/frontend.log &
sleep 4

# 确认服务就绪
curl -s -o /dev/null -w "%{http_code}" http://localhost:8001/api/settings/
curl -s -o /dev/null -w "%{http_code}" http://localhost:5174/
```

## Step 2: 浏览器测试

使用 gstack browse（headless Chromium）执行：

```bash
B=~/.claude/skills/gstack/browse/dist/browse

# 导航到页面
$B goto <url>

# 截图留证
$B screenshot /tmp/e2e-<page>.png

# 检查 JS 错误
$B console --errors

# 检查网络请求
$B network

# 验证元素存在
$B is visible "<selector>"

# 交互测试
$B snapshot -i          # 列出可交互元素
$B click @e<N>          # 点击
$B fill @e<N> "value"   # 填写
$B snapshot -D           # 对比变化
```

## Step 3: 核心验证清单

每个 PR 的 E2E 测试必须覆盖：

1. **首页加载** — 页面渲染、API 请求 200、无 JS 错误
2. **核心功能页** — PR 涉及的页面全部打开验证
3. **错误处理** — 触发 API 错误，验证 toast 弹出而非 alert
4. **Console 错误** — 过滤掉 React Router warnings，其余必须为零
5. **截图留证** — 每个测试页面截图，用 Read 工具展示给用户

## Step 4: 输出报告

```markdown
### E2E 浏览器测试

| 页面 | 结果 | 说明 |
|------|------|------|
| 首页 | ✅/❌ | ... |
| 设置页 | ✅/❌ | ... |
| ... | ... | ... |
| Console 错误 | ✅ 零 JS 错误 | |
```

## ⚠️ 规则

- 必须真正启动服务，不能只跑单元测试冒充 E2E
- 截图必须用 Read 工具展示给用户看
- Console 错误中 React Router warnings 和 404 polling 可忽略
- 测试完成后不需要手动停止服务（后台进程）

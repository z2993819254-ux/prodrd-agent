---
name: api-test
description: >
  后端 API 端点测试。用 curl 验证 API 端点注册、请求/响应格式、错误处理、
  状态码正确性。适用于 REST API 验证。
  触发词: "api测试", "接口测试", "curl测试", "api test", "test api"
allowed-tools: Bash, Read, Grep
argument-hint: <endpoint_or_pr_number> [--base-url <url>]
---

# API 端点测试

## Step 1: 确认服务运行

```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:8001/api/settings/
```

如果服务未运行，先启动：
```bash
cd backend && .venv/bin/python3 run.py &>/tmp/backend.log &
sleep 3
```

## Step 2: 读取 PR diff 确定新增/修改的端点

```bash
gh pr diff <number> | grep -E "@router\.(get|post|put|delete|patch)"
```

## Step 3: 逐个端点测试

### 正常请求
```bash
# GET 端点
curl -s http://localhost:8001/api/<endpoint> | head -c 200

# POST 端点
curl -s -X POST http://localhost:8001/api/<endpoint> \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}' | head -c 200
```

### 错误处理
```bash
# 404 — 不存在的资源
curl -s -X GET http://localhost:8001/api/<endpoint>/nonexistent \
  -w "\nHTTP %{http_code}"

# 422 — 无效参数
curl -s -X POST http://localhost:8001/api/<endpoint> \
  -H "Content-Type: application/json" \
  -d '{}' -w "\nHTTP %{http_code}"
```

## Step 4: 验证清单

- [ ] 端点已注册（不返回 405 Method Not Allowed）
- [ ] 正常请求返回正确状态码（200/201）
- [ ] 响应格式正确（JSON 结构符合预期）
- [ ] 错误请求返回合理错误码（400/404/422）
- [ ] 错误响应包含 `detail` 字段

## Step 5: 输出报告

```markdown
### API 端点测试

| 端点 | 方法 | 场景 | 状态码 | 结果 |
|------|------|------|--------|------|
| /api/xxx | POST | 正常请求 | 200 | ✅ |
| /api/xxx | POST | 空 body | 422 | ✅ |
| /api/xxx/999 | GET | 不存在 | 404 | ✅ |
```

## ⚠️ 规则

- 必须用真实 curl 请求，不是单元测试的 TestClient
- 每个新增端点至少测正常 + 错误两个场景
- 响应内容截取前 200 字符记录

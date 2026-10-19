# workspace

用于验证 CI 与 Anthropic API 的最小仓库。

CI 工作流（`.github/workflows/ci.yml`）中的 smoke job 会真实调用
Anthropic Messages API 完成验证，运行前需要在本仓库配置 Actions secret：

- `ANTHROPIC_API_KEY`：Anthropic API 密钥（`sk-ant-...`）

设置为空或缺失时，smoke job 会直接失败。

## 在 GitHub 上配置 secret

Repo → Settings → Secrets and variables → Actions → New repository secret
（名称 `ANTHROPIC_API_KEY`）。

## 本地运行 smoke

```bash
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-4-5","max_tokens":16,"messages":[{"role":"user","content":"Reply with exactly: pong"}]}'
```

## CI 行为

- 每次 push 到 `main`（或手动触发）都会运行 smoke job；
- job 先检查 secret 是否存在，再用它对 Anthropic Messages API 发起真实请求；
- 瞬时错误（5xx/429）会自动重试 3 次，连续失败或返回空回复则判定 CI 失败。
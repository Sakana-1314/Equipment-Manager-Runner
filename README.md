# Equipment-Manager-Runner

[Equipment-Manager](https://github.com/Sakana-1314/Equipment-Manager)（私有仓库）的 CI 代跑仓库。
主仓库 Actions 额度有限，测试与镜像构建放在本公开仓库执行（ubuntu-24.04，公开仓库免费）。

## 结构

- 主仓库只放 `trigger.yml`：PR/push 事件经 `repository_dispatch` 转发到本仓库（需要主仓库变量 `RUNNER_REPO` 与 Secret `PAT`）。
- 本仓库通过 `secrets.PAT` 签出私有主仓库对应 commit；测试后把状态回写到主仓库 PR 当前 head（context：`测试通过检查（Runner）`），并以单条固定评论汇总门禁。

## 工作流

| 文件 | 触发 | 内容 |
| --- | --- | --- |
| `test.yml` | `repository_dispatch(equipment_pr)` | 服务端（gofmt/vet/全量测试，embedded-postgres）+ 前端（typecheck/lint/单测/构建），按路径过滤；聚合回写门禁状态 |
| `e2e-tests.yml` | `repository_dispatch(equipment_push)` | Playwright 浏览器核心闭环（按路径过滤，可手动触发） |
| `deploy.yml` | 仅手动 `workflow_dispatch` | 构建前后端镜像 → docker save/load 经 SSH 送达测试服务器 → compose up -d → 健康检查 |

## 机密

| 名称 | 用途 |
| --- | --- |
| `PAT` | 签出私有主仓库、回写 commit status 与 PR 评论 |
| `SSH_HOST` / `SSH_PORT` / `SSH_USER` / `SSH_PRIVATE_KEY` | 发布到测试服务器（仅 deploy.yml 使用） |

安全红线：本仓库公开，deploy 类工作流只允许 `workflow_dispatch` 手动触发，不配置
push / pull_request 触发，避免机密暴露给不可信代码。

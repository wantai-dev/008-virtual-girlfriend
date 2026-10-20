# wantai-dev/008-virtual-girlfriend 开发集成与分层 CI

开发成果统一汇总到 `dev`，不为功能小改动创建 PR；集成 PR 只走 `dev → main`。

日常本地按实际变更与影响路径选择检查。远端只在 `dev → main` 集成 PR 阶段手动触发 `.github/workflows/integration-ci.yml`；开 PR、push 和定时器不触发远端 CI。生产发布另行手动执行，并保留已有授权、输入与环境守卫。

`.development-ci.json` 是检查清单。`changed` 按 `checks[].paths` 选择；`full` 执行全部已声明 checks。命令按 argv 执行，不拼接用户输入到 shell。

每天 05:00（Asia/Taipei）完整检查由本机统一调度器串行执行，不在 GitHub Actions 新增回归 schedule。执行机器、隔离检出、基线 SHA、命令和日志以本机调度登记为准；本文不表示定时任务已经启用。

迁移来源：commit `b444e4afbf2da97c7e1c5f3be9dd91e61486043c`，tree `5dba1b3f1c195ce6940568cc6748ddc3c1174a70`。快照用于评审，不表示已应用远端或已通过产品回归。

## 安装

- `.`：`npm install --ignore-scripts --no-audit --no-fund`

## 检查命令

| ID | 工作目录 | 命令 |
| --- | --- | --- |
| eslint | `.` | `npm run lint` |
| typescript | `.` | `node node_modules/typescript/bin/tsc --noEmit` |

## 能力与边界

- 命令来源：package.json lint与已声明TypeScript；直接调用本地tsc做不生成产物的静态检查。
- 当前远端tree无依赖锁文件，npm install不能保证可复现；后续应单独审核并建立锁文件。
- 根目录test-*.js未证实为隔离离线测试，不执行；不运行真实LLM、Supabase、唤醒、登录或资料写入。未纳入Next构建和功能端到端回归。

只运行列明且不依赖生产凭据的离线检查。失败应报告原始失败；真实 API、账号、数据写入、采集、部署和外部接受度需独立验收。

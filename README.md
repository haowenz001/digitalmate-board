# DigitalMate 单仓看板

> 数据源：`openJiuwen-ai/DigitalMate`（单仓深视图：PR / Issue / CI / 测试 / 覆盖率 / 发布）。
> 看板地址：<https://haowenz001.github.io/digitalmate-board/>
> 模板上游：[`agent-CI-Infra/dashboard/single-repo`](https://github.com/agent-CI-Infra/dashboard/tree/main/single-repo)（操作手册见 [`CI-workflow/AGENT.md`](https://github.com/agent-CI-Infra/CI-workflow/blob/main/AGENT.md)）。

## 当前状态

| 项 | 状态 |
| --- | --- |
| 看板仓 | 本仓（public，Pages 由 Actions 部署） |
| 源仓 | `openJiuwen-ai/DigitalMate`，**目前是 private** |
| 采集 | **已暂停**：`Collect dashboard data (PAT)` 的 job 条件为 `vars.BOARD_COLLECT == 'true'` |
| 公开数据 | 未发布任何源仓数据；`site/` 只有静态壳，没有 `site/data/snapshot.json` |

私仓数据不能进公开 Pages（模板会直接拒绝），所以在源仓转 public 之前，这里的采集保持关闭，避免每天定时跑红。

## 启用步骤（源仓转 public 之后）

1. 确认源仓可见性已是 public：

   ```bash
   curl -s https://api.github.com/repos/openJiuwen-ai/DigitalMate | jq -r .visibility
   ```

2. 本仓 **Settings → Secrets and variables → Actions → Variables** 新建 `BOARD_COLLECT = true`。
3. **Actions → Collect dashboard data (PAT) → Run workflow**（首次不要勾 `--incremental`，让 `.sync/` 从零建立）。
4. 等 `Deploy dashboard Pages` 成功，打开 <https://haowenz001.github.io/digitalmate-board/>。

源仓侧还需要先接入门禁并规范测试产物名，否则 CI 页「质量门禁」卡片和测试卡片是空的，见
[`docs/digitalmate-integration.md`](docs/digitalmate-integration.md)。

## 本地预览（不发布）

```bash
cd digitalmate-board
GSB_ALLOW_PRIVATE=1 GITHUB_TOKEN=... \
  python3 publish.py --repo openJiuwen-ai/DigitalMate --incremental
python3 server.py          # http://127.0.0.1:8790，仅回环地址
```

`GSB_ALLOW_PRIVATE=1` 只允许生成本机 `dist/`；`publish.py --publish-repo` 仍会拒绝私有源仓。
本地生成的 `dist/`、`.sync/`、`.cache/` 已在 `.gitignore` 中，且不得提交带私仓数据的快照。

## 变更与上游同步

- 看板配置唯一入口是 [`board-config.json`](board-config.json)（源仓、workflow 名称正则、分支线、部署映射）。
- 上游模板更新时用 `git diff` 对照 `agent-CI-Infra/dashboard/single-repo`，不要把源仓自己的 `.sync/`、`site/data/` 覆盖掉。
- 本仓相对模板只有两处差异：`board-config.json` 的内容，以及 `collect-selfhosted.yml` 增加了 `BOARD_COLLECT` 开关。

## 文档

- [模板原 README](docs/template-readme.md)
- [接入说明（源仓侧要做什么）](docs/digitalmate-integration.md)
- [测试报告契约](docs/test-reports.md)
- [增量同步与历史](docs/incremental-history.md)
- [页面与交互](docs/dashboard-pages.md)
- [分支线](docs/branch-lines.md)

## 验证

```bash
PYTHONDONTWRITEBYTECODE=1 python3 -m unittest discover -s tests -v
node test/sync-e2e.mjs
```

`Dashboard checks` 工作流在每次推送（`site/**`、`.sync/**` 除外）时自动跑上面的检查。

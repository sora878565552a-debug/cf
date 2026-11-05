# CF 优选 IP · 自动发布仓库

这个目录只放**供上游面板读取的文本文件**，由 `D:\cfip\auto.mjs` 每 6 小时自动 commit + push。

## 文件

| 文件 | 内容 | 用途 |
|---|---|---|
| `best.txt` | 每行一个优选 IP | 面板「自定义优选」里填它的 raw 地址 |
| `status.json` | 最近一次扫描摘要（延迟/机房/得分） | 给自己排查用 |

## 初始化（只做一次）

在 `D:\cfip\publish` 目录里执行：

```powershell
cd D:\cfip\publish
git init -b main
git config user.name  "cfip-bot"
git config user.email "cfip@local"
git add -A
git commit -m "init: cfip publish repo"
git remote add origin <你的仓库地址>      # 例: https://github.com/you/cfip-data.git
git push -u origin main
```

> 仓库建 **public**（私有仓库的 raw 地址带 token，面板取不到）。
> 也可以放 Gitee：`https://gitee.com/<user>/<repo>.git`

## 面板里填什么

### GitHub 仓库

```
https://raw.githubusercontent.com/<user>/<repo>/main/best.txt
```

⚠️ `raw.githubusercontent.com` 在国内经常被墙。**推荐用 jsDelivr 镜像**：

```
https://cdn.jsdelivr.net/gh/<user>/<repo>@main/best.txt
```

### Gitee 仓库

```
https://gitee.com/<user>/<repo>/raw/main/best.txt
```

## 全自动链路

```
定时任务（每 6h）
  → D:\cfip\auto.mjs 扫描
  → 写 publish\best.txt + status.json
  → git add / commit / push
  → 面板拉 raw 地址（下一步在面板里点「开始优选」或设自动）
  → worker /sub 输出更新
  → Clash 更新订阅 → 生效
```

## 注意

- `best.txt` 只有 5 行 IP，仓库体积几乎不增长（每次 commit 只改 5 行）
- 如果希望面板立刻反映，面板那边需要重新生成一次优选；Clash 侧把订阅自动更新设成 6h
- 推送失败不会影响本地 `out\clash.yaml`，只会在日志里记一条 `上传: 失败 …`

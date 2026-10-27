# jhentai-altsource（方案 2：自建 AltSource feed，不依赖上游改任何东西）

上游 `jiangtian616/JHenTai` 的 AltSource feed 由它自己 release 工作流的最后两步生成，
存在两类已被证实的故障（v8.0.18 掉更、tag/pubspec 错位导致条目内部不一致）。
本仓库完全不依赖上游 CI 的正确性：**以官方 Releases 本身为事实源**，定时从
`releases/latest` 的 `JHenTai_*.ipa` asset（文件名必然与实际二进制一致）重建 feed。

## 使用

1. 新建仓库（如 `<you>/jhentai-altsource`），放入本目录两个文件：
   - `altsource/AltSource.json`（种子文件，首次运行就会被正确数据覆盖——种子仍是上游的 8.0.17 旧条目）
   - `.github/workflows/refresh.yml`
2. push 后到 Actions 手动 Run workflow 一次，生成首个正确条目（会得到 `8.0.18+341`、
   `downloadURL`/`size` 与真实 asset 一致的条目——顺带修掉上游 feed 里 `buildVersion: "339"` 的错位）。
3. 在 AltStore / Sideloader 里添加 source：
   - GitHub 直链：`https://raw.githubusercontent.com/<you>/jhentai-altsource/master/altsource/AltSource.json`
   - 国内网络更稳的 jsDelivr 镜像：`https://cdn.jsdelivr.net/gh/<you>/jhentai-altsource@master/altsource/AltSource.json`

## 从零推送（无 gh CLI 时）

先在 https://github.com/new 创建**空仓库** `jhentai-altsource`（不要勾选自动 README/.gitignore），然后在本目录：

```bash
git init -b master
git add -A
git commit -m "init: 自建 JHenTai AltSource feed（以官方 release asset 为事实源）"
git remote add origin https://github.com/saki3525/jhentai-altsource.git
git push -u origin master
```

推送后到 Actions 手动 Run workflow 一次，生成首个正确条目（8.0.18+341）。

## 行为

- 每天 03:17 UTC 定时刷新 + 手动触发（workflow_dispatch / repository_dispatch）。
- 静态字段（图标、截图、bundleID 等）每次从上游仓库的 `altsource/AltSource.json` 拉取作为 base，
  上游改了展示字段会自动跟进；拉不到时退回仓库内副本，feed 不会坏。
- 版本条目只从 release asset 名解析：`JHenTai_<version>+<build>.ipa` → version/build；
  `downloadURL`、`size`、发布日期、更新说明分别取 asset 的 `browser_download_url`、`size`、
  `published_at`、release body。**不读 pubspec、不解析 tag 名**，因此对上游任何
  tag/pubspec 错位、tag 不带 `+N`、publish 步骤坏掉都免疫。
- 无 IPA asset 的 release：跳过并保持现有 feed；内容无变化时不产生提交（幂等）。

## 边界与说明

- IPA 的签名/可信度是上游属性，本 feed 只是指向官方 release asset 的另一种“目录”，不改变安装体验与签名要求。
- feed 新鲜度 = cron 频率（上游发版频率约每月一次，daily 绰绰有余；急的话手动触发）。
- 上游官方修复后可直接使用官方 source；本仓库可留作镜像（jsDelivr 对国内友好）或归档。

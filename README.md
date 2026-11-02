# LYTransChat 更新分发

这个仓库**只放安装包和更新元数据**，不含任何源码。它存在的唯一目的，是让已安装的 LYTransChat 客户端能检查并下载新版本。

## 客户端怎么用

客户端通过 `electron-updater` 读取本仓库最新 Release 的 `latest.yml`：

1. 请求 `releases/latest`，拿到最新版本号
2. 下载 `latest.yml`，里面写明版本、安装包文件名、sha512、大小
3. 比对本地版本，有新版才下载安装包（配合 `.blockmap` 做差分，只取变化的块）
4. 校验 sha512 后启动安装程序

## 每次发版的步骤

```bash
# 1. 升 app/package.json 的 version 并打包
cd app && npm run dist

# 2. 发 Release（版本号与 tag 必须一致）
#    ⚠️ -R 不能省：gh 默认用当前目录的 git remote，而当前目录是私有源码仓库
#    LYTransChat，漏掉 -R 会把安装包发进源码仓库（2026-11-02 踩过一次）。
cd .. && gh release create v<版本> -R tradeshift-code/LYTransChat-releases \
  "app/dist/LYTransChat-<版本>-setup.exe" \
  "app/dist/LYTransChat-<版本>-setup.exe.blockmap" \
  "app/dist/latest.yml" \
  --title "LYTransChat <版本>" --notes-file <说明文件>
```

发错了怎么救：

```bash
gh release delete v<版本> -R tradeshift-code/LYTransChat --yes --cleanup-tag
```

三个文件缺一不可：

- `setup.exe` —— 安装包本体
- `.blockmap` —— 差分下载索引，缺了客户端只能全量下载
- `latest.yml` —— 版本清单，缺了客户端认为没有更新

## 注意

- Release 必须是**已发布**状态（不能是 draft 或 prerelease），否则客户端读不到。
- `latest.yml` 里的文件名必须与实际上传的 asset 完全一致，改名就下载失败。
- 便携版（portable）不走自动更新，不需要上传它的安装包。
- 安装包未做代码签名，首次安装 Windows 会提示「已保护你的电脑」，选「更多信息 → 仍要运行」即可。

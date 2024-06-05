

### 强制覆盖本地代码

```bash
git fetch --all                  # 拉取所有更新，不同步
git reset --hard origin/main     # 本地代码同步线上最新版本（会覆盖本地所有与远程仓库上同名的文件）
git pull origin main             # 重新拉取

git fetch --all && git reset --hard origin/main && git pull origin main
```


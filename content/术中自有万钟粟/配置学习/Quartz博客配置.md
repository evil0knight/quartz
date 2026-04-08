# Quartz 博客配置

> 目标：笔记仓库 `D:\Basic_learning_record` 推送到 GitHub 后，博客自动更新。

---

## 原理

```
本地写笔记 → git push → GitHub Actions 自动构建 Quartz → 博客更新
```

---

## 步骤

### 第一步：Fork Quartz 仓库

1. 打开 https://github.com/jackyzha0/quartz
2. 点击右上角 **Fork**，Fork 到自己的 GitHub 账号
3. 仓库名可改为 `blog` 或保持 `quartz`

---

### 第二步：本地克隆 Quartz

在想要放Quartz的地方创文件夹然后执行这些,比如D:\quartz>

```bash
git clone https://github.com/<你的用户名>/quartz.git
cd quartz
npm install
```

---

### 第三步：配置内容路径

Quartz 默认读取 `content/` 文件夹作为笔记来源。

把你的笔记仓库链接进来，不需要复制文件：

```bash
# Windows（管理员 PowerShell）
New-Item -ItemType Junction -Path "content" -Target "D:\Basic_learning_record"
```

---

### 第四步：本地预览

```bash
npx quartz build --serve
```

笔记不能有HTML格式比如

<details>                                                                                                                                                                  
  <summary><h3 style="display:inline">🔴章节名</h3></summary>                                                                                                                
  </details>

里的<h3 style="display:inline">

如果 Quartz 生成 OG 图片时遇到了不支持的 emoji。最简单的解决方案是直接关掉 OG 图片生成。

```
(D:/quartz/quartz.config.ts)
    89        Plugin.Favicon(),
    90        Plugin.NotFoundPage(),
    91        // Comment out CustomOgImages to speed up build time
    92 -      Plugin.CustomOgImages(),
    92 +      // Plugin.CustomOgImages(),
    93      ],
    94    },
    95  }
```

改正,然后⬇️

浏览器打开 http://localhost:8080 预览效果。

如果404,说明缺少一个index.md,新建一个

---

### 第五步：部署到 GitHub Pages

1. 新建一个PS,cd D:\quartz,推送 Quartz 仓库到 GitHub：

   ```bash
   git add .
   git commit -m "init quartz blog"
   git push
   ```
2. 进入仓库 **Settings → Pages**，Source 选择 `GitHub Actions`
3. Quartz 自带 `.github/workflows/deploy.yml`，会自动构建并发布

---

### 第六步：让笔记更新触发博客更新

在笔记仓库 `D:\Basic_learning_record` 中添加 GitHub Actions，每次 push 后触发 Quartz 仓库重新构建：

新建文件 `.github/workflows/trigger-blog.yml`：

```yaml
name: Trigger Blog Update

on:
  push:
    branches: [main]

jobs:
  trigger:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger Quartz rebuild
        uses: peter-evans/repository-dispatch@v3
        with:
          token: ${{ secrets.BLOG_TRIGGER_TOKEN }}
          repository: <你的用户名>/quartz
          event-type: notes-updated
```

同时在 Quartz 仓库的 `deploy.yml` 中添加触发条件：

```yaml
on:
  push:
    branches: [main]
  repository_dispatch:
    types: [notes-updated]
```

---

### 第七步：配置 Token

1. GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens
2. 新建 Token，权限选 `Contents: Read and Write`（针对 quartz 仓库）
3. 复制 Token
4. 在笔记仓库 Settings → Secrets → Actions 中新建 `BLOG_TRIGGER_TOKEN`，粘贴 Token

---

## 完成后的效果

每次在本地写完笔记执行：

```bash
git push
```

博客在 1~2 分钟内自动更新。

---

## 注意事项

- Quartz 对文件名中的中文支持良好，但 URL 会被编码
- 不想公开的笔记可在 `quartz.config.ts` 中配置 `ignorePatterns` 排除
- 首次部署后博客地址为：`https://<你的用户名>.github.io/quartz`

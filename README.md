# edgetunnel 自托管后台页面

> 当前部署地址：<https://ko-kuverse.github.io/edgetunnel-pages/>
> Worker 变量：`PAGES_BASE = https://ko-kuverse.github.io/edgetunnel-pages`

这个仓库只做一件事：给 edgetunnel 的 Cloudflare Worker 提供 4 个静态页面，
替代原来运行时从 `edt-pages.github.io` 拉取的公共页面。

| 目录 | Worker 中的路径 | 用途 |
| --- | --- | --- |
| `admin/index.html` | `/admin` | 管理后台 |
| `login/index.html` | `/login` | 登录页 |
| `noADMIN/index.html` | `/noADMIN` | 未配置 `ADMIN` 变量时的提示页 |
| `noKV/index.html` | `/noKV` | 未绑定 KV 时的提示页 |

页面本身是自包含的（HTML + 内联 CSS/JS），不需要构建。Worker 会在收到对应请求时
`fetch` 这个仓库的 Pages 地址，然后把 HTML 以**你自己的域名**返回给浏览器。

## 部署步骤

### 1. 在 GitHub 创建仓库

- 仓库名建议 `edgetunnel-pages`，**Public**（GitHub Pages 免费版只支持公开仓库；
  这些页面本来也是公开的）。
- 不要勾选 “Add a README file”，避免和本仓库的文件冲突。
- 把这些文件原样上传到仓库根目录（保持 `admin/`、`login/`、`noADMIN/`、`noKV/` 的目录结构）。

### 2. 开启 GitHub Pages

仓库 → **Settings** → **Pages**：

- Source：**Deploy from a branch**
- Branch：`main`，目录：`/ (root)`
- 点击 **Save**，等待 1–2 分钟。

访问 `https://<你的用户名>.github.io/edgetunnel-pages/`，能看到说明页就成功了。

> 如果把这个仓库命名为 `<你的用户名>.github.io`，地址就是 `https://<你的用户名>.github.io`。
> 也可以在 Pages 设置里绑定自定义域名，后面用自定义域名即可。

### 3. 让 Worker 使用这个地址

Cloudflare Worker → **设置** → **变量和机密** → 添加普通变量：

```
PAGES_BASE = https://<你的用户名>.github.io/edgetunnel-pages
```

末尾不要带 `/`（带了也会被自动去掉）。保存后**重新部署**新版 `_worker.js`
（这次改动需要新代码才会读取 `PAGES_BASE`）。

### 4. 验证

1. 打开 `https://<你的 Worker 域名>/login`，应看到登录页。
2. 输入 `ADMIN` 密码，应进入 `/admin`。
3. 在浏览器开发者工具的 Network 里，`login` / `admin` 这两个文档请求仍然指向你的
   Worker 域名；Worker 内部会去拉 `PAGES_BASE`。
4. 如果配置缺失，访问 `/noADMIN`、`/noKV` 对应的提示页应来自你的仓库。

## 以后怎么更新

直接改本仓库文件并提交即可，GitHub Pages 会自动重新发布；**不需要重新部署 Worker**。

如果要跟随上游 UI 更新，可以从
[EDT-Pages/EDT-Pages.github.io](https://github.com/EDT-Pages/EDT-Pages.github.io)
重新下载对应 `index.html` 覆盖，提交前自己 diff 一遍——这等于把“信任上游”
变成了一次显式的人工审查。

## 安全建议

- 这个仓库的页面会在你的管理员会话里运行，能调用 `/admin/*` 接口。
  请开启 GitHub 两步验证，不要随意添加协作者，保护 `main` 分支。
- 页面里仍然有几处客户端第三方依赖：Google Fonts、jsDelivr（Leaflet/二维码）、
  以及部分可选功能用到的 IP/测速 API。它们不经过你的 Worker，但会在管理员浏览器里
  执行/请求；网络受限时后台的核心功能通常仍可用。需要完全离线化时可以再把这些资源
  本地化（这是后续可选项）。
- `admin/index.html` 的“汇聚订阅 → 案例”里有一条写死的演示节点
  （`...@edt-pages.github.io...`），只是帮助说明里的示例文本，不是运行时请求地址。
- 如果你不想依赖 GitHub，这整个目录也可以直接上传到 **Cloudflare Pages**，
  然后把 `PAGES_BASE` 设为 `https://<项目>.pages.dev`，效果相同。

## 来源与许可

页面来自 [EDT-Pages/EDT-Pages.github.io](https://github.com/EDT-Pages/EDT-Pages.github.io)，
是 [cmliu/edgetunnel](https://github.com/cmliu/edgetunnel) 项目的配套前端，遵循 **GPL-2.0**（见 [LICENSE](LICENSE)）。

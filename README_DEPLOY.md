# 青野玄冬号 App 在线网页部署说明

这个文件夹已经整理成静态网页项目，入口文件是 `index.html`。不需要后端、不需要数据库，可以直接部署到 GitHub Pages、Vercel 或 Netlify。

## 方案一：GitHub Pages

1. 登录 GitHub，新建一个仓库，例如：`qingye-robot-app`。
2. 把本文件夹里的全部文件上传到仓库根目录，必须保证 `index.html` 在根目录。
3. 进入仓库 `Settings` → `Pages`。
4. 在 `Build and deployment` 中选择 `Deploy from a branch`。
5. Branch 选择 `main`，目录选择 `/root`，保存。
6. 等待几十秒到几分钟，GitHub 会生成一个公开网页地址。

常见地址格式：

```text
https://你的GitHub用户名.github.io/qingye-robot-app/
```

## 方案二：Vercel

1. 登录 Vercel。
2. 选择 `Add New Project`。
3. 导入你的 GitHub 仓库。
4. Framework Preset 选择 `Other` 或保持默认静态项目。
5. Build Command 留空，Output Directory 留空或填 `.`。
6. 点击 Deploy。

部署后会得到类似：

```text
https://qingye-robot-app.vercel.app/
```

## 方案三：Netlify 拖拽部署

1. 登录 Netlify。
2. 进入 Deploy 页面。
3. 直接把本文件夹或压缩包拖进去。
4. Netlify 会自动发布静态网页。

部署后会得到类似：

```text
https://qingye-robot-app.netlify.app/
```

## 注意事项

- 当前版本是网页端 App 原型，浏览器打开即可演示。
- 在线部署后，手机、平板、电脑都可以访问。
- 若后期要真正连接机器人，机器人端 WebSocket 地址必须能被网页访问。
- 如果网页是 HTTPS，机器人通信最好也升级为安全连接，例如 `wss://`，否则部分浏览器可能拦截连接。
- 实机控制建议加入登录账号、设备绑定、Token 鉴权和硬件急停保护。

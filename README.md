# OING H3 Image Transit

面向 MiniMax H3 的公开图片中转仓库。它只负责把需要给 H3 Worker 读取的参考图变成稳定的公网 HTTPS 图片地址；**不存储 H3 / AutoDL API Key、GitHub Secrets 或任何私密凭据**。

## 已验证链路

`ChatGPT / 分镜图片 → 本仓库 images/ → raw.githubusercontent.com HTTPS 直链 → production-os 私有 H3 队列 → AutoDL MiniMax H3`

2026-09-11 已真实完成一次端到端图生视频测试：H3 成功读取本仓库 Raw 图片直链，单次 POST 完成生成。

## 目录约定

- `incoming/<name>.jpg.b64`：临时传输载荷；每次只提交 1 个新文件。
- `images/<name>.jpg`：Actions 解码后发布的真正图片文件。
- `.github/workflows/decode-image.yml`：自动发布工作流；成功发布后自动清理对应 `incoming/*.b64`。

支持 `.jpg` / `.jpeg` / `.png` / `.webp`。

## H3 图片直链格式

```text
https://raw.githubusercontent.com/suwujin-code/oing-h3-image-transit/main/images/<filename>
```

示例：

```text
https://raw.githubusercontent.com/suwujin-code/oing-h3-image-transit/main/images/h3-test-frame-004.jpg
```

H3 请求中作为 `ref_image_0..8`，Prompt 用 `<Picture 1>..<Picture 9>` 对应引用。

## 生产规则

1. 本仓库保持 Public，仅放允许公开访问的生成素材；客户机密、人像隐私素材不得放入这里。
2. H3 Secret 仍只保存在私有仓库 `suwujin-code/production-os` 的 GitHub Actions Secret 中；本仓库绝不复制 Secret。
3. 正式镜头优先上传原始高质量 JPG/PNG；测试压缩图只用于链路验证，不作为最终画质输入。
4. H3 批量生成严格逐镜串行：上一镜头取得明确终态后再提交下一镜头，避免 concurrency 丢任务和重复付费。
5. 每个 H3 request_id 只允许一次 POST；失败后先判断 reference transport / provider / prompt，再决定是否创建新请求。
6. 对需要严格可复现的项目，建议将引用固定到该图片发布后的 Git commit，而不是长期依赖可变的 `main`。

## 绑定位置

真正的视频生成逻辑不放在本 Public 仓库。执行端位于：

- 私有仓库：`suwujin-code/production-os`
- 分支：`automation/h3-autodl`
- 技能：`.agents/skills/minimax-h3-image-audio-15s/SKILL.md`
- 工作流：`.github/workflows/h3-autodl-queue.yml`

因此本仓库是 **Asset Transport Layer**，`production-os` 是 **Generation / Control Plane**。
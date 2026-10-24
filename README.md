# HT2 (RK3528A) Ubuntu 22.04 自用镜像

把 Hinlink HT2（RK3528A）电视盒子变成一台能用的轻量 Linux 设备。

## 这是什么

在厂商提供的镜像（Ubuntu 22.04 + 厂商 5.10.160 内核）基础上，借助 AI 完成的一系列适配工作的产物：

- **从零实现了一块 VA-API → Rockchip MPP 视频解码转译驱动**（HEVC/VP9 硬解，逐帧像素级验证），mpv/ffmpeg 直接可用，1080p HEVC 播放 CPU 占用降低约 70%
- 自编译 ffmpeg（rkmpp）、mpv 0.37、yt-dlp 工具链，`play <网址>` 一条命令边下边播
- Mali-450 GPU 内核驱动 + 官方 r7p0 用户态库就位（GLES2 应用可用）
- XFCE 轻桌面 + VNC（无显示器自动 1280x720）+ 中文字体输入法
- 首次开机自动重生成 SSH 主机密钥与机器标识

## 基本信息

| 项目 | 值 |
|---|---|
| 默认账号 | `ht2` / `ht2`（SSH、VNC、桌面通用，**刷完第一件事改密码**）|
| VNC | `IP:5900`，密码 `ht2` |
| 镜像 | 完整 58.2GB eMMC 整盘镜像，gzip 压缩约 3.7GB，分 3 卷上传 |

## 下载与合并

Release 里的三个分卷下载到同一目录后合并：

```bash
# Linux / macOS
cat ht2-vpu-ubuntu2204-xfce-v1.0.img.gz.part* > ht2-vpu-ubuntu2204-xfce-v1.0.img.gz

# Windows (cmd)
copy /b ht2-vpu-ubuntu2204-xfce-v1.0.img.gz.part-a + ht2-vpu-ubuntu2204-xfce-v1.0.img.gz.part-b + ht2-vpu-ubuntu2204-xfce-v1.0.img.gz.part-c ht2-vpu-ubuntu2204-xfce-v1.0.img.gz
```

校验 SHA256 后解压得到 `.img`。

## 刷机

**RKDevTool 线刷**：HT2 断电 → 按住复位孔插 USB 线连电脑 → RKDevTool 识别 LOADER 设备 → "下载镜像"页勾选"按地址写" → 添加一项（名称随意，如 `system`），地址填 `0x0`，选解压出的 `.img` → 执行。镜像自带完整引导链（分区表/IDB/u-boot 全在里面），**无需再单独烧 Loader**，完成后设备自动重启。

**Linux dd**：`gunzip -c xxx.img.gz | sudo dd of=/dev/sdX bs=4M conv=fsync`

## 已知缺陷（硬件层面，改不了）

1. **H.264 硬解是坏的**——RK3528 的 VPU 固件缺陷（输出花屏），系统会自动回退软解。1080p H.264 软解可流畅，4K H.264 放不动。片源尽量选 HEVC
2. **GPU 没有开源 3D 驱动**——Mali-450 只有 GLES2 专有库，浏览器 WebGL 是软件渲染。桌面 2D 纯 CPU，够用但别指望丝滑

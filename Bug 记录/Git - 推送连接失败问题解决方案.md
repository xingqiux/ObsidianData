
>今天在提交 Git 的时候遇到无法连接到 GitHub 服务器443端口，主要是因为国内网络情况或者其他的问题，以下是我尝试过的几种方案

## 1. 配置Git代理

如果可以的话，第一步尝试是否是没有科学上网，使用 *系统代理*  模式，

## 2. 切换为SSH连接方式

SSH连接通常比HTTPS更稳定：

```bash
# 在远程仓库找到 git@... 这样的链接
# 修改为SSH URL
git remote set-url origin git@github.com:xingqiux/aurain_admin_spring.git

# 确保已配置SSH密钥，如果没有配置 SSH 密钥可以先去了解在使用此方法
```

## 3. 使用GitHub镜像或加速服务

修改hosts文件或使用国内GitHub加速服务：

```bash
# 修改远程URL为加速服务
git remote set-url origin https://ghproxy.com/https://github.com/xingqiux/aurain_admin_spring.git
```


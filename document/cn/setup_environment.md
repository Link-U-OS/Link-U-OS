# 安装开发环境

## 克隆 Link-U-OS 代码仓库
```bash
git clone --recursive git@github.com:Link-U-OS/Link-U-OS.git
```

## 一键配置开发环境

### 启动配置流程
进入项目目录并打开 VSCode:
```bash
cd Link-U-OS
code .
```

### 配置步骤

**前置准备**
- 建议在执行上述命令前,先在主机上完成 Docker 的安装和配置
- 如果这是首次配置 Docker,系统会要求您重启电脑以应用[用户组权限设置](./faq.md)
- 重启后,请再次运行 `code .` 命令

**容器环境配置**
1. VSCode 自动打开后,右下角会弹出通知提示
2. 点击 **Reopen in Container** 按钮,等待容器构建完成
3. 在 VSCode 中打开新的 Terminal (快捷键: `` Ctrl + ` ``)
4. 输入 `setup-init` 命令并根据提示操作
5. 当看到 **🎉🎉🎉 Setup successful** 提示时,表示配置成功,可以开始开发

## 重新构建或更新 aimde

当 aimde 发布新版本或开发环境出现问题时,可通过以下步骤重建容器:

1. 在 VSCode 中按 `Ctrl + Shift + P` 打开命令面板
2. 输入 `Remote-Containers: Rebuild Container` 并回车
3. 等待容器重新构建完成

> **提示**: 重建容器会清除容器内的临时数据,请确保重要文件已保存到挂载的项目目录中
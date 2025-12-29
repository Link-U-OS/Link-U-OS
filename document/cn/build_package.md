# 产品包构建指南

## 前置准备

### 拉取 integration 仓库

如果已通过[克隆 LinkQ 代码仓库](./setup_environment.md#克隆-linkq-代码仓库)完成操作，integration 仓库应已存在于同级目录中，可跳过此步骤。

否则，执行以下命令：
```bash
cd ..
git clone git@github.com:Link-U-OS/integration.git
```

## 编译产品包

### 基本用法
```bash
cd integration
bash tools/build_package.sh [参数]
```

### 参数说明

| 参数 | 说明 | 默认值 | 示例 |
|------|------|--------|------|
| `-p` | 指定编译平台类型 | `A2_ULTRA` | `-p A2_ULTRA` |
| `-j` | 指定编译线程数 | `0` (自动) | `-j 8` |
| `-g` | 编译调试版本 | 关闭 | `-g` |
| `-o` | 使用本地源码编译指定仓库 | 无 | `-o aimrt_comm=../aimrt_comm` |
| `-c` | 清除编译缓存 | 关闭 | `-c` |
| `-e` | 传递额外编译选项 | 无 | `-e "--config=commit"` |

### 编译模式

**分支模式(默认)**
- 使用 `projects/defs/source_branch.yaml` 中指定的分支源码
- 若需每次自动拉取最新代码，先执行 `bazel clean --expunge`

**节点模式**
- 添加参数：`-e "--config=commit"`
- 使用 `projects/defs/source_commit.yaml` 中指定的提交节点源码

### 使用示例
```bash
# 基本编译
bash tools/build_package.sh

# 使用 8 线程编译调试版本
bash tools/build_package.sh -j 8 -g

# 清除缓存后编译，使用本地 aimrt_comm 源码
bash tools/build_package.sh -c -o aimrt_comm=../aimrt_comm

# 使用指定提交节点编译
bash tools/build_package.sh -e "--config=commit"
```

## 编译结果

### 输出位置

编译完成后，产品包生成在 `/home/vscode/.local/share/local_snapshot/` 目录下，终端会打印具体路径：

![编译示例](./images/snapshot.png)

### 导出到主机

为便于在主机上部署，可将产品包复制到挂载目录：
```bash
cp /home/vscode/.local/share/local_snapshot/A2_ULTRA-741f41e25-2512221514.tar /workspaces/Link-U-OS/
```

> **提示**：请根据实际生成的文件名替换上述命令中的包名称。
# Development Environment Setup

## Clone the Link-U-OS Repository
```bash
git clone --recursive git@github.com:Link-U-OS/Link-U-OS.git
```

## One-Click Development Environment Configuration

### Start Configuration Process
Navigate to the project directory and open VSCode:
```bash
cd Link-U-OS
code .
```

### Configuration Steps

**Prerequisites**
- It's recommended to install and configure Docker on your host machine before executing the above commands
- If this is your first time configuring Docker, the system will require you to restart your computer to apply [user group permission settings](./faq.md)
- After restarting, run the `code .` command again

**Container Environment Configuration**
1. After VSCode opens automatically, a notification will appear in the bottom right corner
2. Click the **Reopen in Container** button and wait for the container to build
3. Open a new Terminal in VSCode (shortcut: `` Ctrl + ` ``)
4. Enter the `setup-init` command and follow the prompts
5. When you see the **🎉🎉🎉 Setup successful** message, the configuration is complete and you can begin development

## Rebuild or Update aimde

When a new version of aimde is released or issues occur with the development environment, you can rebuild the container with the following steps:

1. Press `Ctrl + Shift + P` in VSCode to open the command palette
2. Type `Remote-Containers: Rebuild Container` and press Enter
3. Wait for the container to rebuild

> **Tip**: Rebuilding the container will clear temporary data inside the container. Make sure important files are saved to the mounted project directory
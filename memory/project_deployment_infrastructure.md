---
name: Deployment infrastructure added 2026-04-29
description: Project gained full deployment stack: Dockerfile, docker-compose.prod.yml, Linux/Windows install scripts, systemd service, build_package.sh, PyInstaller spec.
type: project
originSessionId: 85d955f7-35ff-4ef5-9bb9-d985b2fc559d
---
# 部署基础设施（2026-04-29 新增）

**事实**：项目新增了完整的部署/打包基础设施，尚未合并到 main（git status 显示为 untracked）。

**Why**: 项目需要支持多种部署场景：Docker 容器化、Linux 服务器安装、Windows Server 安装、以及离线打包分发。

**新增文件清单**：

| 文件 | 用途 |
|------|------|
| `Dockerfile` | Docker 容器化部署 |
| `docker-compose.prod.yml` | 生产环境 Docker Compose |
| `docs/deployment_guide.md` | 部署指南文档 |
| `scripts/build_package.sh` | 打包脚本（bash，支持 PyArmor 混淆） |
| `scripts/first_run_setup.py` | 首次运行初始化脚本 |
| `scripts/install_linux.sh` | Linux 服务器安装脚本 |
| `scripts/install_windows_server.ps1` | Windows Server 安装脚本 |
| `scripts/manage_services.ps1` | Windows 服务管理脚本 |
| `scripts/smart-energy-platform.service` | systemd 服务单元文件 |
| `scripts/trim_license_manager.py` | 许可证管理器精简工具 |
| `scripts/uninstall_windows_server.ps1` | Windows Server 卸载脚本 |
| `smart_energy.spec` | PyInstaller 打包规格文件 |

**build_package.sh 已知 bug（2026-04-29 修复）**：
- Line 76：`for` 循环结束符误写为 `fi`（应为 `done`），导致 `syntax error near unexpected token 'fi'`
- 已修复为 `done`
- 另有 line 41 的 `$(! $SKIP_OBFUSCATE && ...)` 在 `set -e` 下有潜在问题，已改为 `$([ "$SKIP_OBFUSCATE" = true ] && echo 'NO' || echo 'YES')`

**How to apply**: 涉及部署相关工作时，参考这些文件。`docs/deployment_guide.md` 是用户面向的部署文档。

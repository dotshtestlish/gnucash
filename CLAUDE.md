# GnuCash CI/CD 自动化指南

> 本文件面向 AI 助手和开发者，记录云端编译与 MCP 集成配置。

## GitHub Actions 工作流

### 已有工作流

| 文件 | 触发 | 说明 |
|---|---|---|
| `.github/workflows/ci-tests.yml` | push, pull_request | Ubuntu 22.04 + Address Sanitizer 测试 |
| `.github/workflows/ci-docker.yml` | push, pull_request | Arch Linux Docker 容器测试 |
| `.github/workflows/coverage.yml` | push | C++ 代码覆盖率分析 + GitHub Pages 部署 |
| `.github/workflows/mac-tests.yaml` | push, pull_request | macOS 编译与测试 |
| `.github/workflows/cloud-build.yml` | push + workflow_dispatch | **MCP 驱动的云端编译（多平台）** |

### cloud-build.yml — MCP 云端编译工作流

专为通过 GitHub MCP 触发云端编译设计，支持参数化编译。

#### 手动触发参数

通过 GitHub UI 或 MCP API 传入：

| 参数 | 默认值 | 可选值 |
|---|---|---|
| `platform` | `linux` | `linux`, `macos`, `linux+macos` |
| `build_type` | `RelWithDebInfo` | `Debug`, `RelWithDebInfo`, `Release`, `Asan` |
| `with_python` | `true` | `true`, `false` |
| `upload_artifact` | `true` | `true`, `false` |

## MCP 集成配置

### GitHub MCP 客户端配置

在 MCP 客户端中配置 GitHub MCP 服务器，以通过自然语言触发编译。

**Claude Desktop / Claude Code 配置：**

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "<your-github-personal-access-token>"
      }
    }
  }
}
```

**所需 Token 权限：**
- `repo` (Full control of private repositories)
- `workflow` (Update GitHub Actions workflows)

### MCP 触发示例

配置完成后，AI 可通过以下方式操作：

```
"触发 gnucash 项目的最新云端编译，平台 linux，Release 模式"
"查询最后一次编译的状态"
"下载最新的编译产物"
```

对应的底层 API 调用：

```bash
# 触发 workflow_dispatch
curl -X POST \
  -H "Authorization: Bearer <PAT>" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/<owner>/gnucash/actions/workflows/cloud-build.yml/dispatches \
  -d '{"ref":"stable","inputs":{"platform":"linux","build_type":"Release"}}'

# 查询工作流运行状态
curl -H "Authorization: Bearer <PAT>" \
  https://api.github.com/repos/<owner>/gnucash/actions/workflows/cloud-build.yml/runs

# 列出 artifact
curl -H "Authorization: Bearer <PAT>" \
  https://api.github.com/repos/<owner>/gnucash/actions/artifacts
```

## 构建系统参考

- **构建工具**: CMake ≥ 3.14.5 + Ninja
- **编译器**: GCC ≥ 8.0 或 Clang ≥ 6.0（C++17）
- **C++ 测试**: Google Test 1.8.0+ (gtest + gmock)
- **完整依赖**: 见 [README.dependencies](README.dependencies)
- **CMake 详细指南**: [cmake/README_CMAKE.txt](cmake/README_CMAKE.txt)

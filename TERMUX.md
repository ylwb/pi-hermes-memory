# pi-hermes-memory Termux Build

## 概览

将 `chandra447/pi-hermes-memory` 适配到 Termux (ARM64 Linux)。

## 远程编译流程

```
PC 浏览器 → GitHub Actions → 编译产物 → Termux 安装
```

## GitHub Actions Workflows

| Workflow | 文件 | 用途 |
|----------|------|------|
| CI | `.github/workflows/ci.yml` | 原有 CI（TypeScript 检查、测试） |
| Publish | `.github/workflows/publish.yml` | 发布到 npm |
| **Termux Build** | `.github/workflows/termux-build.yml` | ARM64 编译 + 产物打包 |

## 触发方式

### 方式 1：手动触发（PC 浏览器）
```
GitHub → ylwb/pi-hermes-memory → Actions → Termux Build → Run workflow
```

### 方式 2：命令行触发
```bash
gh workflow run termux-build.yml
```

### 方式 3：Push 触发
```bash
git checkout -b termux
git push -u fork termux
```

## Termux 安装

### 从 npm 安装（推荐）
```bash
pi install npm:pi-hermes-memory
```

### 从 GitHub 安装
```bash
pi install github:ylwb/pi-hermes-memory
```

### 手动安装（使用远程编译产物）
```bash
# 下载产物
gh run download --name pi-hermes-memory-termux -D ~/.pi/agent/extensions/pi-hermes-memory

# 安装
cd ~/.pi/agent/extensions/pi-hermes-memory
npm install --production

# 初始化
/memory-index-sessions
```

## 依赖说明

| 依赖 | 用途 | Termux 兼容性 |
|------|------|--------------|
| `better-sqlite3` | SQLite 存储 | ✅ Node 24 预编译 arm64 |
| `@earendil-works/pi-tui` | Pi TUI | ✅ npm 原生支持 |
| `strip-ansi` | ANSI 处理 | ✅ JS 纯函数 |
| `@earendil-works/pi-coding-agent` | Pi SDK (peer) | ✅ 已安装 |

## better-sqlite3 ABI 兼容性

项目内置 ABI 不匹配恢复机制：
```typescript
// src/store/sqlite-native.ts
// 自动检测 Node 版本差异，触发 npm rebuild
export function loadBetterSqlite3(options): BetterSqlite3DatabaseCtor
```

如果首次加载失败，会自动重建：
```bash
cd ~/.pi/agent/extensions/pi-hermes-memory
npm rebuild better-sqlite3
```

## Bun 运行时回退

Pi 编译版本使用 Bun 运行时，`bun:sqlite` 作为备用：
```typescript
// 当 better-sqlite3 不可用时，使用 bun:sqlite
if (isBunRuntime()) {
  const bunSqlite = require("bun:sqlite");
  // ...
}
```

## 快速验证

```bash
# 检查安装
pi status

# 索引历史会话
/memory-index-sessions

# 测试记忆搜索
/memory-search "authentication"

# 查看记忆
/memory-insights
```

## 文件结构

```
ylwb/pi-hermes-memory/
├── .github/workflows/
│   ├── ci.yml              # 原有 CI
│   ├── publish.yml         # npm 发布
│   └── termux-build.yml    # Termux 构建 ⭐
├── src/                    # TypeScript 源码
├── scripts/                # 构建脚本
├── tests/                  # 测试套件
└── package.json
```

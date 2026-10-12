---
title: npm 包发布完全指南：从零到上线的规范流程
date: 2026-10-12 15:45:00
tags:
  - npm
  - nodejs
categories: nodejs
---

## 前言

npm（Node Package Manager）是全球最大的软件注册表，托管着超过百万个包。对于 Node.js 和前端开发者而言，将自己编写的工具库、组件或插件发布到 npm，是实现代码复用、参与开源贡献、以及在团队内部分享私有工具的标准做法。

发布一个 npm 包在技术操作上并不复杂，但一个“安全、规范、可维护”的包与一个“能装但会出问题”的包之间，差距恰恰在于发布前容易被忽略的细节：哪些文件会被打包上传、版本号如何管理、如何处理作用域包、以及如何配置认证机制。本文系统梳理 npm 包发布的完整流程、配置要点、版本管理、安全实践与常见问题排查，帮助你完成从初始化项目到成功上线的全过程。

---

## 一、前置准备

### 1.1 环境要求

确保本地已安装 Node.js 与 npm，可通过以下命令确认版本：

```bash
node -v
npm -v
```

### 1.2 注册与登录 npm 账号

访问 [npmjs.com](https://www.npmjs.com/) 注册账号，完成邮箱验证。用户名具有全局唯一性，且将作为后续发布作用域包（`@username/package-name`）时的命名空间，建议使用有辨识度的名称。

注册完成后，在终端登录：

```bash
npm login
```

按提示输入用户名、密码和邮箱。登录成功后，终端会显示 `Logged in as <username>`。可通过以下命令确认当前登录状态：

```bash
npm whoami
```

如果输出了你的用户名，说明已成功登录。

### 1.3 检查包名是否可用

npm 上的包名全局唯一，发布前应先确认目标名称未被占用：

```bash
npm view <your-package-name>
```

如果提示 `npm ERR! 404 Not Found`，说明该名称可用。反之，则需要更换名称或改用作用域包。

---

## 二、初始化项目与 package.json 配置

### 2.1 初始化

创建一个项目目录并初始化：

```bash
mkdir my-awesome-package
cd my-awesome-package
npm init -y
```

`npm init -y` 会生成一个默认的 `package.json`，随后需要手动编辑其中的关键字段。

### 2.2 核心字段说明

一个用于发布的 `package.json` 应至少包含以下字段：

```json
{
  "name": "my-awesome-package",
  "version": "1.0.0",
  "description": "一个简洁的描述，便于在 npm 搜索中被发现",
  "main": "./dist/index.cjs",
  "module": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    }
  },
  "files": ["dist", "README.md", "LICENSE"],
  "scripts": {
    "build": "tsup src/index.ts --format esm,cjs --dts",
    "test": "vitest run",
    "prepublishOnly": "npm run test && npm run build"
  },
  "keywords": ["utility", "typescript"],
  "author": "Your Name",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/yourname/my-awesome-package.git"
  }
}
```

**字段说明**：

| 字段      | 作用                    | 注意事项                                                               |
| --------- | ----------------------- | ---------------------------------------------------------------------- |
| `name`    | 包名                    | 全局唯一，不能有大写字母、不能以点或下划线开头，长度不超过 214 字符    |
| `version` | 版本号                  | 遵循语义化版本规范（SemVer）                                           |
| `main`    | CommonJS 入口           | 指向编译后的 `.cjs` 文件                                               |
| `module`  | ESM 入口                | 指向编译后的 `.js` 文件                                                |
| `types`   | TypeScript 类型声明入口 | 使类型化项目无需额外安装 `@types`                                      |
| `exports` | 条件导出映射            | 现代包应使用 `exports` 字段统一定义入口，优先级高于 `main` 和 `module` |
| `files`   | 白名单                  | 控制哪些文件被包含在发布包中，比 `.npmignore` 更安全                   |
| `scripts` | 脚本命令                | `prepublishOnly` 在发布前自动执行，适合用作测试和构建的门禁            |

---

## 三、控制发布内容：files 白名单与 dry-run 验证

### 3.1 为什么需要 files 字段

默认情况下，npm 会发布目录下几乎所有的文件（除 `.gitignore` 中排除的之外），这意味着 `.env`、测试数据、源码映射文件等敏感或不必要的内容可能被意外上传。

**推荐使用 `files` 白名单**，而非 `.npmignore` 黑名单。白名单的逻辑是：只发布明确列出的文件，忘记添加的文件会被排除，而不会泄露。

### 3.2 发布前验证

在执行正式发布之前，务必先查看实际将被上传的文件列表：

```bash
npm pack --dry-run
```

该命令会列出打包进 tarball 的所有文件，但不会实际创建文件。仔细检查输出中是否包含 `.env`、`*.key`、`secrets` 等不应公开的文件。

---

## 四、版本管理与更新

### 4.1 语义化版本规范

npm 使用语义化版本（Semantic Versioning，SemVer），格式为 `MAJOR.MINOR.PATCH`：

| 版本变更                | 适用场景           | 命令                |
| ----------------------- | ------------------ | ------------------- |
| `PATCH` (1.0.0 → 1.0.1) | 向后兼容的缺陷修复 | `npm version patch` |
| `MINOR` (1.0.0 → 1.1.0) | 向后兼容的新功能   | `npm version minor` |
| `MAJOR` (1.0.0 → 2.0.0) | 不向后兼容的变更   | `npm version major` |

### 4.2 使用 npm version 命令

`npm version` 命令会自动更新 `package.json` 中的版本号，并同步更新 `package-lock.json`（如果存在），同时创建一个 Git 提交和标签：

```bash
npm version patch -m "Upgrade to %s for bug fix"
```

其中 `%s` 会被替换为新版本号。

### 4.3 重要规则

**每次发布必须更新版本号**。npm 不允许对同一个包名和版本号重复发布，即使该版本已被撤销（unpublish），该名称和版本的组合也将永久不可用。如果尝试发布已存在的版本，会报错：

```
npm ERR! You cannot publish over the previously published version
```

---

## 五、作用域包（Scoped Packages）

### 5.1 什么作用域包

作用域包以 `@scope-name/package-name` 的形式命名，其中 `@scope-name` 可以是你的用户名或组织名。使用作用域包的好处包括：

- **命名空间隔离**：避免与公共包名冲突。
- **支持私有发布**：可以发布仅团队成员可访问的私有包。
- **组织化**：便于按团队或项目组织包。

### 5.2 创建作用域包

在初始化时指定作用域：

```bash
npm init --scope=@yourname -y
```

这会生成一个名为 `@yourname/package-name` 的包。

### 5.3 发布作用域包

**关键区别**：作用域包默认发布为**私有（restricted）**，如果希望公开可安装，必须显式加上 `--access public`：

```bash
npm publish --access public
```

如果不加此参数，首次发布时会报权限错误。

> **注意**：发布私有组织作用域包需要付费的 npm 组织账号，用户作用域包（`@username/package`）发布为公开时无需付费。

---

## 六、发布与更新

### 6.1 首次发布

确认一切就绪后，执行发布命令：

```bash
npm publish
```

如果是作用域包且需要公开：

```bash
npm publish --access public
```

发布成功后，可以在 `https://www.npmjs.com/package/<package-name>` 查看包页面，其他人即可通过 `npm install <package-name>` 安装使用。

### 6.2 更新已发布的包

更新流程与首次发布基本一致，区别在于**必须升级版本号**：

```bash
# 1. 修改代码
# 2. 升级版本号
npm version patch   # 或 minor / major

# 3. 重新发布
npm publish
```

---

## 七、安全与认证：2FA 与 Access Token

### 7.1 双因素认证（2FA）

npm 现在要求发布包时必须启用以下任一认证方式：

1. **账号级双因素认证（2FA）**：在 npm 账号设置中开启 2FA，发布时需输入一次性验证码。
2. **细粒度访问令牌（Granular Access Token，GAT）** ：创建一个绕过 2FA 的令牌，适用于 CI/CD 自动化场景。

**开启 2FA 的路径**：登录 npmjs.com → Account Settings → Authentication & Security → Two-factor Authentication，选择 “Authorization and publishing” 级别。

### 7.2 使用 Access Token 发布

在 CI/CD 流水线中，推荐使用 Access Token 而非交互式登录。创建令牌的步骤：

1. 访问 `https://www.npmjs.com/settings/<username>/tokens`
2. 点击 “Generate New Token”
3. 选择 “Granular Access Token” 类型
4. 配置包的读写权限，勾选 “Bypass two-factor authentication”

创建完成后，在项目根目录创建 `.npmrc` 文件：

```
//registry.npmjs.org/:_authToken=${NPM_TOKEN}
```

通过环境变量传入令牌：

```bash
export NPM_TOKEN=your_token_here
npm publish
```

> **安全提示**：`.npmrc` 文件中包含令牌信息，务必将其加入 `.gitignore`，避免提交到版本控制。

---

## 八、常见错误与排查

| 错误码               | 含义     | 原因                                           | 解决方案                                      |
| -------------------- | -------- | ---------------------------------------------- | --------------------------------------------- |
| **E401**             | 未授权   | 未登录或令牌失效                               | 重新执行 `npm login` 或更新令牌               |
| **E403**             | 禁止访问 | 包名已被占用，或作用域包未加 `--access public` | 更换包名，或补加 `--access public`            |
| **E404**             | 未找到   | 包名拼写错误，或私有包无权限                   | 检查包名拼写，确认令牌权限                    |
| **EPUBLISHCONFLICT** | 版本冲突 | 尝试发布已存在的版本号                         | 执行 `npm version patch` 升级版本号后重新发布 |

**其他常见问题**：

- **文件过大**：检查 `files` 字段是否误包含了 `node_modules` 或构建产物之外的大文件。
- **忘记构建**：如果使用 TypeScript，`prepublishOnly` 脚本应包含 `npm run build`，确保发布的是编译后的产物而非源码。
- **2FA 验证码无法获取**：如果使用 CI/CD，不应依赖交互式 OTP，应改用带 Bypass 2FA 的 Access Token。

---

## 九、总结

npm 包发布的完整流程可以概括为：**准备账号 → 初始化项目 → 配置 package.json → 控制发布内容 → 管理版本 → 执行发布**。核心原则有三条：

1. **安全第一**：使用 `files` 白名单而非 `.npmignore` 黑名单，发布前用 `npm pack --dry-run` 验证，确保不泄露敏感文件。
2. **版本规范**：严格遵循 SemVer，每次发布必须升级版本号，已发布的版本号不可复用。
3. **认证到位**：启用 2FA 或配置 Granular Access Token，CI/CD 场景使用环境变量传递令牌，避免将令牌提交到仓库。

掌握这套流程后，发布一个 npm 包不过是几分钟的事情。真正需要持续投入的，是包的 API 设计、文档质量和长期维护。

---

_本文基于 npm 10.x 与 Node.js 20.x 实测，命令同样适用于 pnpm（`pnpm publish`）和 Yarn（`yarn publish`）。_

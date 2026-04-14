# MapV-Three Skills

MapV-Three 3D 地图开发 AI 助手 Skills，供 Claude、Cursor 等在使用 MapV-Three 相关 API 时加载，以提供准确的 API 说明与示例。

## 包含的 Skill

| Skill | 说明 |
|-------|------|
| mapv-three-skills | 百度地图 JSAPI Three 版 (MapV-Three)：基于 Three.js 的专业 3D 地图渲染引擎，支持多源底图加载、天空与天气系统、3DTiles、可视化组件、3D 模型加载、LBS 服务、GIS 空间分析、编辑测量等完整 3D 地图开发能力。适用于构建数字孪生、城市可视化、WebGIS 等应用。 |

## 安装方式

任选以下一种方式安装即可。

### 方式一：npx skills add（推荐）

若你的环境支持 skills CLI，可用一条命令安装本仓库的 skills：

```bash
npx skills add baidu-maps/mapv-three-skills
```

会将本仓库中的全部 skills 安装到当前环境的 skills 目录。

### 方式二：手动安装

**第一步：获取仓库**

克隆本仓库：

```bash
git clone https://github.com/baidu-maps/mapv-three-skills.git
cd mapv-three-skills
```

或从 [Releases](https://github.com/baidu-maps/mapv-three-skills/releases) 下载附件 `mapv-three-skills.zip` 后解压：

```bash
unzip mapv-three-skills.zip
```

**第二步：注册 Skill**

将 `skills/` 目录下的 `mapv-three-skills` 链接或复制到当前环境对应的 skills 目录。

**Claude Desktop（本地）**

Skills 目录一般为：`~/.claude/skills/`

注册（软链，推荐）：

```bash
ln -sfn "$(pwd)/skills/mapv-three-skills" ~/.claude/skills/mapv-three-skills
```

或直接把 `skills/mapv-three-skills` 文件夹复制到 `~/.claude/skills/` 下。

**Cursor**

Skills 目录一般为：`~/.cursor/skills/`

注册（软链，推荐）：

```bash
ln -sfn "$(pwd)/skills/mapv-three-skills" ~/.cursor/skills/mapv-three-skills
```

或直接把 `skills/mapv-three-skills` 文件夹复制到 `~/.cursor/skills/` 下。

## 如何使用

在支持 Skills 的客户端里，当你的问题涉及「MapV-Three」「mapv-three」「3D 地图」「数字孪生」等时，助手会优先参考本仓库中对应 skill 的文档来回答，从而给出更贴合 MapV-Three 的代码与用法。

## 仓库结构

```text
.
├── skills/
│   └── mapv-three-skills/          # MapV-Three 3D 地图 Skill
│       ├── SKILL.md                # Skill 入口与索引
│       └── references/             # API 参考文档
└── README.md
```

`SKILL.md` 中会列出其下所有参考文档，便于 AI 按需读取。

## 许可

MIT

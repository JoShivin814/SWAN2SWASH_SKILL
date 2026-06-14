# SWAN2SWASH_SKILL
a tool about transform swan to swash
# swan2swash Skill 导出包

将本文件夹整体复制到其他电脑或智能体环境即可使用。

## 目录结构

```
swan2swash_skill_export/
├── README.md                 # 本说明
├── swan2swash/                # Skill 主体（必需）
│   ├── SKILL.md
│   ├── mapping.md
│   └── examples.md
└── swan2swash_ex/            # 参考算例（建议一并带上）
    ├── 2bb1v4.swn
    └── 2bb1v4.sws
```

## 安装到 Cursor

**个人级（所有项目可用）：**

将 `swan2swash` 文件夹复制到：

- Windows: `%USERPROFILE%\.cursor\skills\swan2swash\`
- macOS/Linux: `~/.cursor/skills/swan2swash/`

**项目级（仅当前仓库）：**

复制到项目根目录：`.cursor/skills/swan2swash/`

在对话中输入 `/swan2swash` 或说明要做 SWAN/SWASH 转换即可触发。

## 安装到其他智能体

| 平台 | 做法 |
|------|------|
| 支持 Agent Skills 的 IDE | 同上，放到该产品的 `skills` 目录 |
| 自定义 GPT / Claude Project | 将 `SKILL.md` + `mapping.md` + `examples.md` 上传为知识库，或把 `SKILL.md` 全文写入系统提示 |
| 通用脚本助手 | 把 `swan2swash/SKILL.md` 作为每次任务开头的系统指令片段 |

## 配套手册（可选）

Skill 内引用了 `swanuse.md`、`swashuse.md`。若在不含 usebook 的项目中使用，请：

1. 把两份手册放在工作区根目录，或  
2. 在对话中 @ 引用手册路径。

## 已复制位置（本机）

- 个人 Cursor skills: `C:\Users\Y9000P\.cursor\skills\swan2swash\`
- 项目内原路径: `usebook\.cursor\skills\swan2swash\`

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

**每日猜成语Wordle Handle** — 汉字 Wordle，玩家需在 10 次尝试内猜出一个四字成语。每次猜测会返回汉字匹配状态（完全匹配 / 位置错误 / 不存在）和拼音声调的反馈。

线上地址：wordle.luomor.com

## 常用命令

| 命令 | 说明 |
|---------|-------------|
| `pnpm install` | 安装依赖 |
| `pnpm dev` | 启动开发服务器（端口 4444） |
| `pnpm build` | 生产环境构建 |
| `pnpm preview` | 预览生产构建 |
| `pnpm test` | 运行 vitest 测试 |
| `pnpm test <文件>` | 运行指定测试文件 |
| `pnpm lint` | 运行 eslint |
| `pnpm run update` | 更新成语数据库（通过爬取 zdic.net，由 `vite-node tools/update.ts` 执行） |

包管理器：pnpm@7.8.0（pnpm workspace，包含 `packages/*`）。

## 架构

### 技术栈
- Vue 3（Composition API）+ Vite
- UnoCSS 样式，VueUse 组合式函数
- unplugin-auto-import + unplugin-vue-components 自动导入
- Vitest 测试，jsdom DOM 环境

### 源码结构

```
src/
  App.vue              # 根组件，页面路由分发
  main.ts              # 入口文件
  state.ts             # 游戏响应式状态（parsedAnswer、parsedTries、isPassed 等）
  storage.ts           # localStorage 持久化（尝试记录、历史、设置）
  i18n.ts              # 国际化（zh-cn / zh-tw）
  init.ts              # 应用初始化逻辑

  logic/               # 核心游戏引擎
    types.ts           # ParsedChar、MatchResult、TriesMeta、InputMode 类型定义
    constants.ts       # WORD_LENGTH=4、TRIES_LIMIT=10、START_DATE 等常量
    idioms.ts          # 成语加载与校验
    check.ts           # 答案判定逻辑
    utils.ts           # 拼音解析工具
    index.ts           # 统一导出

  answers/             # 每日答案系统
    list.ts            # 预生成的答案列表（[成语, 提示] 数组）
    index.ts           # getAnswerOfDay(day) — 通过 seedrandom 确定性选取
    utils.ts           # 答案工具函数

  components/          # Vue 组件（通过 unplugin-vue-components 自动导入）
    Play.vue           # 主游戏面板 / 输入
    CharBlock.vue      # 单字方块
    Dashboard.vue      # 统计 / 历史面板
    ResultFooter.vue   # 游戏结束结果展示
    ShareDialog.vue    # 分享功能
    Settings.vue       # 设置弹窗
    ...

  data/                # 游戏数据
    idioms.txt         ~20k+ 有效成语列表
    polyphones.json    多音字 / 特殊发音成语
    new.txt            待处理的新成语
    t2s.json           繁简映射表

packages/tools/        @hankit/tools — 可复用的汉字/拼音工具库
  src/pinyin/          拼音解析、音素、样式
  src/shuangpin/       双拼输入支持（搜狗、小鹤）
  src/zhuyin/          注音（ㄅㄆㄇ）支持
  src/hanzi/           汉字工具（筛选、转换）
  src/map/             映射数据（音素、声调符号等）

tools/                 构建 / 自动化脚本
  update.ts            爬取 zdic.net 更新成语数据库
  zdict.ts             汉典 API 客户端

test/                  Vitest 测试文件
```

### 核心游戏流程

1. **答案选取**：`answers/index.ts` → `getAnswerOfDay(day)` 通过 seedrandom 确定性选取每日成语
2. **输入解析**：`state.ts` 中的 `parseWord()` 将用户输入转为 `ParsedChar[]`（汉字 + 拼音各部分）
3. **匹配判定**：`testAnswer()` 将解析后的输入与答案逐字对比，返回每个字的 `MatchResult`
4. **状态管理**：`state.ts` 持有游戏响应式状态；`storage.ts` 将尝试记录和历史持久化到 localStorage
5. **输入模式**：支持拼音（`py`）、注音（`zy`）、双拼（`sp`）三种输入模式

### 注意事项

- 答案列表（`src/answers/list.ts`）为预生成；当天数超出列表长度后，通过确定性种子随机选取
- `logic/constants.ts` 中的 `START_DATE` 控制每日谜题的起始日期，当前设为 2026-01-01
- `WORD_LENGTH=4`（四字成语），`TRIES_LIMIT=10`（最多 10 次尝试）
- `@hankit/tools` 在 vite/tsconfig 中配置了别名并自动导入，修改该包会影响整个应用
- 全部样式使用 UnoCSS，除 `src/styles/main.css` 用于全局 reset 外无传统 CSS 文件

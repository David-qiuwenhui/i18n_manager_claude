# Design: I18N词条管理工具

## Context

### 当前状态

- 项目有 20+ 开发者参与前端开发
- I18N 词条需要手动导入到统一的公服系统
- 词条管理缺乏规范化和可追溯性
- 多人协作时容易产生冲突，修改历史难以追踪

### 约束条件

- 必须与现有的 Git 工作流集成
- 不能破坏现有的前端开发流程
- 需要保持与公服系统的兼容性（Excel 导入格式）
- 开发者学习成本需要尽可能低
- 词条量约 10000 条，需确保性能可接受

### 利益相关者

- **开发者**: 需要简单易用的词条管理方式
- **代码审查者**: 需要清晰的词条变更视图
- **运维/平台团队**: 需要稳定的工具和 CI 集成

## Goals / Non-Goals

**Goals:**

- 提供按业务模块组织的词条文件结构，便于团队协作
- 实现全局词条校验，确保数据质量（唯一性、完整性、格式）
- 支持导出为公服系统可导入的 Excel 文件
- 提供词条差异对比功能，便于同步管理
- 完全基于 Git 工作流，实现修改可追溯

**Non-Goals:**

- 不提供实时同步到公服系统的功能（保持手动导入流程）
- 不提供 Web UI 管理界面（纯 CLI 工具）
- 不支持除中英文之外的其他语言（可后续扩展）
- 不修改公服系统本身

## Decisions

### 1. 技术栈选择：yarn + TypeScript + vitest + xlsx

**决策**: 使用 yarn、TypeScript、vitest 和 xlsx 库开发 CLI 工具

**理由**:
- **yarn**: 更快的安装速度和更好的依赖管理
- **TypeScript**: 提供类型安全，减少运行时错误
- **vitest**: 快速的单元测试框架，与 Vite 生态一致
- **xlsx**: 轻量级 Excel 库，满足 10000 条词条的导出需求
- **commander**: 简洁的 CLI 框架
- **chalk**: 终端颜色化输出

**替代方案**:
- Python: 需要 Python 环境，与前端项目技术栈不一致
- Go: 编译后二进制分发，但缺乏 npm script 集成便利性
- exceljs: 功能更强大但更重，xlsx 足够满足需求

### 2. 词条文件格式：JSON

**决策**: 使用 JSON 格式存储词条

**理由**:
- 原生支持于 JavaScript/TypeScript，无需额外解析
- 与前端项目现有的配置文件格式一致
- 易于 Git diff 查看变更
- 支持注释（虽然标准 JSON 不支持，可通过工具处理）

**替代方案**:
- YAML: 更简洁，但需要额外解析依赖
- TOML: 不够流行，学习成本高

### 3. 文件组织方式：按业务模块拆分

**决策**: 按业务模块（`ebankingPermission`, `channelPermission`, `physicalPermission`, `common`）拆分词条文件

**理由**:
- 减少文件锁定冲突，多人可并行编辑不同模块
- 便于按业务领域分配维护责任
- 文件大小可控，易于导航和查找

**文件结构**:
```
i18n/
├── ebankingPermission.json
├── channelPermission.json
├── physicalPermission.json
└── common.json
```

**替代方案**:
- 单文件: 冲突频繁，大文件难以维护
- 按页面拆分: 文件过多，管理复杂

### 4. CLI 框架：Commander.js

**决策**: 使用 Commander.js 构建 CLI

**理由**:
- 简洁的 API，易于定义子命令
- 自动生成帮助信息
- 广泛使用，社区成熟

**核心命令结构**:
```bash
i18n-cli validate          # 校验
i18n-cli export            # 导出
i18n-cli diff <file>       # 差异对比
```

### 5. Excel 生成库与格式：xlsx

**决策**: 使用 xlsx 库生成 Excel 文件，采用公服系统要求的格式

**理由**:
- 轻量级，满足 10000 条词条的导出需求
- 良好的 UTF-8 中文支持
- 内存构建即可，无需流式写入

**输出格式**（公服系统要求）:
| Key | IContent | Language | Remark |
|-----|----------|----------|--------|
| ebankingPermission.login.title | 用户登录 | zh-CN | |
| ebankingPermission.login.title | User Login | en-US | |

**说明**: 每条词条展开为两行（中英文各一行），按语言列 `Language` 区分

### 6. 校验策略：先收集后校验

**决策**: 跨模块收集所有词条后再执行校验

**理由**:
- 支持跨模块键名唯一性检测
- 一次扫描完成所有校验，性能更优
- 错误报告更完整，便于批量修复

**校验规则**:
1. JSON 格式正确性
2. 键名格式（小写 + 点分隔）
3. 跨模块键名唯一性
4. 中英文翻译完整性

### 7. 配置管理：i18n.config.json

**决策**: 使用配置文件管理项目设置

**理由**:
- 支持自定义词条目录位置
- 可配置校验规则（如允许的键名格式）
- 便于未来扩展

**配置示例**:
```json
{
  "entryDir": "./i18n",
  "defaultLocale": "zh-CN",
  "supportedLocales": ["zh-CN", "en-US"],
  "validation": {
    "keyFormat": "^[a-z]+(\\.[a-z]+)*$",
    "requireAllLocales": true
  }
}
```

## Architecture

### 模块结构

```
i18n-cli/
├── src/
│   ├── commands/
│   │   ├── validate.ts
│   │   ├── export.ts
│   │   └── diff.ts
│   ├── core/
│   │   ├── loader.ts       # 词条文件加载
│   │   ├── validator.ts    # 校验器
│   │   ├── exporter.ts     # Excel 导出
│   │   └── differ.ts       # 差异对比
│   ├── types/
│   │   └── index.ts        # 类型定义
│   ├── utils/
│   │   ├── config.ts       # 配置管理
│   │   └── logger.ts       # 日志工具
│   └── index.ts            # CLI 入口
├── tests/
│   ├── core/
│   │   ├── loader.test.ts
│   │   ├── validator.test.ts
│   │   ├── exporter.test.ts
│   │   └── differ.test.ts
│   └── fixtures/
│       └── i18n/           # 测试用词条文件
├── package.json
├── tsconfig.json
├── vitest.config.ts
└── i18n.config.json        # 默认配置
```

### 数据流

```
词条文件 → Loader → Validator → Exporter/Differ → 输出
```

### 测试策略

**测试框架**: vitest

**测试覆盖范围**:
- `loader.test.ts`: 词条文件加载、合并逻辑
- `validator.test.ts`: 各校验规则（唯一性、完整性、格式）
- `exporter.test.ts`: Excel 生成、格式转换
- `differ.test.ts`: 差异对比逻辑

**测试类型**:
- 单元测试：每个核心模块独立测试
- 集成测试：端到端命令测试
- Fixture: 使用 `tests/fixtures/i18n/` 提供测试数据

**测试命令**:
```bash
yarn test              # 运行所有测试
yarn test:watch        # 监听模式
yarn test:coverage     # 覆盖率报告
```

## Risks / Trade-offs

### 风险 1: Git 合并冲突

**风险**: 多人同时修改同一模块词条文件可能产生冲突

**缓解措施**:
- 提供清晰的冲突解决指南
- 考虑未来支持词条文件自动合并工具

### 风险 2: 键名命名规范不一致

**风险**: 开发者可能使用不一致的键名格式

**缓解措施**:
- 校验器强制执行键名格式规范
- 提供自动格式化工具（未来考虑）

### 风险 3: Excel 格式变更

**风险**: 公服系统可能变更导入格式

**缓解措施**:
- Excel 导出模块设计为可配置
- 支持自定义列名和格式

### Trade-off: 手动导入 vs 自动同步

**权衡**: 选择手动导入而非自动同步

**理由**:
- 降低复杂度和风险
- 保持人工审查环节
- 符合现有运维流程

## Migration Plan

### 部署步骤

1. **安装 CLI 工具**
   ```bash
   yarn global add i18n-cli
   # 或作为项目依赖
   yarn add -D i18n-cli
   ```

2. **初始化词条目录**
   ```bash
   i18n-cli init
   # 生成 i18n/ 目录和示例文件
   ```

3. **配置 yarn script**
   ```json
   {
     "scripts": {
       "i18n:validate": "i18n-cli validate",
       "i18n:export": "i18n-cli export",
       "i18n:diff": "i18n-cli diff"
     }
   }
   ```

4. **迁移现有词条**
   - 从公服系统导出当前词条
   - 使用 `i18n-cli import` 转换为代码仓库格式（未来功能）
   - 按模块拆分词条文件

5. **CI 集成**
   - 在 PR 流程中添加 `yarn i18n:validate` 校验
   - 确保合并主分支前词条质量
   - 使用 vitest 保证代码质量

### 回滚策略

- 如需回滚，直接从公服系统重新导出词条
- Git 历史保留所有变更记录，可随时恢复

## Open Questions

1. **是否需要支持词条嵌套结构？**
   - 当前假设是扁平键名（如 `ebankingPermission.login.title`）
   - 未来可考虑支持嵌套对象以提升可读性

2. **是否需要支持词条变量插值？**
   - 如 `Welcome, {{name}}!` 类型的动态词条
   - 暂不纳入 MVP，后续根据需求评估

3. **如何处理已废弃的词条？**
   - 是否需要标记机制？
   - 建议暂时直接删除，保留在 Git 历史中

4. **是否需要 Web 可视化工具？**
   - 当前仅提供 CLI
   - 后续可根据团队反馈考虑开发简单 Web 界面

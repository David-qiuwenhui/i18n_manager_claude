# Spec: I18N词条导出

## ADDED Requirements

### Requirement: 合并所有模块词条
系统 SHALL 收集所有模块的词条文件，并合并为统一的词条数据结构。

#### Scenario: 收集多模块词条
- **WHEN** 执行导出命令
- **THEN** 系统应扫描 `i18n/` 目录下所有 `.json` 文件
- **AND** 合并所有模块的 `zh-CN` 和 `en-US` 词条

#### Scenario: 按键名排序输出
- **WHEN** 合并词条数据
- **THEN** 系统应按键名字母顺序排序
- **AND** 确保输出结果的可读性和一致性

### Requirement: 生成 Excel 文件
系统 SHALL 将合并后的词条数据转换为公服系统可导入的 Excel 文件格式。

#### Scenario: 标准Excel格式
- **WHEN** 生成 Excel 文件
- **THEN** 文件应包含三列：键名、中文、英文
- **AND** 第一行为表头：`Key`, `zh-CN`, `en-US`

#### Scenario: Excel文件内容示例
| Key | zh-CN | en-US |
|-----|-------|-------|
| ebankingPermission.login.title | 用户登录 | User Login |
| ebankingPermission.login.button | 登录 | Login |
| channelPermission.create.title | 创建渠道 | Create Channel |

### Requirement: Excel 文件命名
系统 SHALL 使用时间戳生成的文件名，避免覆盖历史导出文件。

#### Scenario: 带时间戳的文件名
- **WHEN** 生成 Excel 文件
- **THEN** 文件名格式应为 `i18n-export-YYYYMMDD-HHmmss.xlsx`
- **AND** 示例：`i18n-export-20250215-143022.xlsx`

#### Scenario: 指定输出目录
- **WHEN** 执行导出命令时指定输出目录
- **THEN** 系统应将 Excel 文件保存到指定目录
- **AND** 默认输出到项目根目录

### Requirement: 导出命令
系统 SHALL 提供 `i18n-cli export` 命令用于执行导出操作。

#### Scenario: 基本导出命令
```bash
$ i18n-cli export
✅ Exported 150 entries to i18n-export-20250215-143022.xlsx
```

#### Scenario: 指定输出目录
```bash
$ i18n-cli export --output ./exports
✅ Exported 150 entries to ./exports/i18n-export-20250215-143022.xlsx
```

### Requirement: 导出前校验
系统 SHALL 在导出前自动执行词条校验，确保导出的数据质量。

#### Scenario: 校验失败不导出
- **WHEN** 执行导出命令
- **AND** 词条校验失败
- **THEN** 系统应显示校验错误信息
- **AND** 不生成 Excel 文件

#### Scenario: 校验通过正常导出
- **WHEN** 执行导出命令
- **AND** 词条校验通过
- **THEN** 系统应生成 Excel 文件
- **AND** 显示导出成功信息

#### Scenario: 跳过校验强制导出
```bash
$ i18n-cli export --skip-validation
⚠️  Skipping validation...
✅ Exported 150 entries to i18n-export-20250215-143022.xlsx
```

### Requirement: 导出统计信息
系统 SHALL 在导出完成后显示统计信息。

#### Scenario: 导出成功统计
```bash
$ i18n-cli export
✅ Export successful!
  - Total entries: 150
  - Files merged: 5
  - Output: i18n-export-20250215-143022.xlsx
```

#### Scenario: 按模块统计
```bash
$ i18n-cli export --verbose
✅ Export successful!
  - Total entries: 150
  - By module:
    * ebankingPermission.json: 52 entries
    * channelPermission.json: 31 entries
    * physicalPermission.json: 28 entries
    * common.json: 39 entries
  - Output: i18n-export-20250215-143022.xlsx
```

### Requirement: npm script 集成
系统 SHALL 支持通过 package.json 中的 script 命令调用导出功能。

#### Scenario: package.json 配置
```json
{
  "scripts": {
    "i18n:export": "i18n-cli export"
  }
}
```

#### Scenario: 通过 npm script 执行
```bash
$ npm run i18n:export

> i18n-cli export

✅ Exported 150 entries to i18n-export-20250215-143022.xlsx
```

### Requirement: Excel 文件编码
系统 SHALL 生成的 Excel 文件使用 UTF-8 编码，支持中文等多语言字符。

#### Scenario: Excel 文件字符编码
- **WHEN** 生成 Excel 文件
- **THEN** 文件应使用 UTF-8 编码
- **AND** 正确显示中文和特殊字符

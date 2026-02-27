# Spec: I18N词条校验

## ADDED Requirements

### Requirement: 跨模块键名唯一性校验
系统 SHALL 收集所有模块的词条信息，并校验键名在全局范围内的唯一性。

#### Scenario: 检测重复键名
- **WHEN** 执行校验命令
- **AND** 存在相同的键名出现在多个模块文件中
- **THEN** 系统应报错并提示重复的键名及其所在文件

#### Scenario: 无重复键名通过校验
- **WHEN** 执行校验命令
- **AND** 所有键名在全局范围内唯一
- **THEN** 系统应输出校验通过的信息

#### Scenario: 重复键名错误报告
```bash
$ i18n-cli validate
❌ Validation failed:
  - Duplicate key 'common.submit' found in:
    * i18n/ebankingPermission.json
    * i18n/common.json
```

### Requirement: 翻译完整性校验
系统 SHALL 确保每条词条同时存在中文和英文翻译。

#### Scenario: 缺少中文翻译
- **WHEN** 执行校验命令
- **AND** 某条键在 `en-US` 中存在但 `zh-CN` 中不存在
- **THEN** 系统应报错并提示缺少的中文翻译

#### Scenario: 缺少英文翻译
- **WHEN** 执行校验命令
- **AND** 某条键在 `zh-CN` 中存在但 `en-US` 中不存在
- **THEN** 系统应报错并提示缺少的英文翻译

#### Scenario: 翻译完整性错误报告
```bash
$ i18n-cli validate
❌ Validation failed:
  - Missing translation for 'ebankingPermission.login.title':
    * zh-CN: "用户登录" ✓
    * en-US: ✗ (missing)
```

### Requirement: 格式规范校验
系统 SHALL 校验词条文件的 JSON 格式正确性。

#### Scenario: JSON 格式错误
- **WHEN** 执行校验命令
- **AND** 某个词条文件的 JSON 格式不正确
- **THEN** 系统应报错并提示文件名和具体的格式错误位置

#### Scenario: JSON 格式正确
- **WHEN** 执行校验命令
- **AND** 所有词条文件的 JSON 格式正确
- **THEN** 系统应通过格式校验

### Requirement: 键名格式规范校验
系统 SHALL 校验词条键名是否符合点分隔的命名规范。

#### Scenario: 无效键名格式
- **WHEN** 执行校验命令
- **AND** 存在不使用点分隔格式的键名（如 `userLoginTitle`）
- **THEN** 系统应报错并提示不符合规范的键名

#### Scenario: 键名包含非法字符
- **WHEN** 执行校验命令
- **AND** 键名包含大写字母或特殊字符（除点号外）
- **THEN** 系统应报错并提示不符合规范的键名

#### Scenario: 键名格式错误报告
```bash
$ i18n-cli validate
❌ Validation failed:
  - Invalid key format 'ebankingPermissionLoginTitle':
    * Expected: lowercase with dots (e.g., 'ebankingPermission.login.title')
    * Found in: i18n/ebankingPermission.json
```

### Requirement: 全局词条收集
系统 SHALL 在执行校验前收集所有模块的词条信息。

#### Scenario: 收集所有词条文件
- **WHEN** 执行校验命令
- **THEN** 系统应扫描 `i18n/` 目录下所有 `.json` 文件
- **AND** 解析每个文件并合并所有词条信息

#### Scenario: 跳过非词条文件
- **WHEN** 扫描 `i18n/` 目录
- **AND** 目录中包含非 `.json` 文件
- **THEN** 系统应忽略这些文件

### Requirement: 校验命令输出
系统 SHALL 提供清晰的校验结果输出，包括成功/失败状态和具体的错误信息。

#### Scenario: 校验成功输出
```bash
$ i18n-cli validate
✅ Validation passed!
  - Total keys: 150
  - Files checked: 5
  - All translations complete
```

#### Scenario: 校验失败输出
```bash
$ i18n-cli validate
❌ Validation failed with 3 errors:
  1. Duplicate key 'common.submit' in i18n/ebankingPermission.json and i18n/common.json
  2. Missing en-US translation for 'channelPermission.create.title'
  3. Invalid key format 'ebankingPermissionLoginTitle' in i18n/ebankingPermission.json
```

### Requirement: 校验退出码
系统 SHALL 使用适当的退出码表示校验结果，便于 CI/CD 集成。

#### Scenario: 校验成功退出码
- **WHEN** 校验通过
- **THEN** 系统应返回退出码 0

#### Scenario: 校验失败退出码
- **WHEN** 校验失败
- **THEN** 系统应返回非零退出码（如 1）

# Spec: I18N词条存储

## ADDED Requirements

### Requirement: 按业务模块组织词条文件
系统 SHALL 支持按业务功能模块组织词条文件，每个模块对应一个独立的 JSON 文件。

#### Scenario: 创建 ebanking 模块词条文件
- **WHEN** 开发者创建 `i18n/ebanking.json` 文件
- **THEN** 系统应识别该文件为网银业务模块的词条文件
- **AND** 文件应包含网银相关的中英文词条键值对

#### Scenario: 创建 channel 模块词条文件
- **WHEN** 开发者创建 `i18n/channel.json` 文件
- **THEN** 系统应识别该文件为渠道业务模块的词条文件
- **AND** 文件应包含渠道相关的中英文词条键值对

#### Scenario: 创建 physicalPermission 模块词条文件
- **WHEN** 开发者创建 `i18n/physicalPermission.json` 文件
- **THEN** 系统应识别该文件为物理权限模块的词条文件
- **AND** 文件应包含物理权限相关的中英文词条键值对

#### Scenario: 创建 common 模块词条文件
- **WHEN** 开发者创建 `i18n/common.json` 文件
- **THEN** 系统应识别该文件为通用模块的词条文件
- **AND** 文件应包含通用功能的中英文词条键值对

### Requirement: 词条文件格式规范
每个词条文件 SHALL 遵循 JSON 格式，包含中英文两个顶级键，值为词条键值对对象。

#### Scenario: 标准词条文件格式
- **WHEN** 创建词条文件
- **THEN** 文件应包含 `zh-CN` 和 `en-US` 两个顶级键
- **AND** 每个键的值应为对象，包含该语言的词条键值对

#### Scenario: 词条文件示例
```json
{
  "zh-CN": {
    "ebanking.login.title": "用户登录",
    "ebanking.login.button": "登录"
  },
  "en-US": {
    "ebanking.login.title": "User Login",
    "ebanking.login.button": "Login"
  }
}
```

### Requirement: 词条键命名规范
词条键 SHALL 使用点分隔的命名空间格式，格式为 `<module>.<feature>.<action>`。

#### Scenario: 有效键名
- **WHEN** 定义词条键
- **THEN** 键名应使用小写字母和点号
- **AND** 格式应为 `模块.功能.操作` 如 `ebanking.login.title`

#### Scenario: 键名唯一性范围
- **WHEN** 在不同模块文件中定义词条
- **THEN** 相同的键名不应出现在多个文件中
- **AND** 系统应在校验时检测跨模块的重复键

### Requirement: 词条目录结构
项目 SHALL 使用 `i18n/` 目录存储所有词条文件。

#### Scenario: 标准目录结构
- **WHEN** 初始化词条管理
- **THEN** 应在项目根目录创建 `i18n/` 目录
- **AND** 所有词条文件应放在该目录下

#### Scenario: 多模块目录结构示例
```
i18n/
├── ebanking.json
├── channel.json
├── physicalPermission.json
└── common.json
```

### Requirement: 空词条文件处理
系统 SHALL 支持空的词条文件，允许逐步添加词条。

#### Scenario: 新模块空文件
- **WHEN** 创建新的模块词条文件
- **THEN** 文件可以只包含空的 `zh-CN` 和 `en-US` 对象
- **AND** 系统不应报错

### Requirement: 词条文件编码
词条文件 SHALL 使用 UTF-8 编码存储，支持中文等多语言字符。

#### Scenario: 文件编码检查
- **WHEN** 读取词条文件
- **THEN** 系统应使用 UTF-8 编码解析
- **AND** 应正确处理中文、特殊字符等

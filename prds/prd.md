# Skills Util - PRD

## 项目概述

### 项目名称
Skills Util (sutil)

### 项目类型
Node.js TypeScript 项目

### 项目背景
在开发 Agent Skill 时需要编写脚本，但经常遇到以下问题：
- 很多 Skill 脚本存在重复代码
- 每次调用 shell 代码可能不同，容易出错
- 缺乏统一的工具函数库

### 项目目标
提供最简化的调用形式，支持命令行和代码导入两种方式，减少重复代码，提高开发效率。

## 技术架构

### 技术栈
- Node.js
- TypeScript
- 单元测试框架：Vitest

### 调用方式

#### 命令行调用
```bash
sutil cmd 参数列表
```

#### 代码导入调用
```typescript
import { xx } from 'sutil'
```

## 设计原则

1. **双端支持**: 每个函数功能在命令行和脚本中都能够调用
2. **模块化组织**: 相同类型的功能放在同一个文件里（如文件操作放一个文件）
3. **独立调用**: 保证每一个功能都能被单独调用

## 功能需求

### 1. 全局配置管理

**配置文件位置**: 从调用位置开始往上级目录查找 `.sutil/config.json`
**配置文件格式**: JSON

**命令行示例**:
```bash
sutil set-config key value
sutil set-config key1 value1 key2 value2 key3 value3
sutil get-config key
sutil get-config key1 key2 key3
sutil list-configs
sutil delete-config key
sutil delete-config key1 key2 key3
```

**代码示例**:
```typescript
import { setConfig, getConfig, listConfigs, deleteConfig, setConfigs, getConfigs, deleteConfigs } from 'sutil'

setConfig('key', 'value')
const value = getConfig('key')
const allConfigs = listConfigs()
deleteConfig('key')

setConfigs({
  key1: 'value1',
  key2: 'value2',
  key3: 'value3'
})

const configs = getConfigs(['key1', 'key2', 'key3'])

deleteConfigs(['key1', 'key2', 'key3'])
```

### 2. 文件/目录删除

**功能**: 删除指定的目录或文件

**参数**:
- `--force, -f`: 强制删除，不需要确认
- `--recursive, -r`: 递归删除目录（删除目录时自动生效）
- `--verbose, -v`: 显示详细信息

**命令行示例**:
```bash
sutil delete-path /path/to/file_or_dir
sutil delete-path /path/to/dir --force --verbose
sutil delete-path /path/to/dir -frv
```

**代码示例**:
```typescript
import { deletePath } from 'sutil'

deletePath('/path/to/file_or_dir')
deletePath('/path/to/dir', { force: true, verbose: true })
```

### 3. 文件/目录复制

**功能**: 复制指定的目录或文件

**参数**:
- `--force, -f`: 强制覆盖目标（如果目标已存在）

**命令行示例**:
```bash
sutil copy-path /source/path /target/path
sutil copy-path /source/path /target/path --force
sutil copy-path /source/path /target/path -f
```

**代码示例**:
```typescript
import { copyPath } from 'sutil'

copyPath('/source/path', '/target/path')
copyPath('/source/path', '/target/path', { force: true })
```

### 4. JSON 文件读取

#### 4.1 读取 JSON 文件某个路径字段的内容

**参数**:
- `file`: JSON 文件路径
- `path`: 字段路径，使用 `.` 分隔，如 `user.name` 或 `items[0].id`

**命令行示例**:
```bash
sutil read-json-field /path/to/file.json user.name
sutil read-json-field /path/to/file.json items[0].id
```

**代码示例**:
```typescript
import { readJsonField } from 'sutil'

const value = readJsonField('/path/to/file.json', 'user.name')
const itemId = readJsonField('/path/to/file.json', 'items[0].id')
```

#### 4.2 读取 JSON 文件某个路径字段数组的第 x 个元素

**参数**:
- `file`: JSON 文件路径
- `path`: 数组字段路径，如 `users`
- `index`: 数组索引（从0开始）

**命令行示例**:
```bash
sutil read-json-array-item /path/to/file.json users 0
sutil read-json-array-item /path/to/file.json items 5
```

**代码示例**:
```typescript
import { readJsonArrayItem } from 'sutil'

const item = readJsonArrayItem('/path/to/file.json', 'users', 0)
const item = readJsonArrayItem('/path/to/file.json', 'items', 5)
```

#### 4.3 读取 JSON 文件某个路径字段数组的长度

**参数**:
- `file`: JSON 文件路径
- `path`: 数组字段路径

**命令行示例**:
```bash
sutil get-json-array-length /path/to/file.json users
sutil get-json-array-length /path/to/file.json items
```

**代码示例**:
```typescript
import { getJsonArrayLength } from 'sutil'

const length = getJsonArrayLength('/path/to/file.json', 'users')
const length = getJsonArrayLength('/path/to/file.json', 'items')
```

### 5. 目录文件读取

**功能**: 读取某个文件夹下指定目录的文件

**内置排除列表**: 默认自动排除以下目录和文件
- `node_modules`
- `.git`
- `.DS_Store`
- `dist`
- `build`
- `coverage`
- `*.log`

**参数**:
- `--recursive, -r`: 递归列出子目录的文件
- `--exclude, -e`: 额外排除的目录或文件（支持多个，逗号分隔）
- `--no-default-exclude`: 禁用默认排除列表
- `--ext, --extension`: 过滤文件类型（支持多个，逗号分隔）
- `--output, -o`: 将结果输出到指定文件（JSON 格式）

**输出格式（JSON）**:
```json
{
  "total": 10,
  "files": [
    {
      "path": "/path/to/file.ts",
      "name": "file.ts",
      "size": 1024,
      "modified": "2024-01-30T00:00:00Z"
    }
  ]
}
```

**命令行示例**:
```bash
sutil list-files /path/to/dir
sutil list-files /path/to/dir --recursive
sutil list-files /path/to/dir --recursive --exclude=.vscode,*.tmp
sutil list-files /path/to/dir --no-default-exclude
sutil list-files /path/to/dir --ext=ts,js --output result.json
sutil list-files /path/to/dir -r --no-default-exclude --ext ts,js -o result.json
```

**代码示例**:
```typescript
import { listFiles } from 'sutil'

const files = listFiles('/path/to/dir')
const files = listFiles('/path/to/dir', {
  recursive: true
})
const files = listFiles('/path/to/dir', {
  recursive: true,
  exclude: ['.vscode', '*.tmp']
})
const files = listFiles('/path/to/dir', {
  recursive: true,
  useDefaultExclude: false
})
const result = listFiles('/path/to/dir', {
  recursive: true,
  extensions: ['ts', 'js'],
  output: '/path/to/result.json'
})
```

## 项目结构

```
sutil/
├── src/
│   ├── index.ts           # 导出入口
│   ├── cli.ts             # 命令行入口
│   ├── config/            # 配置管理模块
│   │   └── index.ts
│   ├── file/              # 文件操作模块
│   │   ├── index.ts
│   │   ├── delete.ts
│   │   └── copy.ts
│   ├── json/              # JSON 操作模块
│   │   ├── index.ts
│   │   ├── read.ts
│   │   └── array.ts
│   └── list/              # 列表操作模块
│       └── index.ts
├── tests/                 # 单元测试
│   ├── config.test.ts
│   ├── file.test.ts
│   ├── json.test.ts
│   └── list.test.ts
├── package.json
├── tsconfig.json
└── README.md
```

## 非功能性需求

### 单元测试
- 测试框架：Vitest
- 为每个功能模块编写单元测试
- 测试覆盖率达到 90%
- 支持本地运行测试命令

### TypeScript 支持
- 完整的类型定义
- 类型安全的 API 设计

### 文档
- 提供使用文档
- 每个函数都有清晰的注释说明

### 错误处理
- 友好的错误提示信息
- 错误时显示堆栈跟踪便于调试
- 错误日志写入文件

### 日志输出
- 支持级别化日志（INFO/WARN/ERROR 等）
- 区分不同严重程度的日志输出

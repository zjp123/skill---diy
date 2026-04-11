# Quality Gate Command
# 质量门禁命令

Run the ECC quality pipeline on demand for a file or project scope.
按需对文件或项目范围运行 ECC 质量流水线。

## Usage
## 使用方法

`/quality-gate [path|.] [--fix] [--strict]`

- default target: current directory (`.`)
- 默认目标：当前目录（`.`）
- `--fix`: allow auto-format/fix where configured
- `--fix`：允许在已配置的地方自动格式化/修复
- `--strict`: fail on warnings where supported
- `--strict`：在支持的地方将警告视为失败

## Pipeline
## 流水线

1. Detect language/tooling for target.
1. 检测目标的语言/工具链。
2. Run formatter checks.
2. 运行格式化检查。
3. Run lint/type checks when available.
3. 在可用时运行 lint/类型检查。
4. Produce a concise remediation list.
4. 生成简洁的修复建议列表。

## Notes
## 注意事项

This command mirrors hook behavior but is operator-invoked.
此命令镜像钩子行为，但由操作员手动调用。

## Arguments
## 参数

$ARGUMENTS:
- `[path|.]` optional target path
- `[path|.]` 可选目标路径
- `--fix` optional
- `--fix` 可选
- `--strict` optional
- `--strict` 可选

# Test Coverage
# 测试覆盖率

Analyze test coverage, identify gaps, and generate missing tests to reach 80%+ coverage.
分析测试覆盖率、识别空白，并生成缺失的测试以达到 80%+ 覆盖率。

## Step 1: Detect Test Framework
## 步骤 1：检测测试框架

| Indicator | Coverage Command |
|-----------|-----------------|
| 检测标志 | 覆盖率命令 |
| `jest.config.*` or `package.json` jest | `npx jest --coverage --coverageReporters=json-summary` |
| `jest.config.*` 或 `package.json` jest | `npx jest --coverage --coverageReporters=json-summary` |
| `vitest.config.*` | `npx vitest run --coverage` |
| `vitest.config.*` | `npx vitest run --coverage` |
| `pytest.ini` / `pyproject.toml` pytest | `pytest --cov=src --cov-report=json` |
| `pytest.ini` / `pyproject.toml` pytest | `pytest --cov=src --cov-report=json` |
| `Cargo.toml` | `cargo llvm-cov --json` |
| `Cargo.toml` | `cargo llvm-cov --json` |
| `pom.xml` with JaCoCo | `mvn test jacoco:report` |
| `pom.xml` 含 JaCoCo | `mvn test jacoco:report` |
| `go.mod` | `go test -coverprofile=coverage.out ./...` |
| `go.mod` | `go test -coverprofile=coverage.out ./...` |

## Step 2: Analyze Coverage Report
## 步骤 2：分析覆盖率报告

1. Run the coverage command
1. 运行覆盖率命令
2. Parse the output (JSON summary or terminal output)
2. 解析输出（JSON 摘要或终端输出）
3. List files **below 80% coverage**, sorted worst-first
3. 列出**覆盖率低于 80%** 的文件，按最差优先排序
4. For each under-covered file, identify:
4. 对每个覆盖率不足的文件，识别：
   - Untested functions or methods
   - 未测试的函数或方法
   - Missing branch coverage (if/else, switch, error paths)
   - 缺失的分支覆盖（if/else、switch、错误路径）
   - Dead code that inflates the denominator
   - 使分母虚增的死代码

## Step 3: Generate Missing Tests
## 步骤 3：生成缺失测试

For each under-covered file, generate tests following this priority:
对每个覆盖率不足的文件，按以下优先级生成测试：

1. **Happy path** — Core functionality with valid inputs
1. **正常路径** — 使用有效输入的核心功能
2. **Error handling** — Invalid inputs, missing data, network failures
2. **错误处理** — 无效输入、缺失数据、网络故障
3. **Edge cases** — Empty arrays, null/undefined, boundary values (0, -1, MAX_INT)
3. **边界情况** — 空数组、null/undefined、边界值（0、-1、MAX_INT）
4. **Branch coverage** — Each if/else, switch case, ternary
4. **分支覆盖** — 每个 if/else、switch case、三元运算符

### Test Generation Rules
### 测试生成规则

- Place tests adjacent to source: `foo.ts` → `foo.test.ts` (or project convention)
- 将测试放在源文件旁：`foo.ts` → `foo.test.ts`（或遵循项目规范）
- Use existing test patterns from the project (import style, assertion library, mocking approach)
- 使用项目中已有的测试模式（导入风格、断言库、模拟方式）
- Mock external dependencies (database, APIs, file system)
- 模拟外部依赖（数据库、API、文件系统）
- Each test should be independent — no shared mutable state between tests
- 每个测试应相互独立——测试之间无共享的可变状态
- Name tests descriptively: `test_create_user_with_duplicate_email_returns_409`
- 用描述性名称命名测试：`test_create_user_with_duplicate_email_returns_409`

## Step 4: Verify
## 步骤 4：验证

1. Run the full test suite — all tests must pass
1. 运行完整测试套件——所有测试必须通过
2. Re-run coverage — verify improvement
2. 重新运行覆盖率——验证改善情况
3. If still below 80%, repeat Step 3 for remaining gaps
3. 若仍低于 80%，对剩余空白重复步骤 3

## Step 5: Report
## 步骤 5：报告

Show before/after comparison:
展示前后对比：

```
Coverage Report
──────────────────────────────
File                   Before  After
src/services/auth.ts   45%     88%
src/utils/validation.ts 32%    82%
──────────────────────────────
Overall:               67%     84%  PASS:
```

## Focus Areas
## 重点关注区域

- Functions with complex branching (high cyclomatic complexity)
- 具有复杂分支的函数（高圈复杂度）
- Error handlers and catch blocks
- 错误处理程序和 catch 块
- Utility functions used across the codebase
- 跨代码库使用的工具函数
- API endpoint handlers (request → response flow)
- API 端点处理程序（请求 → 响应流程）
- Edge cases: null, undefined, empty string, empty array, zero, negative numbers
- 边界情况：null、undefined、空字符串、空数组、零、负数

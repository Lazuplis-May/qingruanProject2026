# 1. 单元测试目标

在本次软件开发任务中，你需要针对项目代码编写单元测试，确保核心功能的正确性。

后端项目使用 **JUnit 5** 进行单元测试；

前端项目使用 **Vitest** 进行单元测试。

通过本实验，你将：

✔ 掌握单元测试框架的基本用法，编写规范的测试用例

✔ 学会使用断言（Assertion）验证代码逻辑是否符合预期

✔ 提高代码质量，通过测试发现潜在 Bug

✔ 适应团队开发规范，确保提交的代码经过充分测试

# 2. 测试工具与环境

## （1）后端项目

### 所需工具

- JUnit 5（Java 单元测试框架）
- IntelliJ IDEA / Eclipse（推荐 IntelliJ IDEA）
- Maven（依赖管理工具）

### 环境配置

确保项目已引入 JUnit 依赖：

```
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.9.2</version>
    <scope>test</scope>
</dependency>
```

测试代码存放位置：

```
src/test/java
```

与：

```
src/main/java
```

对应。

## （2）前端项目

### 所需工具

- Vitest（前端单元测试框架）
- Node.js
- npm 或 pnpm
- IntelliJ IDEA / VS Code

### 环境配置

安装 Vitest：

```
npm install -D vitest
```

或：

```
pnpm add -D vitest
```

在 package.json 中添加测试命令：

```
{
  "scripts": {
    "test": "vitest"
  }
}
```

测试代码建议存放位置：

```
src/**/*.test.js
src/**/*.test.ts
src/**/*.spec.js
src/**/*.spec.ts
```

例如：

```
src/utils/user.test.ts
```

# 3. 测试步骤指南

## （1）创建测试类（后端）

测试类命名规范：

例如：

```
UserService.java
```

对应：

```
UserServiceTest.java
```

示例：

```
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class UserServiceTest {

    @Test
    void testLoginSuccess() {
        // 测试代码
    }
}
```

## （2）创建测试文件（前端）

例如：

```
user.ts
```

对应：

```
user.test.ts
```

示例：

```
import { describe, it, expect } from 'vitest'

describe('sum', () => {
    it('should return correct result', () => {
        expect(1 + 1).toBe(2)
    })
})
```

## （3）编写测试用例

### 正常情况测试

验证方法在正确输入下的输出。

后端示例：

```
@Test
void testAddUser() {
    UserService service = new UserService();
    boolean result =
        service.addUser("Alice", "alice@example.com");

    assertTrue(result);
}
```

前端示例：

```
it('should add two numbers', () => {
    expect(add(1, 2)).toBe(3)
})
```

### 异常情况测试

验证方法对非法输入的处理。

后端示例：

```
@Test
void testAddUserWithInvalidEmail() {
    UserService service = new UserService();

    assertThrows(
        IllegalArgumentException.class,
        () -> service.addUser("Bob", "invalid-email")
    );
}
```

前端示例：

```
it('should throw error', () => {
    expect(() => validateEmail('abc'))
        .toThrow()
})
```

### 边界条件测试

验证空值、极限值等情况。

后端示例：

```
@Test
void testLoginWithEmptyPassword() {
    UserService service = new UserService();

    assertFalse(
        service.login("admin", "")
    );
}
```

前端示例：

```
it('should return false for empty password', () => {
    expect(login('admin', ''))
        .toBe(false)
})
```

## （4）常用断言方法

### JUnit

```
assertEquals(expected, actual)
assertTrue(condition)
assertFalse(condition)
assertNull(object)
assertThrows(Exception.class, executable)
```

### Vitest

```
expect(actual).toBe(expected)
expect(actual).toEqual(expected)
expect(condition).toBeTruthy()
expect(condition).toBeFalsy()
expect(value).toBeNull()
expect(fn).toThrow()
```

## （5）运行测试

### 后端项目

IDE：

```
右键测试类 → Run Tests
```

命令行：

```
mvn test
```

### 前端项目

命令行：

```
npm run test
```

或：

```
pnpm test
```

确保所有测试通过（✅）。

# 4. 测试覆盖率要求

## 后端项目

- 核心功能（如用户注册、登录、数据查询等）必须达到 100% 覆盖
- 辅助功能（工具类等）覆盖率不低于 80%

推荐工具：

- JaCoCo
- IntelliJ IDEA Coverage

## 前端项目

- 核心业务逻辑必须达到 100% 覆盖
- 工具函数及公共模块覆盖率不低于 80%

推荐工具：

- Vitest Coverage
- Istanbul
- V8 Coverage

示例：

```
vitest run --coverage
```

# 5. 提交要求

## 代码仓库

确保提交完整测试代码：

后端：

```
src/test/java
```

前端：

```
*.test.js
*.test.ts
*.spec.js
*.spec.ts
```

## 测试报告

提交测试覆盖率截图：

- JaCoCo 报告（后端）
- Vitest Coverage 报告（前端）

## 文档说明

需说明：

- 如何运行测试（如 mvn test、npm run test）
- 测试覆盖率情况
- 主要测试内容说明

未编写测试或测试覆盖率明显不足的项目，将影响实验成绩评定。
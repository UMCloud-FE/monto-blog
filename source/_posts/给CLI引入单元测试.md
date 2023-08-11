---
layout: post
title: 给CLI引入单元测试
date: 2023-07-27 08:48:11
tags: CLI
---

这一节来讲如何给CLI引入单元测试

自动化测试类型有三种
- Unit: 单元测试
- Integration: 集成测试
- End-to-End: 端到端测试

这里我们选择单元测试对CLI进行检查，保证代码的稳定性。

> 单元测试，我们直接选择当前最流行的jest框架。jest也有CLI工具，向我们的CLI一样，提供各种测试命令

## 一、给项目配置jest

**安装**
`yarn add -D jest`或`npm install jest -D`

**配置(非必需)**

如果你想获得更多的jest配置，可以安装配置，安装方法也很简单`yarn jest --init`，执行命令后出现

```
√ Choose the test environment that will be used for testing » node
√ Do you want Jest to add coverage reports? ... yes
√ Automatically clear mock calls and instances between every test? ... yes
```

根据下面提示选择即可，这里我们选择node环境，需要代码测试覆盖率报告，自动清除每个单元测试之间的模拟调用和实例。

执行完成后，我们发现在项目根目录下多了一个`jest.config.js`文件，里面包含了各种配置说明

**快速执行**

package.json

```json
{
  "scripts": {
    "start": "node index.js",
    "test": "jest",
    "coverage": "jest --coverage"
  }
}
```
执行命令启动单元测试`yarn test`或`yarn coverage`

## 二、如何开始写单元测试

> 首先，不用紧张，单元测试入门很简单，掌握用法即可

比如我们有sum.js

```js
export function sum(a, b){
    return a + b;
}
export function mins(a, b){
    return a - b;
}
```
为这个函数写测试文件

```js
import { sum, mins } from './sum'
it(`add 1 + 2 to equal 3`, () => {
    expect(sum(1, 2)).toBe(3);
});
test(`mins 2 - 1 to equal 1`, () => {
    expect(min(2, 1)).toBe(1);
})
```

入门就是这么简单，只需要针对每个函数做一些预期的校验即可，当不小心改动了源代码导致输出的结果和预期不符，将会测试不通过，这样就保证了代码功能的稳定。

在项目中，我们通常有两种管理单元测试的方法
- 1、在项目根目录下新建tests目录，将单元测试文件放在其中，测试文件命名`xx.test.js`，优点是可以更好的管理测试文件，缺点是不好找到源文件
- 2、在对应文件的目录下新建`__test__`目录，测试文件放置其中，优点就是容易找到执行文件，但不容易过滤和管理

**注意**

当然这里只是最简单的单元测试，Jest框架还提供了其他的能力，比如生成快照、模拟请求、类型判断等，可以参考[Jest](https://jestjs.io/zh-Hans/docs/getting-started)

## 三、查看单元测试的结果

单元测试完成后，执行测试命令

`yarn test` 或 `npx jest`

![jest测试结果](/images/cli/jest.png)

**结果解读**

- Stmts (Statements)：语句覆盖率，即被测试覆盖的代码语句的百分比。在你的代码中，43.63% 的语句被测试覆盖。
- Branch：分支覆盖率，即被测试覆盖的条件分支的百分比。在你的代码中，53.33% 的分支被测试覆盖。
- Funcs (Functions)：函数覆盖率，即被测试覆盖的函数的百分比。在你的代码中，30% 的函数被测试覆盖。
- Lines：行覆盖率，即被测试覆盖的代码行数的百分比。在你的代码中，44.44% 的行被测试覆盖。
- Uncovered Line #s：未覆盖的行号。这一列列出了未被测试覆盖的代码行的行号范围。
- Test Suites: 4 passed, 4 total：这表示你有 4 个测试套件，其中所有的 4 个测试套件都通过了。
- Tests: 9 passed, 9 total：这表示你一共运行了 9 个测试，其中所有的 9 个测试都通过了。
- Snapshots: 0 total：这表示没有使用快照测试（Snapshot Testing），因此没有生成任何快照。
- Time: 1.497 s, estimated 2 s：这表示测试运行的时间为 1.497 秒，估计耗时为 2 秒。

## 四、在推送前执行单元测试

暂无
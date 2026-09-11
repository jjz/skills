---
name: dev-testing-zh
description: 跨语言规划与编写单元、集成、端到端测试——测试金字塔（L1 单元 / L2 集成 / L3 E2E）、与 docker-compose 绑定的集成测试、各语言运行器（Jest、Vitest、Bun test、pytest、Go、Rust、Solidity）、Playwright E2E，以及 PDPO 安全的合成测试数据。用于为某次改动选择该测什么、编写或审查测试，或排查不稳定/过慢的套件。
---

# Dev 测试 — 单元、集成与端到端

*[English version →](../dev-testing/SKILL.md)*

权威来源（这些会更新，不要凭记忆）：
[Martin Fowler — Test Pyramid](https://martinfowler.com/bliki/TestPyramid.html) ·
[Kent C. Dodds — Testing Trophy](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications) ·
[Jest](https://jestjs.io/) ·
[Vitest](https://vitest.dev/) ·
[Bun test](https://bun.sh/docs/cli/test) ·
[pytest](https://docs.pytest.org/) ·
[Playwright](https://playwright.dev/) ·
[Testcontainers](https://testcontainers.com/) ·
[Go testing](https://pkg.go.dev/testing) ·
[rust test](https://doc.rust-lang.org/rustc/tests/index.html) ·
[Foundry](https://book.getfoundry.sh/) ·
[Hardhat](https://hardhat.org/)

测试套件是设计工具，不是打勾清单。单元、集成、端到端三层回答不同的问题、成本各异，而大多数套件失败的原因，是拿其中一层去干另一层的活（会连数据库的"单元"测试、"集成"测试却 mock 了它本应验证的东西、给每个 HTTP 接口各写一条 E2E）。下面每一节都要过一遍，不要只看和这次请求"看起来相关"的那一节。

## 1. 三层定义（以及如何选择）

| 层级 | 回答的问题 | 依赖 | 速度 | Mock | 典型占比 |
|------|-----------|------|------|------|---------|
| **L1 单元** | "这段逻辑对不对？" | 无（同进程） | 毫秒 | mock 所有外部 I/O | 多 |
| **L2 集成** | "这些零件真的装得起来吗？" | 真实 DB/缓存/消息队列，docker-compose | 秒 | 不 mock 被测基础设施 | 较少 |
| **L3 E2E** | "用户旅程走得通吗？" | 全栈 + 浏览器 | 秒–分钟 | UI 路径上不造假 | 最少 |

按**改动碰了什么**来选择，而不是按哪个好写：

- 改了纯函数 / 校验 / 单模块分支逻辑 → **L1**。
- 改了 HTTP 处理器、ORM 查询、迁移、Guard/中间件，或应用与真实依赖的装配方式 → **L2**。
- 改了完整用户流程、关键 UI，或跨服务集成 → **L3**。
- **改什么，测什么**：优先在 bug 真正可能存在的层写测试。L1 已覆盖的逻辑不要再用 L2 重复覆盖，也不要给每个 API 路由都写 L3。

## 2. 单元测试（L1）

- **同进程且快**：不连数据库、不联网、不碰文件系统、不碰时钟（除非注入）。一条"单元"测试若要环境才能过，它其实是伪装成单元的集成测试——把它往上挪一层。
- **在边界处 mock，不 mock 内部**：stub 掉单元依赖的*端口*（HTTP 客户端、DB 适配器、LLM 客户端、时钟、随机源），而不是私有辅助函数。mock 内部实现会把测试和实现细节绑死，让重构莫名其妙地红。
- **确定性**：注入时间和随机数，不要内联 `Date.now()` / `Math.random()`；一条"时过时不过"的测试比没有测试更糟。
- **Arrange–Act–Assert**，每个*行为*一个断言（不是每行一个）。测试名描述行为而非实现：`余额不足时返回错误`，而不是 `test_process_1`。
- **F.I.R.S.T.**：Fast（快）、Independent（独立，不依赖别的测试的副作用）、Repeatable（可重复）、Self-validating（断言而非 `console.log`）、Timely（及时写）。

## 3. 集成测试（L2）— docker-compose 生命周期

集成测试验证的是*装配*：对真实数据库的 schema/迁移、真实缓存/消息队列的往返、真实 HTTP 上的 Guard 与中间件。唯一要命的规则：

> **绝不 mock 该测试本应验证的基础设施。** 如果测试是关于数据库的，它就得连真库。若你用 `jest.mock` 把 repository 换掉来"提速"，那就不是集成测试——只是加了戏的单元测试。

- **把 compose 绑到测试生命周期**——用 `up` 起，务必用 `stop`/`down` 收，失败也要收：
  - `docker-compose up -d <service>`（只起本套件需要的服务）→ 跑测试 → `docker-compose stop`/`down`。
  - 用 shell `trap` / 全局 `beforeAll` + `afterAll` / 测试运行器的 fixture，保证 Ctrl-C 和异常也能清理。"只为本次测试开"的端口长期占用，会和下次运行撞车。
- **幂等** `up -d`：配置不变时重复 up 不能制造多套实例，也不能抢端口。
- **compose 文件跟着被测包走**（如 `docker-compose.test.yml`），随代码版本化，让 CI 里的 DB/缓存版本与本地、线上一致（或声明兼容）。
- **连之前先等健康检查**：等容器健康检查（或对端口做重试循环），不要固定 sleep——固定延迟既不稳定又慢。
- **优先用打包好的命令**：跑包自己那个封装了 `up → test → down` 的 `test` 脚本，而不是每次手搓生命周期。
- **没有 Docker 的 CI**：要么只跑 L1，要么用 **service 容器**（GitHub Actions `services:`、GitLab `services:`）替代 compose——但**二选一**，compose 或 service 容器，绝不能两个都上（双实例）。连接串与端口必须和 compose 映射一致。
- **Testcontainers** 是运行器支持时更干净的替代（Java、Go、Python、Node 都支持）：把同样的 compose/容器生命周期封装进代码，setup/teardown 不可能被跳过。

## 4. 端到端测试（L3）— Playwright

E2E 是**少数几条高价值的用户旅程**，不是在浏览器里把所有接口重新测一遍。

- **驱动真实浏览器**（Playwright）打组装好的栈。优先用面向用户的定位器（`getByRole`、`getByLabel`、`getByText`），少用脆弱的 CSS/XPath；用 `data-testid` 属性，而不是依赖 DOM 位置。
- **状态隔离**：每个场景自己造数据（或用已知 fixture）、事后清理，这样测试可以任意顺序、并行运行。
- **分清"不稳定"和"坏了"**：重试是安全网，不是修复手段——需要靠重试才过的测试掩盖了真实的竞态；修根因（等一个条件，而不是等一个超时），别只调大重试次数。
- **走公开接口**：不要绕过鉴权，不要在产品路径是 UI 表单时用私有 DB 写入造数——E2E 应演练用户真正会做的事，否则它证明不了旅程本身。
- **保持小规模、晚点跑**：E2E 是最慢最贵的一层——只覆盖关键旅程，在 CI 里排在 L1/L2 通过之后；绿 build 不能替代实际核对状态码/行为。

## 5. 各语言运行器

上面的金字塔与语言无关，运行器才因语言而异。跟随该包已有约定，而不是硬套一个。

### TypeScript / JavaScript

- **Jest** —— 长生命周期的 Node/NestJS 后端默认项；TS 用 `jest.config` + `ts-jest`，`describe/beforeAll/it`，必要时用快照测试。
- **Vitest** —— 包基于 Vite 时的默认项（Next/React 前端、Vite 库）；API 兼容 Jest、更快、ES 模块原生。
- **Bun test** —— Bun 内置运行器（`bun test`）；API 兼容 Jest 但*不是* Jest——别默认每个 Jest 插件/glob 都能用。
- **每个包只选一个**：monorepo 里一个包用 Jest、另一个包用 Vitest 是合理的——跟着包走，别随意迁移，更别把一个包的运行器配置套到另一个包。
- 一律通过 **`bun run <script>`**（或该包声明的包管理器）运行；脚本才是"用哪个运行器、哪份配置"的唯一来源。

### Python

- **pytest**（经项目环境管理器运行，如 `uv run pytest`）：用 `fixture` 做共享 setup/teardown，`@pytest.mark.parametrize` 做表驱动用例，`tmp_path`/`monkeypatch` 做隔离，`pytest-asyncio` 做异步。
- 优先 fixture 而非 `setUp`/`tearDown`；带 `yield` 的 fixture 能给出干净的 teardown。fixture 作用域越窄越好（`function` 优于 `session`）。

### Go（预留）

- 标准 `testing` 包：表驱动的子测试（`t.Run`）、`go test ./...`、`-race`、`-cover`。断言/套件/mock 用 `testify`。集成测试用 **Testcontainers-go** 或 compose + 一个只 setup/teardown 一次的 `TestMain`。

### Rust（预留）

- `cargo test`：`#[test]`，异步/参数化用 `#[tokio::test]`/`#[rstest]`。`assert!`/`assert_eq!`/`assert_matches!`。集成测试放 `tests/`（面向公开 crate API），单元测试用 `#[cfg(test)]` 模块。容器用 `testcontainers` crate。

### Solidity / EVM（预留）

- **Foundry**（`forge test`）是合约测试默认项：Solidity 测试合约、`vm.expectRevert`、模糊测试（`testFuzz_`）、不变量测试、`forge snapshot` 看 gas。**Hardhat** 用于 JS/TS 原生工作流（mocha + ethers + `hardhat test`）。
- 必覆盖：revert 路径（`vm.expectRevert`）、访问控制（onlyOwner/roles）、溢出/下溢（Solidity ≥0.8 默认检查）、以及金融合约的**不变量测试**——只有在模糊序列下才被破坏的不变量，正是资金损失类 bug 的藏身处。

## 6. 测试数据与环境

- **绝不在 fixture 里用真实个人数据。** 生产环境的 PII（身份证、电话、住址、KYC 文档、与个人绑定的钱包地址）是受规管数据——fixture 里带真实客户数据等于埋下一个数据泄露隐患。用合成/匿名生成器（faker/polygon），从生产拷来的 fixture 一律脱敏。
- **工厂优于手写种子数据**：用工厂/构造器构建测试所需的最小对象，只覆盖场景关心的字段。这能保持测试可读，也避免无关断言被绑到共享 fixture 的某个具体值上。
- **清理副作用**：测试写入的东西（行、文件、缓存）应在 teardown 中删除，或每次运行隔离（唯一 ID、每次运行独立 schema、`tmp_path`）。
- **连接串与端口用同一份配置源**（env / `.env.test`），与 compose 映射一致——硬编码端口会漂移，两个套件并行时就悄悄崩了。

## 7. 最佳实践摘要

- **正确 > 快，快 > 全**：10 毫秒跑完的 L1 每次保存都会跑；几分钟的 L2 只在 CI 跑——偏向快的那层，风险真需要时才去够慢的那层。
- **测试独立、无顺序依赖**：测试间无共享可变状态，没有"必须先跑某条"的测试。
- **覆盖率是信号，不是目标**：追*为什么*（一条关键分支、一条错误路径、一个你刚修的回归——修 bug 必补回归测试），不是追*数字*。80% 是地板不是目标；一条没覆盖的关键路径比一个被覆盖的 getter 更重要。
- **断言真实行为**：断言 status/`errorCode`/可观察输出，不是 `console.log` 或"没抛异常"。业务失败要保持单一来源的契约（如 `POINTS_ERRORS` 表），`throw` 与测试共用——绝不在测试里硬编码 `402` / `'INSUFFICIENT_BALANCE'` 字面量。
- **超时与错误路径是一等公民**：异步/流式代码要有超时，错误路径要断言，不能只有 happy path。流式接口应断言 chunk 边界与流结束标记。
- **CI 门禁**：合并前 `lint → build → test`；先跑 L1、再 L2、最后小规模 E2E，保持反馈快。失败要可读地 fail build，而不是甩一段堆栈。

## 8. 改动清单

1. 给改动分类：L1、L2 还是 L3——选*对*的层，不是选好写的那层。
2. L1：同进程、mock 外部 I/O、注入时间/随机、按行为命名。
3. L2：经 compose（或 service 容器/Testcontainers）接真实依赖，`up → test → down` 且失败也清理，连之前等健康检查，不 mock 基础设施。
4. L3：只覆盖关键旅程，浏览器驱动，状态隔离，面向用户的定位器，修 flaky 而不是调大重试。
5. 匹配该包的运行器（Jest/Vitest/Bun test/pytest/Go/Rust/Foundry）——别跨包套配置。
6. 测试数据：合成、工厂构建、事后清理、无真实 PII。
7. 每个修掉的 bug 补一条回归测试；跑 `lint → build → test`（dev 常驻时 `lint → test`），并在 CI 里设为门禁。

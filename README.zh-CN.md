# LinkChat 文档与研究材料

**中文** | [English](README.md)

LinkChat Documents 是 LinkChat v1.0 的语言无关研究配套仓库，收录协议规范、
密码学配置、公开一致性向量、设计说明和研究论文。它使研究者、审阅者或独立实现者
不依赖某个 Rust API 或内存布局，也能研究、审查和复现协议。

本仓库面向密码学研究者、协议设计者、形式化方法研究者、实现者和审阅者。可执行的
Rust 内核与 Lean 工件位于配套的
[`LinkChat`](https://github.com/SkmdysK/LinkChat) 仓库。

## 研究问题

LinkChat 研究一种严格交替的端到端协议：对端能够提交经认证的输入，却不能选择诚实
端点的下一个私有 Receiver State。每个端点拥有私有 Receiver Package，并发布与之
密码学绑定的公共投影。只有有效输入可以推进一个应用轮次；格式错误、重放、过期、
未来轮次或上下文不合法的输入不得推进状态。

文档将下列对象作为可独立审计的边界：

- 端点本地私有 Receiver State 与公开 Receiver Package；
- 严格 Alice/Bob 应用轮次与重放语义；
- identity、session、cipher suite、token、turn、generation 与 package hash 绑定；
- canonical encoding、重新编码检查与固定 4096-byte Envelope；
- X25519 与 ML-KEM-768 混合密钥建立；
- 域分离 HKDF-SHA-256 以及认证 Header/Message 上下文；
- prepare、durable commit、recover 和诚实的 rollback 能力声明；
- 不得推进应用状态的 transport 与 mailbox 行为；
- 形式化对应关系、有界一致性和安全游戏义务。

## 阅读路径

### 协议与实现审查

1. [`SUMMARY.md`](SUMMARY.md)
2. [`spec/00-overview.md`](spec/00-overview.md)
3. [`spec/04-cryptographic-profile.md`](spec/04-cryptographic-profile.md)
4. [`spec/05-wire-format.md`](spec/05-wire-format.md)
5. [`spec/06-state-machine.md`](spec/06-state-machine.md)
6. [`spec/08-storage-and-recovery.md`](spec/08-storage-and-recovery.md)
7. [`spec/11-conformance.md`](spec/11-conformance.md)

### 密码学与形式化研究审查

1. [`spec/09-security-model.md`](spec/09-security-model.md)
2. [`spec/10-formal-verification.md`](spec/10-formal-verification.md)
3. [`paper/LinkChat-Protocol-Research.pdf`](paper/LinkChat-Protocol-Research.pdf)
4. 配套实现仓库中的 `formalization/` 目录

### 独立实现

建议按“术语、密码学配置、wire format、状态机、存储/恢复、一致性要求”的顺序阅读。
不要从宿主语言对象布局或单个向量反推 wire format；应先实现规范性要求，再使用公开
向量做交叉验证。

## 密码学配置

| 边界 | 标准构造 |
| --- | --- |
| 身份认证 | Ed25519 |
| 经典密钥协商/KEM 包装 | X25519 |
| 后量子密钥封装 | ML-KEM-768 |
| 密钥派生 | 域分离 HKDF-SHA-256 |
| Package/Transcript 摘要 | 对 canonical encoding 计算 SHA-256 |
| Header 与 Payload 保护 | ChaCha20-Poly1305 |
| 可选 MAC 边界 | HMAC-SHA-256 |
| 新鲜随机数 | OS CSPRNG |

规范定义这些原语的组合、绑定、编码和失败语义，但不替代对标准原语的安全分析、实现
审查或参数选择指导。

## 仓库结构

| 路径 | 用途 |
| --- | --- |
| `SUMMARY.md` | 研究总览与阅读指南 |
| `spec/` | 规范性与解释性协议章节 |
| `vectors/` | 公开 JSON 一致性向量与清单 |
| `paper/LinkChat-Protocol-Research.tex` | 研究论文 LaTeX 源码 |
| `paper/LinkChat-Protocol-Research.pdf` | 渲染后的研究论文 |
| `CHANGELOG.md` | 研究材料的变更记录 |

规范章节覆盖术语、架构、身份/session 绑定、密码学、wire format、状态转换、
transport/mailbox 契约、存储/恢复、安全模型、形式化验证、一致性、版本化和完整示例。

## 证据与声明边界

项目明确区分四类证据：

1. **规范性要求：**规范定义必需字段、编码、上下文绑定、状态规则、存储顺序和集成
   义务。
2. **机器检查的结构性结果：**配套 Lean 模型检查特定状态机、package 轮换、非干扰、
   安全游戏接口和归约记账性质。
3. **可执行一致性证据：**Rust 工作区提供向量、单元/属性测试、fuzz smoke、崩溃矩阵
   和有界差分检查。
4. **外部假设：**标准原语安全性、操作系统随机数、安全擦除、侧信道、rollback
   resistance 和部署架构仍属于外部假设。

有限测试覆盖和当前包含的符号/形式模型都不能单独证明：任意具体实现或部署能够抵抗
每一个概率多项式时间对手。文档明确保留这些限制，以便采用者在正确威胁模型下评估协议。

## 范围与非目标

本仓库不定义或运营公开服务器、DHT、中继、生产 Mailbox、账户系统、GUI、聊天客户
端、流量分析防御或 key-transparency 基础设施。这些属于独立部署与集成问题，不能从
协议内核中推断为已解决。

文档描述的是一个冻结的研究内核，不代表无条件的生产就绪或完整密码学安全保证。

## 许可证

本项目采用 MIT License。可执行代码的许可证和贡献规则请参阅配套实现仓库。

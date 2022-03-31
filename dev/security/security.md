# Security

# **通过准确性、隔离性和模糊性实现安全性**

从历史上看，有 [3 种主要的方法](https://theinvisiblethings.blogspot.com/2008/09/three-approaches-to-computer-security.html) 来保护计算机系统。 我们如何将这些方法应用于 zkSync 和区块链？

## **1. 准确性**

准确性是指尽可能主动地生产没有错误或恶意行为代码的软件。 我们通过遵循行业的最佳实践来做到这一点。构建由 Trail of Bits 维护的[安全合约](https://github.com/crytic/building-secure-contracts)、由 Consensys Diligence 维护的[智能合约最佳实践](https://consensys.github.io/smart-contract-best-practices/)以及由 CIA 官员维护的[安全和安全研究基地](https://github.com/OffcierCia/ultimate-defi-research-base#security--safety)是记录行业从生产测试中学习的众多重要资源中的三个。 过去几年。 我们的合同和密码学也接受内部和[外部审计](https://zksync.io/updates/security-audits.html)。

## **2. 隔离性**

隔离安全的目标是将系统分成更小的部分，以尽量减少一个部分的故障对所有其他部分的影响。 遵循这一理念，我们的智能合约被拆分和设计，假设所有其他部分都可能是拜占庭式的——不可靠、有缺陷或受到损害。

此外，我们相信通过增加冗余可以进一步加强隔离的安全性。 在 zkSync 中，所有提交的交易在包含在区块中之前都通过简单的执行进行验证。 因此，如果我们的 ZK 电路存在漏洞，攻击者需要同时突破密码学 * 和 ***** 序列器（我们的服务器目前，但最终切换到多验证者 PoS 共识）。

## **3. 模糊性**

这种方法依赖于默默无闻，使攻击者的生活尽可能艰难，但区块链文化的基础是对开发人员和用户的彻底透明，并得到我们强大的白帽黑客社区的补充。

为了在安全保密和透明度之间取得平衡，我们致力于以下事项：

- **更新**: 所有升级代码都会在上线前一个月公开。 这使用户有足够的时间选择退出升级，并且白帽社区可以通过激励黑客网络以不信任的方式查找错误——这是一个独立的升级主网实例，由 Matter Labs 内部提供的赏金。
- **审计**: 完整的审计报告发布在我们的[文档](https://zksync.io/updates/security-audits.html)中。
- **漏洞披露**: 在部署修复程序之前，必须静默处理所有漏洞，并且 Matter Labs 团队不仅对根本原因进行了彻底的内部审查，而且还对整体安全设计中的潜在薄弱环节进行了彻底的内部审查。 在系统更改完全实施后，将发布每个漏洞的公开事后分析。 请查看我们的[完整的漏洞披露政策](https://zksync.io/dev/security/disclosure.html)和更新的[已知漏洞列表](https://zksync.io/dev/security/bugs.html)。
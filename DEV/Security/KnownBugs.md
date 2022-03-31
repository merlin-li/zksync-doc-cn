# KnownBugs

您将在下面找到 JSON 格式的已知安全漏洞列表。 该文件本身托管在此 GitHub 中。 此列表于 2022 年 1 月 1 日开始，涵盖版本 5.2 及更高版本。

已知漏洞的 JSON 文件是一个对象列表，每个漏洞对应一个对象，具有以字段：

**name**: 漏洞的唯一名称

**uid**: 唯一标识，格式： **ZKSYNC1-<year>-<sequential id>**

**summary**: 简短描述

**description**: 详细

**links**: 具有更详细信息的相关 URL 列表（可选）

**introduced**: 第一个发布的包含漏洞的zkSync版本

**fixed**: 第一个发布的 zkSync 版本不再包含漏洞

**severity**: 漏洞的严重性：低、中、高、严重，考虑到影响的严重性和被利用的可能性

```json
  [
    {
      "name": "SELFDESTRUCT main via delegatecall",
      "uid": "ZKSYNC1-2021-01",
      "summary": "The Proxy’s target code allowed setting the main zkSync contract to SELFDESTRUCT, resulting in a freeze of user funds.",
      "description": "The initialize function in the zkSync main contract could be called on the target contract with any parameters at any time, allowing anyone to set additionalZkSync in the target contract storage to any address. If the attacker sets additionalZkSync to an address that would execute the SELFDESTRUCT opcode on any entry, and then call any function on the zkSync main contract that uses logic from additionalZkSync via delegatecall, the main zkSync target contract could have been destroyed and all funds would have been frozen. Funds could not be stolen because the Proxy contract owns the rollup assets and it did not contain a vulnerability, only the code of the Proxy’s target.",
      "links": "https://zksync.io/dev/security/ZKSYNC1-2021-01",
      "introduced": "v5.1",
      "fixed": "v5.2",
      "severity": "Critical"
    }
  ]
```

在 [这里](https://merlin-li.github.io/dev/security/ZKSYNC1-2021-01.html) 获取更详细信息。
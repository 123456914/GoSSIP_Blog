# Understanding and Securing Device Vulnerabilities through Automated Bug Report Analysis

> 作者: Xuan Feng, Xiaojing Liao, XiaoFeng Wang, Haining Wang, Qiang Li, Kai Yang, Hongsong Zhu, Limin Sun
>
> 单位：Beijing Key Laboratory of IoT Information Security Technology, Indiana University, University of Delaware, Beijing Jiaotong University, University of Chinese Academy of Sciences
>
> 会议: USENIX Security 2019
>
> 链接: https://www.eecis.udel.edu/~hnw/paper/security19.pdf
>

## 0 Abstract

作者发现目前大多数针对IoT的攻击都是利用已知的安全缺陷，因此与之前的IoT漏洞分析不同，本文提供了一种对IoT设备漏洞检测的新思路，即从公开的漏洞报告中提取信息来制定相应的检测手段和防御手段，作者开发了一个自动化工具IoTShield来检测设备接收流量中的潜在漏洞操作。

<!-- more -->

## 1 Introduction

通常来说，在发现一个IoT设备漏洞之后，发现漏洞的人应当首先与设备供应商联系，在供应商创建补丁之后，再公开漏洞信息。
然而，作者发现很多IoT漏洞报告公开时供应商没有提供任何安全更新，并且大约80%的漏洞报告都是与利用方法一起发布的，这些报告可能通过博客、论坛或邮件等方式公开，任何人都能够获得有关漏洞的信息，其中的利用方法很容易被黑客利用。

因此，针对这些利用漏洞报告中披露的安全漏洞来远程攻击联网IoT设备的攻击者，作者提出了从漏洞报告中提取信息进行检测防御的思路。文章研究内容大致分为三个部分：

- 首先，作者收集了大量真实IoT攻击案例（蜜罐收集、攻击工具、僵尸网络）进行分析，来了解真实世界中的IoT漏洞和利用。
- 然后，作者提出了利用攻击者对已知漏洞/攻击脚本的依赖性来对IoT攻击进行间接检测防御的思路，并设计了IoTShield工具，包含了报告收集、漏洞提取以及规则生成阶段。
- 最后，作者设计了一系列实验评估了IoTShield的有效性和性能。

## 2 Understanding Real-World Threats

首先，作者收集了大量真实IoT攻击案例（蜜罐收集、攻击工具、僵尸网络）进行分析，得出了“**目前几乎所有的IoT攻击都是利用已知漏洞和公开攻击脚本实现的**”的结论，包含以下三种方法：

### 2.1 Honeypot

本文中的蜜罐包含真实设备和模拟设备，用于收集真实攻击事件。

- 真实设备蜜罐：作者部署了8台存在问题的IoT设备（3台路由器和8台摄像机），存在的问题如表1所示。

![](/images/2020-12-22/tab1.PNG)

- 虚拟设备蜜罐：作者部署了4个模拟的IoT设备蜜罐，如表2所示。

![](/images/2020-12-22/tab2.PNG)

在两个月的时间内，这些蜜罐收集了来自175国家47,089个IP的190,380个HTTP请求。作者通过收集的漏洞利用代码以及常见的攻击模板，识别了来自60个国家大概2000个IoT利用尝试，详细结果如表3所示。分析结果显示至少有90%的恶意攻击来自对已知漏洞的利用，有96%的攻击使用了与收集到的漏洞报告中包含的相同/相似的攻击脚本。

![](/images/2020-12-22/tab3.PNG)

### 2.2 Underground attack tools

作者从地下市场收集了4种攻击工具及其源代码。通过分析源码，作者发现这些工具利用的都是已知漏洞，它们的攻击脚本都是从漏洞报告中复制的，或者仅做了微小的改变。

![](/images/2020-12-22/tab4.PNG)

### 2.3 IoT Botnets
作者还分析了最近一些流行的IoT botnets，结果表明这些攻击里面利用的所有漏洞也被包含在作者收集到的漏洞报告中，攻击脚本也是报告中代码的复制或变种。

![](/images/2020-12-22/tab5.PNG)

## 3 IoTShield

因此，由于IoT攻击脚本通常是在漏洞报告中公开的，攻击者对已知漏洞或公开脚本的依赖也可以帮助我们进行防御。作者提出了**利用攻击者对已知漏洞/攻击脚本的依赖性来对IoT攻击进行间接检测防御**的思路，并设计了IoTShield工具，该工具主要分为三部分：数据收集，IoT漏洞提取，自动签名生成。

![](/images/2020-12-22/fig2.PNG)

### 3.1 Data Collection

作者从13个网站上爬取报告，如表6所示，这些网站是从CVE的参考文献中获取，经过手动筛选得到的。

![](/images/2020-12-22/tab6.PNG)

### 3.2 IoT Vulnerability Extraction

漏洞提取是指从爬取的漏洞报告中提取出漏洞描述信息，主要包含以下三部分：

- Preprocessing
作者首先删除了与漏洞无关的文本（如图片、广告），将网页的主要内容、URL、标题等保留。由于不同网站拥有不同的模板和HTML结构，作者手动分析了上述13个网站来识别出有用的部分。

- Corpora quality analyzer
接下来是筛选去与IoT漏洞报告不相关的文档。
    - 字典单词百分比。作者删除了内容中包含大量字典单词的文档（大概占82%），这是由于作者认为大部分漏洞报告中主要包含的非文本信息，例如漏洞路径、函数、脚本或PoC等。
    - 超链接的数量。作者认为物联网漏洞报告不应该包含过多超链接，因此作者删除了超过25个超链接的文档。
      合理性。作者随机采样了100个被丢弃的文档，手动检查是否存在错误的丢弃，发现这些被丢弃的文档均与IoT漏洞报告无关，证实了这些阈值的设置是合理的。

- IoT vulnerability recognition
最后是从IoT漏洞报告中提取关键信息，作者运用正则表达式从漏洞报告中提取出设备类型、制造商、产品名称以及漏洞类型4个实体以进行后续分析。

![](/images/2020-12-22/tab7.PNG)

![](/images/2020-12-22/fig3.PNG)


### 3.3 Automatic Defense Rule Generation

作者对描述相同IoT漏洞的报告进行聚类，然后利用NLP技术从报告中进一步提取漏洞的语义（漏洞位置或利用路径）以及其他的结构信息（攻击脚本），如图4所示。

![](/images/2020-12-22/fig4.PNG)

- Report clustering：一些报告可能从不同的方面描述了一个相同的漏洞，作者通过使用之前生成实体来对描述相同IoT漏洞的报告进行聚合。

- Semantic and structured information retrieval
作者利用NLP技术分析漏洞报告来发现漏洞的语义，包括漏洞类型、位置和利用语句，主要原理是作者观察到重要的（感兴趣的）语句都具有一个稳定的句子结构。
    - 作者使用正则表达式来找到漏洞位置、漏洞参数，之后使用SDP(Stanford Dependency Parser)来解析这些语句。
    - 作者使用正则表达式来识别不同的结构化信息（攻击脚本），并将其转化为一个通用的模板：```http://HOST:PORT/file.suffix?{parameters}```
    - 其中HOST为设备IP地址，PORT为应用层服务端口（默认为80），file.suffix为漏洞位置，{paramenters}为漏洞利用参数。

![](/images/2020-12-22/fig5.PNG)

- Signature generation
    - 首先，作者比较漏洞位置信息和结构化信息模板中的"file.suffix"是否匹配，如果匹配，则认为漏洞的语义信息是与模板建模的规则相关。
    - 其次，作者基于漏洞类型决定是否需要忽略利用参数部分，这是由于某些漏洞不需要利用参数（信息泄露和目录遍历）。对于需要参数的漏洞，IoTShield会从描述中识别出利用参数来覆盖{ paramenters }字段。
    - 对于一些没有漏洞描述或结构化信息的报告，作者通过其结构化信息或者漏洞描述来生成利用签名。


## 4 Evaluation

作者实现了IoTShield，并对其进行了有效性评估和性能评估。

### 4.1 Effectiveness

如表6所示，IoTSheild从43万篇文章中收集了7514个IoT漏洞报告，这些报告披露了12286个IoT漏洞。作者从识别的报告中随机抽取200份进行了人工验证，准确率达到了94%。
作者发现这些IoT漏洞与设备类型和供应商相关，近60%的漏洞设备来自具有最多安全漏洞的前10供应商，如表8所示。

![](/images/2020-12-22/tab8.PNG)

表9列出了数据集中的十大漏洞类型，其中大多是可远程利用的。XSS、命令注入、命令执行常被恶意软件用于在一个僵尸网络节点设备上执行命令。

![](/images/2020-12-22/tab9.PNG)

作者使用从IoT设备和蜜罐收集到的190K个HTTP请求来评估IoTShield规则生成的有效性，评估结果如表10所示。

![](/images/2020-12-22/tab10.PNG)


此外，作者还在工业控制系统的HMI蜜罐中捕获的流量来评估IoTShield，IoTShield报告了7396条警报，作者手动检查确认了大约6705条警报确实为物联网攻击，其他警报为普通Web服务器上的漏洞攻击。


### 4.2 Performance
作者测试了IoTShield在各个阶段处理漏洞报告的时间，运行环境为商用台式计算机（Ubuntu 18.04, 8GB of memory, 64-bit OS, 4-core Intel(R) Core(TM) i7-4790 CPU @ 3.60GHz），如表11所示。结果表明IoTShield是高效的，几乎不会带来额外的开销。

![](/images/2020-12-22/tab11.PNG)

## 5 Discussion

### 5.1 Limitation
作者讨论了IoTShield一些不足之处：

- 本文的数据收集检索了13个不同的网站，并不能涵盖所有的IoT漏洞。
- 由于蜜罐IoT设备类型和部署时间不同，所收集的流量日志可能存在偏差。
- 由于IoTShield处理的漏洞信息可能是不完整的（缺少漏洞描述或结果信息等），可能漏报一些漏洞的变种。
- IoTShield无法处理使用特定程序语言编写的漏洞利用，并且其处理范围仅限于流量日志、脚本、Linux命令等。

### 5.2 Mitigation
作者根据研究结果提出了几种有效的缓解策略：
- 供应商应该提供一个官方漏洞报告平台，及时检查并回复漏洞报告。
- 供应商需要为停产的产品提供技术支持，尤其是仍广泛使用的设备。
- 供应商应当尽量正确配置设备，并及时通知用户更新设备信息或自动更新。
- 漏洞的发现者应当遵循漏洞报告的准则，尽量在供应商发布补丁程序之后，公开漏洞信息。
- 设备用户需要注意设备配置，及时更新设备。
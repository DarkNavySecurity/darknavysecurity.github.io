+++
title = '2024年度最“安全”的防线'
date = 2025-02-15T17:54:34+08:00
draft = false
images = ["attachments/5232e288-9057-4205-9c0d-c8ca0dead169.webp"]
+++

 ![](attachments/a5186d85-b7a4-4ede-9f70-d2ceb5be30ff.webp)

在攻防对抗日益激烈的2024年，安全软件一直被视为企业安全防线的重要基石。然而，这些安全软件本身也可能存在漏洞，甚至被攻击者利用作为入侵的跳板来对用户造成危害。多年来，因为安全软件而导致的事故不禁让人产生一个疑问——安全软件真的可信吗？

以下为本期《深蓝洞察 | 2024 年度安全报告》的第八篇。

 ![图片](attachments/5232e288-9057-4205-9c0d-c8ca0dead169.webp)

## 安全软件被滥用为攻击工具

在当前APT（高级持续性威胁）组织频繁活动的环境下，EDR/XDR已成为企业安全防御体系的核心组件，负责监控数百万端点和服务器。然而，权力越大，责任也越大。这些承担重任的安全软件一旦存在安全漏洞，便可能成为攻击者手中的利器，用于部署勒索软件、窃取敏感信息，并且难以被察觉或移除。

近日，SCRT的安全研究人员通过逆向工程与动态分析，发现Palo Alto Networks Cortex XDR产品中存在一个权限提升漏洞（**CVE-2024-5907**）。用户用受Cortex XDR保护的设备请求Cortex服务（cyserver.exe）生成日志，管理员可以使用这些日志来排除代理的潜在问题。

 <img src="attachments/32b7c186-da40-4371-afe3-335b9bc49c5f.webp" style="display: block; margin-left: auto; margin-right: auto; zoom: 100%;"/>

**CVE-2024-5907**漏洞的核心在于利用Cortex XDR的日志生成机制实现权限提升。攻击者通过非特权用户身份请求生成日志，触发以SYSTEM权限运行的cyserver.exe服务。该服务在处理日志生成时，会在`C:\Windows\Temp`目录下创建临时文件夹，其名称遵循可预测的`tmp<进程PID><递增ID>`模式，其中ID基于字典序递增，并可从历史日志文件推断，使得攻击者能够预测未来临时目录的路径。

 ![创建临时文件夹](attachments/0003bbc5-7f76-4488-8f09-16278246a1f0.webp)

由于这些临时文件夹继承了`C:\Windows\Temp`的宽松访问控制列表，所有用户均拥有写入权限，攻击者便可预先在预测路径下创建恶意的NTFS Junction软链接，将该目录重定向至**高权限敏感位置**，例如`C:\Config.msi`。具体攻击过程如下：

* 首先，当cyserver.exe服务执行临时文件清理操作时，会对临时目录进行递归删除，但其在处理Junction链接时，并未进行充分的路径合法性校验，直接以SYSTEM权限对Junction链接指向的目标目录执行删除操作，从而错误地删除了系统关键目录`C:\Config.msi`下的文件。
* 其次，攻击者进一步利用Windows Installer服务在处理`C:\Config.msi`文件夹时存在的竞争条件漏洞，通过反复触发日志生成与目录删除操作，干扰系统对Config.msi资源的锁定机制。
* 最终，在服务尝试重建该目录时，劫持文件写入流程，从而以SYSTEM权限执行操作，达成本地权限提升的目的。

此漏洞的成功利用高度依赖于多个竞争条件，如临时目录名称预测的准确性、软链接植入时机与文件删除操作时序的精确同步，虽然漏洞利用过程可能需要多次尝试，但这个漏洞依然危害严重。这意味着，在用户权限受限的情况下，攻击者也能如入无人之境，绕过XDR的层层保护，直接威胁到企业内网的敏感数据和关键基础设施，使安全软件的防护形同虚设。

## 一次全球性的防护软件灾难

2024年7月19日，美国知名网络安全公司CrowdStrike因分发异常更新，导致全球数百万台计算机崩溃，严重影响航空、银行和媒体等行业的运作，预计在全球范围内造成数十亿美元的损失。

CrowdStrike提供一系列安全防护软件帮助客户保护计算机抵御网络攻击，旗下的Falcon Sensor（猎鹰传感器）产品在个人电脑Windows操作系统的内核层面安装传感器，用来检测和预防网络攻击，而这种高级别的防护恰恰成为造成本次事件的重要原因。当安全软件拥有**内核权限**之后它的防护能力大大提升，但同时风险也随之提高——一旦出现问题，对用户造成的影响异常严重。

尽管CrowdStrike官方宣称此事件并非安全漏洞，而是驱动程序错误，但这次事件无疑给用户敲响了警钟：**纵然是顶尖的安全软件，也可能因其自身的缺陷引发难以预料的灾难。**

 <img src="attachments/be660b01-7543-496e-b5b4-982c0f3858cf.webp" style="display: block; margin-left: auto; margin-right: auto; zoom: 60%;"/>

## 防线上的"后门"

2024年Fortinet公司旗下的FortiManager（用于集中管理FortiGate设备的工具）产品中又出现了严重的CVSS 9.8漏洞（"FortiJump" **CVE-2024-47575**）该漏洞是 FortiManager和FortiManager Cloud中的FortiGate到FortiManager (FGFM) 守护进程 (fgfmsd) 中缺少的身份验证漏洞。

利用FortiJump漏洞，未经身份验证的远程攻击者可以使用有效的FortiGate证书在 FortiManager中注册未授权的设备，攻击者可通过特制请求执行任意代码或命令。

Fortinet官方公告中也承认："FortiManager fgfmd守护进程存在一个严重的功能认证缺失漏洞，可能允许远程未认证的攻击者通过特制请求**执行任意代码或命令**。"根据 Mandiant的一份最新报告显示，FortiJump漏洞自2024年6月以来，已在0Day攻击中被广泛利用，涉及不同行业的 50 多台可能被入侵的FortiManager设备。

这并不是Fortinet公司的产品第一次出现漏洞，在此之前Fortinet FortiOS目录遍历漏洞(**CVE-2018-13379**)、Fortinet防火墙身份认证绕过漏洞（**CVE-2022-40684**）、Fortinet FortiGate SSL VPN存在格式化字符串漏洞（**CVE-2024-23113**）等漏洞。**这次的FortiJump漏洞再次让人思考，安全软件真的安全吗？**

**安全软件的脆弱性**

安全软件的漏洞屡见不鲜，这不仅是技术层面的偶发问题，更是一种持久存在的安全挑战。在攻防技术螺旋上升的趋势下，厂商虽致力于对安全产品进行更新与优化，但攻击者的漏洞挖掘也从未停止。以下这些漏洞不仅体现了安全产品在技术复杂性上的脆弱性，也反映出攻防对抗的**长期性和动态性**的现实：

* 2015年，有史以来唯一获得100次VB100的产品ESET NOD32中的存档支持模块中的基于堆的缓冲区溢出允许远程攻击者通过大量语言在类型为SIS_FILE_MULTILANG的EPOC安装文件中执行任意代码。（**CVE-2015-8841**）
* 2017年，俄罗斯的一家拥有25年以上安全行业经验的公司——Kaspersky，其用于保护嵌入式系统安全的产品Kaspersky Embedded Systems Security(KESS)被发现了一个内存破坏类型的缺陷，攻击者可以利用其中一个驱动程序实现本地权限提升。（**CVE-2017-12823**）
* 2021年，McAfee Agent for Windows的maconfig中存在权限管理不当漏洞，允许本地用户访问敏感信息。该程序可由低权限用户在文件系统上的任何位置运行。（**CVE-2021-31836**）
* 2025年初，Ivanti公司的Connect Secure产品存在堆栈缓冲区溢出问题，允许远程未经身份验证的攻击者实现远程代码执行。（**CVE-2025-0282**）

 <img src="attachments/e909e938-a3b3-4869-80bf-6bfb97b99124.webp" style="display: block; margin-left: auto; margin-right: auto; zoom: 60%;"/>

安全防护软件为实现更好的防护效果深入操作系统底层，拥有系统权限。然而，这种高权限的安全软件一旦被攻破，攻击者即可利用其权限执行任意操作。

这些安全事件并非孤立个案，而是引出了一个深层次的问题：**为何作为防御核心的安全软件，反而频频成为攻击者的突破口？**

随着技术的不断发展，安全防护逐渐从单点防御转向多层次、多维度的综合体系，但这也必然导致安全软件复杂度的增加。安全软件作为防御体系的重要一环，其本身的安全性直接关系着整个系统的安全性，未来安全软件的发展需要在功能性与安全性之间实现更好的平衡。


---

## 深蓝洞察

作为安全防护的工具，这些安全软件原本是为抵御威胁而设计，但在复杂多变的网络环境中，它们却可能反过来成为攻击者的突破口。从历史漏洞到新挖掘的漏洞利用，无不反映出攻防对抗螺旋上升的动态现实。

这一现象不仅揭示了安全软件在特定情况下的脆弱性，也表明这绝非一时之患，而是一个贯穿整个行业的持久挑战，提醒我们必须以更全面、更深入的视角重新审视网络安全的防御策略。


---

## 参考

* \[1\] [https://blog.scrt.ch/2024/12/05/attacking-cortex-xdr-from-an-unprivileged-user-perspective/]()
* \[2\] [https://zh.wikipedia.org/wiki/2024年CrowdStrike大规模蓝屏事件]()
* \[3\] [https://www.anquanke.com/post/id/301285]()
* \[4\] [https://security.paloaltonetworks.com/CVE-2024-5907]()
* \[5\] [https://www.fortiguard.com/psirt/FG-IR-24-423]()
* \[6\] [https://www.tenable.com/blog/cve-2024-47575-faq-about-fortijump-zero-day-in-fortimanager-fortimanager-cloud]()
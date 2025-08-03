互联网被描述为一个复杂、动态、大规模分布式、可扩展且互联的系统，其设计者无法预见它会演变成人类最伟大的创造之一。互联网流量通过使用域间协议的连接进行交换。连接互联网的关键但不安全的域间协议称为边界网关协议（BGP），它通过在自治系统（AS）网络之间路由流量来连接超过 8 万个自治系统（此后称为 AS）。考虑到私有和保留的 AS 时，使用的 AS 数量可能会更多。网络协议创新与互联网的历史紧密相连，尽管网络协议安全并非互联网设计者的主要关注点。例如，传输控制协议和互联网协议（TCP/IP）至今仍是网络的核心，它是由美国国防高级研究计划局（DARPA）的工作发展而来的。恶意和非恶意的 BGP 事件都影响了主要的互联网实体，包括 Akamai、亚马逊、苹果、Facebook、谷歌、万事达卡和微软。BGP 异常的影响范围从相对无害的路由抖动到破坏性的路由泄露和劫持。BGP 事件分为直接（有意或无意）、间接或中断。BGP 劫持和路由泄露事件是直接事件的典型例子（如马来西亚电信事件），而间接事件包括扰乱关键互联网操作并间接影响 BGP 的网络事件（如 Wannacrypt 等互联网蠕虫）。中断则源于自然灾害和 / 或能源系统故障（如莫斯科停电事件）。

尽管存在来自 AS 的多元数据相关性，但当前的 BGP 异常检测技术存在局限性，其分析大多来自单个监测点。然而，互联网是一个复杂的现象，有许多监测点和交互，而 BGP 本身是一个非线性动态系统，需要高维异常检测。充其量，大多数技术试图从单个可观测数据推断动态和信息。尽管先前对多元数据和许多特征进行了成功分析，但这些技术在应对更高级的攻击时存在局限性。例如，一种高级 BGP 攻击旨在避开公共收集器基础设施，针对路由监测盲点，以利用单可观测方法。现有技术提供的检测可见性不足，高级 BGP 检测技术需要在多 AS 互联网络组级别具备能力。这要求对 BGP 发言人进行充分建模，以研究 AS 组在交互方面的相似性和差异性，以及大群 AS 的动态。

许多技术已用于分析来自单个 AS 或单个监测点的 BGP 流量，并从收集器进行推断。然而，为了识别能够捕获和量化多个分布式 AS 组的动态以进行多视角（MVP）异常检测的候选技术，我们将 BGP 建模为一个多维、多可观测系统。捕获 AS 的群体级信息和动态至关重要，包括研究使用多个监测和有利位置检测 BGP 异常的能力，以及通过识别收集器内的对等体如何相互作用、发挥的群体动态以及收集器中的对等体可能存在的差异，来确定在 BGP 收集中使用多个对等体的有效性。换句话说，就是能够捕获、量化和使用群体级 AS 信息进行群体级 AS 异常检测的能力。鉴于 BGP 已被成功证明具有非线性动态系统的特征，我们认为捕获和研究 AS 组之间的交互和群体动态，无论是公共还是私有收集和监测基础设施，都有助于实现高级群体级异常检测技术。

虽然之前已经对时间序列异常检测和 BGP 异常检测进行了综述，但尚未有对能够进行 MVP BGP 异常检测或检测高级 BGP 攻击的技术的综述。先前下一代 BGP 异常检测的标准不足以检测一些高级 BGP 攻击，如那些规避公共收集器监测的攻击。本综述通过识别能够增强 BGP 基础设施抵御高级攻击能力的技术，直接解决了 BGP 中的安全弱点。本综述对不同技术进行了评估，作为其 MVP 能力和下一代异常检测要求的证据。我们的贡献如下：

确定了 AS 组级多视角方法检测高级 BGP 攻击的需求和条件。
为了研究人员的利益，对 178 种独特的异常检测技术进行了系统综述。
确定了针对路由收集器可见性限制的高级 BGP 攻击检测的可能 MVP 检测候选技术。
对一些已确定但从未应用于 BGP 异常检测的候选技术进行了早期探索性分析，并报告了初步结果。

本文的其余部分组织如下。在第二部分中，我们总结了 BGP 的功能。第三部分描述了不同类型的 BGP 攻击。第四和第五部分阐述了计算高效的 AS 组 MVP 异常检测的需求。第六部分根据复杂性和群体级 MVP 异常检测要求评估了所有已知的攻击类别。第七部分是对 178 种独特异常检测技术的综述，旨在识别高级 MVP BGP 异常检测候选技术。请注意，某些技术可能会在多个类别中被识别和归类。在第八部分中，我们简要描述了可能的候选方法，并对其中两种方法进行了一些初步的详细评估。第九部分讨论了一些未来的研究机会，第十部分对本文进行了总结。

作为互联网的默认域间路由协议，BGP 被描述为一种路径向量和距离向量变体协议。尽管互联的路由域（AS）使用共享协议（BGP）进行通信，但它们是由单个机构管理的自治实体。AS 通常是大型、复杂的网络群组。互联的 AS 也不仅仅受物理或地理限制，而是由公司、组织和政治因素形成的；基于拓扑中心的互联网操作推断可能存在缺陷，有关互联网交互和动态的重要信息可能在抽象过程中丢失。

每个 BGP 消息都有一个一致的头部结构，包括标记、长度和类型字段，总大小为 19 字节。标记字段长 16 字节，全为 1，用于标识消息的开始。长度字段占 2 字节，指定消息的整体大小（包括头部）。类型字段标识消息的类别，可以是五种类型之一：OPEN、UPDATE、NOTIFICATION、KEEPALIVE 和 ROUTE REFRESH，具体在各种 RFC（如 1771、4271、2918 和 RFC 7313）中指定。TCP 会话的启动会触发发送 OPEN 消息，标志着达到 ESTABLISHED 状态所需的 BGP 消息交换的开始。会话的终止和维护分别通过 NOTIFICATION 和 KEEPALIVE 消息进行通信。

BGP 对等体中的路由存储通过三个关键数据库进行管理，统称为路由信息库（RIB）：Adj - RIB - In、Adj - RIB - Out 和 Loc - RIB。Adj - RIB - In 负责保存从其他对等体的 UPDATE 消息中获取的路由，本质上反映了从相邻对等体学到的有助于路径选择过程的路由。相反，Adj - RIB - Out 存储传播给其他对等体的路由，而 Loc - RIB 维护对等体当前选择的最佳路由，该路由由 Adj - RIB - In 数据和对等体的内部路径选择标准确定。路由信息的修改（如公告、撤回或现有属性的更新）在 RIB 数据交换后进行通信。
在本节中，我们将全面概述 BGP 异常和攻击，根据其意图和影响进行分类。本节还将讨论高级 BGP 攻击，这些攻击通常通过针对公共 BGP 收集器的盲点来规避检测。讨论强调了下一代异常检测技术的关键需求，这些技术能够通过分析群体级 AS 交互来应对这些高级威胁。
先前研究表明，高达 72% 的域容易受到最基本的 BGP 子前缀劫持攻击，高达 70% 的域容易受到相同前缀攻击。研究还表明，从排名前五的证书颁发机构（CA）获取虚假证书非常容易，所有这些机构都容易受到标准 BGP 劫持攻击。在这些经过验证的攻击之后，一些 CA 开始实施缓解措施，尽管高度针对性的精确 BGP 攻击仍然是一种威胁。

传统的前缀劫持涉及未经授权地公告 IP 前缀，有效地将原本发往合法前缀所有者的流量吸引到恶意 AS。传统前缀劫持的简单性和有效性在许多情况下得到了证明，造成了广泛的破坏，并且在缓解方面存在重大挑战。前缀劫持中，攻击者 AS（ASA）故意公告它不拥有的 IP 前缀，从而误导流量通过未经授权的路径。
子前缀劫持是 BGP 路由攻击的一种微妙形式，因其能够通过未经授权的路径秘密转移互联网流量而备受关注，从而实现一系列恶意活动，包括流量拦截、监视和潜在的数据操纵。在传统的子前缀劫持中，恶意 AS 故意公告比合法所有者更具体的 IP 前缀，有效地将发往合法前缀的流量引诱到攻击者处。由于 BGP 在路由决策中倾向于更具体的前缀，互联网上的路由器会被欺骗，将流量重定向到恶意 AS，而合法前缀所有者和不知情的用户往往不知道其数据已受到攻击者的控制。传统子前缀劫持的影响可能是毁灭性的，使攻击者能够窃听数据、进行中间人攻击，甚至将流量黑洞化，从而破坏通信并可能造成重大的运营和声誉损失。
在 BGP 生态系统中，拦截攻击（特别是作为中间人操作执行的拦截攻击）代表了前缀劫持技术的复杂演变。与仅仅重定向流量的传统劫持不同，拦截攻击的特点是能够转移并随后转发流量，确保通信到达预期目的地，尽管是通过攻击者的网络。这种双重行动使攻击者能够保持不被发现，同时保留网络的连接性和功能，同时秘密监视或操纵数据。
在 BGP 安全领域，重放和抑制攻击带来了微妙的挑战，将域间路由的稳定性和可靠性与寻求利用 BGP 固有信任和缺乏身份验证的对手的恶意意图交织在一起。这些攻击的机制基于对 BGP 撤回消息的操纵，而 BGP 撤回消息对于维护网络内路由路径的完整性和最优性至关重要。
与可能源自单个 AS 的其他攻击不同，合谋攻击涉及两个或多个非相邻 AS 的协同恶意行为。这些 AS 在它们之间创建虚拟隧道，建立 BGP 会话，通过该会话它们可以交换和传播伪造的路由信息，而不会引起明显的路由冲突。
MED 修改攻击侧重于操纵 BGP 属性中的多出口鉴别器（MED），AS 使用该属性向其邻居传达传入流量的更优路径。MED 是影响路由选择的关键属性，特别是在两个 AS 之间存在多条路径的情况下。通过恶意修改 BGP 公告中的 MED 值，攻击者可以影响路径选择过程，使流量通过特定的、可能是恶意的路径进行路由。这可以被利用来促进流量拦截、创建网络拥塞，或仅仅通过迫使流量通过次优路径来降低网络性能。MED 修改攻击的微妙之处在于它们能够在不违反 BGP 路径选择规则或引起明显路由冲突的情况下操纵路由决策。
利用 RFD 和 MRAI 定时器引入了另一层协议操纵，攻击者试图利用 BGP 机制来缓解路由抖动并控制 BGP 公告的频率。路由抖动抑制（RFD）旨在通过抑制频繁抖动的路由来稳定 BGP，而最小路由公告间隔（MRAI）定时器控制连续 BGP 更新之间的最小时间间隔，旨在减少 BGP 路由器的负载和网络中的 BGP 更新消息数量。通过策略性地操纵 BGP 公告以利用这些机制，攻击者可能会导致合法路由被抑制，创建路由不稳定，或操纵 BGP 更新在互联网上的传播。例如，通过故意使路由抖动，攻击者可以触发 RFD 并导致路由被抑制，从而影响路由决策并可能启用其他类型的攻击。
在 BGP 上下文中，拒绝服务（DoS）攻击涉及恶意操纵路由表和路径，使网络或其部分无法访问。BGP DoS 攻击可能特别有害，因为它们可以破坏跨多个网络的数据流量，而不仅仅是直接目标。在 BGP DoS 攻击中，攻击者可能操纵 BGP 公告以丢弃发往特定前缀的流量或创建路由循环。这可能涉及公告不属于攻击者的 IP 前缀，导致流量被错误路由，或故意撤回合法的 BGP 公告，导致网络中断。
越来越智能的攻击被设计为避开公共 BGP 收集器和监测基础设施；这些攻击被有效地部署为 “监测感知” 攻击。BGP 策略或社区也可以被操纵以协助高级攻击（例如，MED 修改、本地偏好操纵或 BGP 社区标记和工程）。这些攻击利用对全球 BGP 政策格局的了解以及路径选择过程中的特定漏洞，以确保恶意路径在通过 AS 时不会触发任何警报，也不会比公共收集器基础设施监测的路径更受青睐。
本节强调了在支持 BGP 的路由器中低参数和计算高效的异常检测技术的必要性。由于 BGP 路由器的计算资源（如 CPU 功率、内存和能量）有限，这些设备必须优先处理路由更新和 BGP 消息。复杂、参数众多的异常检测算法会引入处理延迟和可扩展性挑战，损害路由器的核心功能和网络性能。轻量级和快速的异常检测技术对于确保及时识别威胁和保持弹性至关重要，同时不会给网络基础设施带来过重负担。本节强调了减少检测技术中使用的参数数量对于在大规模 BGP 网络中保持速度和可扩展性的关键作用。
支持 BGP 的路由器受到有限计算资源的限制，包括 CPU 功率、内存和能量。这些资源主要用于处理路由表更新和 BGP 消息。因此，任何额外的功能（如异常检测方案）必须以资源高效的方式实现，以避免损害路由器的主要功能。BGP 网络的高速和大规模性质要求快速、轻量级和计算高效的异常检测技术，以确保及时识别威胁和网络弹性。
镜像拉取：
~~~
docker pull mcr.microsoft.com/mssql/server:2022-latest
~~~
运行：

~~~
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=Gaoyifeng1." \
   -p 1433:1433 --name sqlserver01 --hostname sqlserver01 \
   -d \
   -v /docker/sqlserver:/var/opt/mssql \
   mcr.microsoft.com/mssql/server:2022-latest
~~~

~~~
docker exec -it sqlserver01 /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'Gaoyifeng1.'
~~~


USE YGGL202307080256

CREATE TABLE [dbo].Employees202307080256
(
    EmployeeID    char(6)     NOT NULL  PRIMARY KEY,
    Name          char(10)    NOT NULL,
    Education     char(4)     NOT NULL,
    Birthday      date        NOT NULL,
    Sex           bit         NOT NULL  DEFAULT 1,
    Workyear      INT         NULL,
    Address       varchar(40) NULL,
    PhoneNumber   char(12)    NULL,
    DepartmentID  char(3)     NOT NULL
);


CREATE TABLE [dbo].[Salary202307080256] (
  [EmployeeID] char(6) COLLATE SQL_Latin1_General_CP1_CI_AS  NOT NULL,
  [InCome] float(53)  NOT NULL,
  [OutCome] float(53)  NOT NULL,
  CONSTRAINT [PK__Salary20__7AD04FF1857FB37A] PRIMARY KEY CLUSTERED ([EmployeeID])
WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON)  
ON [PRIMARY]
)  
ON [PRIMARY]
GO

ALTER TABLE [dbo].[Salary202307080256] SET (LOCK_ESCALATION = TABLE)
GO

EXEC sp_addextendedproperty
'MS_Description', N'员工编号，主键',
'SCHEMA', N'dbo',
'TABLE', N'Salary202307080256',
'COLUMN', N'EmployeeID'
GO

EXEC sp_addextendedproperty
'MS_Description', N'收入',
'SCHEMA', N'dbo',
'TABLE', N'Salary202307080256',
'COLUMN', N'InCome'
GO

EXEC sp_addextendedproperty
'MS_Description', N'支出',
'SCHEMA', N'dbo',
'TABLE', N'Salary202307080256',
'COLUMN', N'OutCome'




CREATE TABLE [dbo].[Departments202307080256] (
  [DepartmentID] char(3) COLLATE SQL_Latin1_General_CP1_CI_AS  NOT NULL,
  [DepartmentName] char(20) COLLATE SQL_Latin1_General_CP1_CI_AS  NOT NULL,
  [Note] text COLLATE SQL_Latin1_General_CP1_CI_AS  NULL,
  CONSTRAINT [PK__Departme__B2079BCD2F52A57C] PRIMARY KEY CLUSTERED ([DepartmentID])
WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON)  
ON [PRIMARY]
)  
ON [PRIMARY]
TEXTIMAGE_ON [PRIMARY]
GO

ALTER TABLE [dbo].[Departments202307080256] SET (LOCK_ESCALATION = TABLE)
GO

EXEC sp_addextendedproperty
'MS_Description', N'部门编号，主键',
'SCHEMA', N'dbo',
'TABLE', N'Departments202307080256',
'COLUMN', N'DepartmentID'
GO

EXEC sp_addextendedproperty
'MS_Description', N'部门名',
'SCHEMA', N'dbo',
'TABLE', N'Departments202307080256',
'COLUMN', N'DepartmentName'
GO

EXEC sp_addextendedproperty
'MS_Description', N'备注',
'SCHEMA', N'dbo',
'TABLE', N'Departments202307080256',
'COLUMN', N'Note'
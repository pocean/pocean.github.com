---
author: pandao
comments: false
date: 2013-09-14 07:43:34+00:00
layout: post
published: false
slug: log-print-standard-of-software-platform
title: 软件平台日志打印规范
thread: 254
categories:
- 杂七杂八
tags:
- 规范
---



软件平台日志打印规范V1.0.0




 
目  录Table of Contents 
1	概述	5
1.1	背景	5
1.2	说明	5
1.3	读者对象	5
2	告警、事件、日志差别：	5
3	日志输出说明：	6
3.1	日志级别定义（适用于主机、OMU）：	6
3.2	日志类型定义（目前只有主机有此类型定义，OMU没有）：	7
4	日志打印控制原则：	7
4.1	ERR日志打印原则：	7
4.2	NOTICE日志打印原则：	8
4.3	日志内容精简充分原则：	8
4.4	日志输出格式要求：	8
4.5	日志避免冗余打印建议：	9
4.5.1	大错误引发小错误，酌情只打印大错误日志，不打印小错误日志：	9
4.5.2	循环体内打印，考虑把日志挪到循环外层，避免过多日志打印：	9
4.5.3	函数接口调用打印：	9
4.5.4	定时任务打印，避免每次出错都打印：	10
4.5.5	数据库、资源异常，控制过多打印	10
4.5.6	无法确认是否是系统错误时，不打印ERR日志，而修改为WARN或INFO日志，并考虑增加其他定位手段方法：	10


术语和定义Term&Definition;：
缩略语Abbreviations 	英文全名 Full spelling	中文解释 Chinese explanation
		
		

 
1	概述
1.1	背景
日志是定位问题的重要手段之一，可以提供一些系统运行异常时的内部信息，以便研发根据这些信息定位问题。

目前日志输出存在较多问题，主要有：
1、打印的日志信息无法支撑问题定位，信息不全；
2、打印信息泛滥，不必要的信息太多，淹没了有效信息，获取日志信息困难；
3、打印缺乏规范约束，打印出来的信息分析困难。

为了使日志能够有效支撑问题定位，特制定日志输出规范。

1.2	说明
日志定位能力要求：
我们的日志要求支撑80%以上的网上问题定位，出现问题时，只通过日志就能定位出80%的问题。为此，日志打印必须提供足够的定位信息，对出现问题的各种异常场景都能够记录下来。同时，由于现网日志的获取困难，又要求我们必须把日志控制在一定的量范围之内。用尽量少的日志打印，支撑尽量多的问题定位。这就是我们的日志要实现的目标。

日志打印关键是开发人员思想上的重视，只有从思想上重视了以下两点，才能真正实现日志打印的目标。
1、明确日志资源的宝贵（日志文件大小有限）；
2、提供的定位信息足够定位问题。

1.3	读者对象
本规范的主要读者包括：
1、 代码架构师 — 控制部门代码的日志打印，指导部门日志打印能力的提高，评
价各模块的定位能力；
2、 日志SE     — 设计、改进日志模块；
3、 开发人员   — 编码时，编写出合格的日志；
4、 测试人员   — 提问题单时，根据日志给出每个问题对应模块的定位能力。

2	告警、事件、日志差别：
告警/事件/日志 的差别如下：
【告警】：系统检测到故障或者可能产生故障（有隐患）时，需要用户进行干预或处理的，输出告警。举例：检测到故障，如单板故障；有隐患的，如补丁长时间未确认（当前是激活的，一旦模块复位，就失效了）。

【事件】：系统运行时输出的信息，不需要主动通知用户进行干预或处理，主要用于用户（外部客户/技服）问题分析定位，或该信息未来可能需要通过统计发出告警的情况，可以设计为事件。

【日志】：系统内部运行信息、程序异常运行分支、调试用的信息等用于开发内部定位问题使用的信息作为日志记录，使用者为研发人员。

日志和事件相比，主要差异在使用的对象上，如果只是研发内部定位问题用，用户、GTS看不懂或不需要看的，就做成日志，如果用户也能理解分析或者可以自动分析的，就以事件方式输出。
举例：
1、“硬盘状态改变”用来记录硬盘在状态迁移中的关键状态，这个事件可能意味着硬盘故障，也可能只是正常的状态迁移，当硬盘故障时也有相应的故障告警送出，通过观察这个事件可以了解硬盘的状态迁移过程，协助用户/GTS处理硬盘故障告警，或者频繁的状态迁移可能也是故障的前兆，用户可以通过主动挖掘来发现隐患。或者故障自动定位到FRU需要的信息也需要输出事件。
2、“设置E1T1属性失败”原来是作为故障告警，它在CGP调用CGA的HPI接口失败时送出（不一定是MML命令触发），这种属于程序异常运行分支，理论上可能出现，但一旦出现，用户也没法处理，只能开发人员定位问题，且如果是MML命令触发，命令报文直接会返回失败，所以修改为记录日志，不送事件。

3	日志输出说明：
通过日志级别控制日志输出范围，按照日志类型进行分类保存，日志输出级别和类型严格按照如下定义要求填写。

3.1	日志级别定义（适用于主机、OMU）：
只允许使用这四个级别，其他日志级别进行匹配转换。
INFORMATIONAL  ：提示级别。用于打印系统运行轨迹信息。比如函数入口和出口打印、模块间接口消息等。由于打印量较大，在正式发布版本中此级别默认不输出，只有在需要时通过维护命令打开。
NOTICE         ：重要提示级别。标识系统重要状态变迁，如：链路状态迁移、主备倒换等。在正式发布版本中此级别默认输出。
（只有OMU有此级别，主机不使用，主机通过DW_LOG日志类型来表示此类型日志）
WARNING       ： 警告级别。不会导致处理出错，但是有潜在危险。比如备份失败打印，备份失败是不会影响正常流程的。但是有潜在危险，如果主备倒换，由于数据没备份过去，会导致呼损。在正式发布版本中此级别默认不输出。
ERR           ： 错误级别。会导致系统运行期错误的错误。比如输入指针为空、编解码失败等错误，一旦发生了，就没办法再继续处理。这类错误要通过ERR级别打印出来，在任何情况下，ERR级别的打印都是需要被关注的。在正式发布版本中此级别默认输出。

3.2	日志类型定义（目前只有主机有此类型定义，OMU没有）：
OS_LOG       ：  操作系统日志。如果操作系统出错，比如死机信息，按照此类型保存。此类型日志在OMS的如下目录：/opt/HUAWEI/ims/workshop/omu/share/run_log/omu/devlog/logs/网元版本/网元ID/模块号/runlog/os 。
DEBUG_LOG    ：  应用软件运行日志。各业务模块日志，平台与操作系统无关的日志，按照此类型保存。此类型日志保存在如下目录：/opt/HUAWEI/ims/workshop/omu/share/run_log/omu/devlog/logs/网元版本/网元ID/模块号/runlog/debug。
DW_LOG       ：  告警模块日志。此类型日志保存在如下目录/opt/HUAWEI/ims/workshop/omu/share/run_log/omu/devlog/logs/网元版本/网元ID/模块号/runlog/warning。
用于保存系统重要状态变迁，如：链路状态迁移等。
IS_LOG       ：  内部统计日志。不单独保存。和DEBUG_LOG类型日志保存在相同目录下。用于保存内部统计信息。

4	日志打印控制原则：
说明：下面的原则建议，都是针对ERR、NOTICE级别。WARN、INFO级别由于默认不输出，不在此规范中进行限制，具体打印可以参考此规范。

4.1	ERR日志打印原则：
不是每次出错都必须打印，对定位问题没有帮助的错误，不用打印ERR日志；
不只是打印出问题，更主要的是要打印出发生问题的原因。

1、如下情况必须打印ERR日志：
不打印此条日志，将导致定位问题很不方便，无法通过其他定位手段推断出此处错误原因，无法定位问题。
比如：调用系统函数失败，需要打印出调用系统函数时的输入参数、失败错误码、失败原因描述等。
当状态变化（比如：由好变坏）时打印ERR日志。
由坏变好时，则可以打印NOTICE日志。

2、如下情况禁止打印ERR日志：
不打印此条日志，也可以通过其他错误日志，推断出此处错误原因，并能定位问题。
如果已经有告警/事件或者其他可观测的输出，日志可以考虑不输出或者至少不输出ERR级别。
比如：定时任务检测的状态已经有告警，则不打印ERR日志；其他可观察的输出包括在消息中返回了错误码，通过用户跟踪/信令跟踪已经可以知道具体业务异常的情况。

4.2	NOTICE日志打印原则：
NOTICE日志必须慎重打印，无控制的NOTICE打印将导致日志过多，不利于保持日志文件的定位能力。NOTICE日志打印需要经过部门代码架构师审核、备案。

由于NOTICE日志级别默认打开，使用了NOTICE级别的日志就意味着该条日志量并没有减少。

如下情况可以打印NOTICE日志：
a、重要流程：
对于一些流程打印，检查流程是否经过某点，可以使用INFO级别；
除非流程非常重要，而且在正常运行过程中最多只出现几次（以进程的一个生命周期来计算：即从进程启动到进程退出），可以使用NOTICE级别日志；
比如：进程初始化的重要流程。
b、重要系统状态变迁：
比如：链路状态变换、主备倒换等。


4.3	日志内容精简充分原则：
日志内容要简练，既清楚说明问题，又不拖泥带水。
对于引起此错误的全局变量、关键变量、参数、返回值、系统调用错误码、错误原因描述都需要打印到错误日志中。
如：杜绝只打印“Call fun fail!”类似打印，需要打印出调用函数的输入、输出参数、返回值等重要信息“Call fun fail, param1 = %d, param2 = %d, retcode =  %d”。


4.4	日志输出格式要求：
1、日志输出内容中禁止使用换行：
日志换行，导致整理、分析日志时比较困难，增加工作量。

2、日志打印禁止使用<或[特殊字符：
<文件名（不带全路径，只是文件名） ： 行号>  [函数名]   
除了上面的文件名、函数名允许带特殊字符“<”、“>”、“[”、“]”外，不允许日志内容再带上述特殊字符。
有些日志分析工具或人工分析日志打印时，会默认“<”后面是文件名、“[”后面是函数名。如果日志内容中也包含“<”、“[”，将导致分析出错。

4.5	日志避免冗余打印建议：

容易引起洪泛的打印，不打印ERR日志；或者模块内增加控制变量控制ERR日志输出量。
比如：每秒定时器查询数据库操作，如果数据库操作失败，则第一次打印日志；后续操作失败，则增加控制变量，每分钟打印一次。

如下是一些典型场景，避免冗余打印的建议：
4.5.1	大错误引发小错误，酌情只打印大错误日志，不打印小错误日志：
比如链路下面有很多节点；当链路出现问题时，其下面的节点肯定都故障。
这时候，只打印链路日志，节点不再打印（这样，能节省很多ERR日志，又能准确定位）。

4.5.2	循环体内打印，考虑把日志挪到循环外层，避免过多日志打印：
考虑把日志打印移到循环之外。
如：
for ()
{
    if (发生错误)
    {
        Log_Text();         -- 打印错误日志
    }
}
如果if判断的错误是不依赖于循环计数的，就把该日志移到循环之外；如果依赖于计数，则考虑如何避免每次循环都打印，可以通过增加控制变量，尽量减少日志。

4.5.3	函数接口调用打印：
模块间接口，把日志信息都打印清楚：
调用错误，必须打印，并且需要把输入参数、返回的错误码等信息都打印出来，而不仅仅是打印调用某接口错误。我们需要的是有助于定位的信息，不是告诉别人发生了错误，更重要的是告诉别人发生错误的原因。

模块内接口，可以考虑减少调用错误相关的日志打印：
调用错误，可以根据本模块实际，不用每次调用都打印错误，但最底层的函数错误必须打印，上层的调用可以自行考虑是否不打印。
前提：必须保证不降低定位能力，保证提供足够的定位信息。

4.5.4	定时任务打印，避免每次出错都打印：
（1）、状态变换日志：
当状态发生变换（比如设备由好变坏或者由坏变好时，进行打印）；
状态由好变坏，打印ERR级别；状态由坏变好，打印NOTICE级别。

（2）、控制ERR打印：
如果每次定时任务都是设备坏的状态，则考虑不用每次都打印，但要把每次错误中的重要信息记录下来；每隔一定时间（比如：1分钟）输出保存的信息或者通过软调等其他方式获取。

4.5.5	数据库、资源异常，控制过多打印
最重要的是要考虑业务：为什么业务会出现此种异常？此种异常是否会发生？是不是真的把该异常当作错误来处理，会不会不应该认为是错误？

更多的是考虑业务层面，如何来避免此种异常。

如果异常是正常情况，比如进行GT翻译，本来就有找不到记录的情况，找不到记录是正常，已经有其他可观察的输出，不需要找不到记录就打印。
如果异常是错误情况，则进行日志打印，并考虑模块内增加控制变量，控制重复日志输出。

4.5.6	无法确认是否是系统错误时，不打印ERR日志，而修改为WARN或INFO日志，并考虑增加其他定位手段方法：
比如承载模块的MGC_Add_Event函数中查询端点失败，可能是正确的场景，也可能是系统错误。比如UMG的单板故障（非业务使用的），将导致MSX3000大量的查询失败。建议将级别设置为WARN。
比如DB的数据查询接口。数据不存在时可能是正常的，也可能是系统错误，调用的频度非常高，建议级别设置为INFO级别或者WARN级别。


# 前言

今年有机会进入一个初创团队，从以前的纯服务端开发接触到devops，有了从零开始构建企业级服务的机会。特意记录下构建过程中遇到的问题，作为一种分享，同时也是总结。

构建服务架构之初，首先确定了几条原则，在之后搭建的过程中主要围绕这几条原则来做决定。

**少造轮子**

造轮子是很多同学喜欢做的，其实对个人而言我是提倡尝试造轮子的，在这个过程中能够接触到大量处理问题的细节，从而学到很多书本以外的实战经验。但由于目前团队是初创，人员非常紧张，在这种情况下，需要集中资源优先做优先级最高的东西。自己造的轮子很有可能会因为设计不够周全、测试不够充分、维护人员不足等，埋下一个又一个的坑。因此在开源项目非常成熟并且没有太多定制需求的情况下，前期是坚决杜绝造轮子的。

**微服务化**

微服务概念越来越普及的今天，其重要性已经毋庸置疑。业务发展情况是技术同学很难提前预估的，我们很难设计出一个完美的架构能够适用于业务发展的所有阶段。因此最好的方式就是在设计之初便实行微服务化，在业务的各个阶段，可以根据当前不同的需求升级各个服务。另外，微服务的一个重要前提是界定好任务区间，而不是为了微服务化而硬拆分出各种服务模块，这样反而增加维护成本。

**docker化**

相信做过服务端的同学都或多或少遇到过环境不一致导致的问题，开发环境、测试环境、线上环境总是会因为各种各样的原因没有办法完全保持一致，甚至部署在同在一台机器上的服务依赖的类库版本不同也是经常遇到的棘手问题，而docker在解决这个问题上非常优秀。docker这几年也越来越成熟，社区也非常活跃，已经有越来越多的公司开始使用docker，风险相对还是较小的。docker也对微服务的普及做出了卓越的贡献。

**重视基础服务**

很多人都说小公司业务简单人员又少，什么事情都简单来好了，不用搞的太复杂。我认为这是一种误解。小公司正是因为人少，才需要节省大家的精力，专注到最关键的事情上，重复性劳动是一定要避免的。良好的基础服务对于研发人员来说，才能更好的把控线上服务质量，也能够更快的支持新业务。


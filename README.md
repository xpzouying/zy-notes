# zy-notes

- [Homelab - Intel NUC 11 安装Esxi 7](./tech-notes/nuc11/README.md)

## Reading Notes

### 架构学习



- [万字长文！Go 后台项目架构思考与重构](https://www.aminer.cn/research_report/5ea534c2ab6e30e67b2c8f6d)

  本文介绍腾讯云团队k8s集群管理的项目的代码重构过程。
  目前，我在后端开发过程中，希望找到一些在代码规范、代码维护的最佳实践。此文中有一些观点很认可，
  - 我们现在也是走大概MVC这一套，这一套感觉不是太适合Golang。
  - 完全的DDD太难在团队内部落地，并且个人觉得不适合国内大部分的互联网公司。因为1)模块的拆分及边界划分太难了，也没有那么多人配合来定义以及提供时间来推行；2)大部分互联网公司由业务驱动，技术人员在做模块划分、规范推进的工作，短期收益难以评估。
  
  此文介绍的v2版本，后续我会借鉴在组内推进，看看后续效果如何。

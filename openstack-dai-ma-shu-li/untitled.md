# oslo.reports

在OpenStack的（生产）部署中出现问题时，收集调试数据是分类和最终解决问题的关键的第一步。 像Nova这样的项目已广泛使用了日志记录功能，该功能可生成大量数据。 但是，这不能使管理员获得有关系统当前活动状态的准确视图。 例如，正在运行哪些线程，有效的配置参数等等。

项目oslo.reports托管了一个通用的错误报告生成框架，称为“guru meditation report”，以解决上述问题


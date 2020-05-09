# Activiti 配置

默认通过加载一个默认的配置文件 ==**activiti.cfg.xml**==, 构建出 ==**ProcessEngineConfiguration**== 配置对象, 然后通过 ==**ProcessEngineConfiguration**== 对象构建出 ==**ProcessEngine**==, 然后就可以通过 ==**ProcessEngine**== 构建出各种 ==**xxxService**==(例如 ==**RepositoryService**==)


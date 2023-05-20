####
- InfluxDB 与 Prometheus 两个时序数据库可以说是在一个十字路口，背向而行的两个数据库。
- InfluxDB 是 push 的方式获取监控指标数据， Prometheus 是 pull 的方式获取监控指标数据，
- promethues 的生态也很完善，比如我们可以使用 cortex 来实现 多租户的管理， influxDB ，还不清楚。这里需要简单的去看一下 influxDB 和 prometheus 两个数据库，做一个比较。

- 一般的，使用 Telegraf + InfluxDB + Grafana + Kapacitor 搭建一套监控体系
- 一般的，使用 Exporter+ Prometheus+ Grafana + Alertmanager 搭建一套监控体系



- 如果只考虑监控， Prometheus 是最好的选择，至少在 Prometheus 和 InfluxDB 里面。 Prometheus 是最优秀的。
- 但是，如果除了监控，还会有其他的一些业务指标，InfluxDB 是比较合适的。
- 最好的情况是，Promnetheus 和 InfluxDB 共存，负责不同的监控

---
title: Mysql不同发行版的区别
categories:
  - 数据库
tags:
  - mysql
abbrlink: df0ea61
date: 2018-05-29 20:39:39
---

- MySQL 常见的发行版本
  - MySQL 官方版本（社区版、企业版）
  - Percona MySQL：在 MySQL 官方发行版上进行二次开发
  - MariaDB
- 各个发行版本之间的区别和优缺点

|            | MySQL                                     | Percona MySQL                              | MariaDB                                 |
| ---------- | ----------------------------------------- | ------------------------------------------ | --------------------------------------- |
| 服务器特性 | 开源                                      | 开源                                       | 开源                                    |
|            | 支持分区表                                | 支持分区表                                 | 支持分区表                              |
|            | InnoDB                                    | XtraDB                                     | XtraDB                                  |
|            | 企业版监控工具<br />社区版不提供          | Percon Monitor 工具                        | Monyog                                  |
|            |                                           |                                            |                                         |
| 高可用特性 | 基于日志点复制                            | 基于日志点复制                             | 基于日志点复制                          |
|            | 基于 Gtid 复制                            | 基于 Gtid 复制                             | 基于 Gtid 复制，但 Gtid 同 MySQL 不兼容 |
|            | MGR                                       | MGR & PXC                                  | Galera Cluster                          |
|            | MySQL Router                              | Proxy SQL                                  | MaxScale                                |
|            |                                           |                                            |                                         |
| 安全特性   | 企业级防火墙                              | ProxySQL FireWall                          | MaxScale FireWall                       |
|            | 企业版用户审计                            | 审计日志                                   | 审计日志                                |
|            | 用户密码生命周期                          | 用户密码生命周期                           |                                         |
|            | sha256_password<br />caching_sha_password | sha256_password<br />caching_sha_pass_word | ed25519<br />sha256_password            |
|            |                                           |                                            |                                         |
| 开发及管理 | 窗口函数(8.0)                             | 窗口函数(8.0)                              | 窗口函数(10.2)                          |
|            | -                                         | -                                          | 支持基于日志回滚                        |
|            | -                                         | -                                          | 支持记在表中记录修改                    |
|            | Super read_only                           | Super read_only                            |                                         |


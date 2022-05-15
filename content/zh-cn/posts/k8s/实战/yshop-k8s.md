+++
title = "Yshop K8s"
date = 2022-05-15T14:46:38+08:00
featured = false
draft = false
comment = true
toc = true
reward = true
pinned = false
categories= [
"云原生"
]
tags = [
"云原生",
"k8s"
]
series = [
"k8s实战"
]
images = []

+++



<!--more-->

## nacos部署

xx

初始化mysql

`nacos-mysql.sql`

```sql
/*
 Navicat Premium Data Transfer

 Source Server         : nginx-nfs
 Source Server Type    : MySQL
 Source Server Version : 50736
 Source Host           : 106.53.24.153:3306
 Source Schema         : yshop_cloud_nacos

 Target Server Type    : MySQL
 Target Server Version : 50736
 File Encoding         : 65001

 Date: 15/05/2022 22:53:45
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for config_info
-- ----------------------------
DROP TABLE IF EXISTS `config_info`;
CREATE TABLE `config_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) COLLATE utf8_bin NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) COLLATE utf8_bin DEFAULT NULL,
  `content` longtext COLLATE utf8_bin NOT NULL COMMENT 'content',
  `md5` varchar(32) COLLATE utf8_bin DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COLLATE utf8_bin COMMENT 'source user',
  `src_ip` varchar(50) COLLATE utf8_bin DEFAULT NULL COMMENT 'source ip',
  `app_name` varchar(128) COLLATE utf8_bin DEFAULT NULL,
  `tenant_id` varchar(128) COLLATE utf8_bin DEFAULT '' COMMENT '租户字段',
  `c_desc` varchar(256) COLLATE utf8_bin DEFAULT NULL,
  `c_use` varchar(64) COLLATE utf8_bin DEFAULT NULL,
  `effect` varchar(64) COLLATE utf8_bin DEFAULT NULL,
  `type` varchar(64) COLLATE utf8_bin DEFAULT NULL,
  `c_schema` text COLLATE utf8_bin,
  `encrypted_data_key` text COLLATE utf8_bin NOT NULL COMMENT '秘钥',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB AUTO_INCREMENT=46 DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';

-- ----------------------------
-- Records of config_info
-- ----------------------------
BEGIN;
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (23, 'application-dev.yml', 'DEFAULT_GROUP', '# Spring\nspring:\n  jackson:\n    serialization:\n      write-dates-as-timestamps: true\n  datasource:\n    username: root\n    password: 123456\n  redis:\n    database: 6 \n    host: 127.0.0.1\n    port: 6379\n    password:\n    lettuce:\n      pool:\n        min-idle: 8\n        max-idle: 500\n        max-active: 2000\n        max-wait: 10000\n    timeout: 5000\n#请求处理的超时时间\nribbon:\n  ReadTimeout: 10000\n  ConnectTimeout: 10000\n\n# feign 配置\nfeign:\n  sentinel:\n    enabled: true\n  okhttp:\n    enabled: true\n  httpclient:\n    enabled: false\n  client:\n    config:\n      default:\n        connectTimeout: 10000\n        readTimeout: 10000\n  compression:\n    request:\n      enabled: true\n    response:\n      enabled: true\n\n# 暴露监控端点\nmanagement:\n  endpoints:\n    web:\n      exposure:\n        include: \'*\'\n\n# 认证配置\nsecurity:\n  oauth2:\n    client:\n      client-id: yshop\n      client-secret: 123456\n      scope: server\n    resource:\n      loadBalanced: true\n      token-info-uri: http://yshop-auth/oauth/check_token\n    ignore:\n      urls:\n        - /v2/api-docs\n        - /actuator/**\n        - /user/info/*\n        - /api/auth/**\n        - /weixinOauth/**\n        - /shopindex/**\n        - /operlog\n        - /logininfor\n        - /wechat/notify\n        - /notify/refund\n        - /wechat/serve\n        - /wechat/auth\n        - /wxapp/config\n        - /wechat/config\n        - /logs/save\n        - /file/**\n        - /upload/**\n        - /localStorage/**\n        - /admin/order/detail/*\n        - /order/paySuccess\n        - /article/**\n        - /order/retrunStock\n        - /canvas/**\n        - /products/integral\n        - /yxSystemConfig/getData\n# 文件存储路径\nfile:\n  path: /home/yshop/file/\n  avatar: /home/yshop/avatar/\n  # 文件大小 /M\n  maxSize: 100\n  avatarMaxSize: 5\n\nmybatis-plus:\n  check-config-location: true\n  mapperLocations: classpath:mapper/**/*.xml\n  configuration:\n    map-underscore-to-camel-case: true\n    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl\n  global-config:\n    db-config:\n      id-type: auto\n      logic-delete-value: 1\n      logic-not-delete-value: 0\n\n\n#是否允许生成代码，生产环境设置为false\ngenerator:\n  enabled: true\n\n\n# swagger 配置\nswagger:\n  title: 系统模块接口文档\n  license: Powered By yshop\n  licenseUrl: https://www.yixiang.co\n  authorization:\n    name: Yshop OAuth\n    auth-regex: ^.*$\n    authorization-scope-list:\n      - scope: server\n        description: 客户端授权范围\n    token-url-list:\n      - http://localhost:8081/auth/oauth/token\n\ntask:\n  pool:\n    # 核心线程池大小\n    core-pool-size: 10\n    # 最大线程数\n    max-pool-size: 30\n    # 活跃时间\n    keep-alive-seconds: 60\n    # 队列容量\n    queue-capacity: 50\n\nyshop:\n  tenant:\n    tenantIdColumn: tenant_id\n    enabled: true', '2028615191672ea337ac5f6ab056c1f1', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', '通用配置', NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (24, 'application-prod.yml', 'DEFAULT_GROUP', '# Spring\nspring:\n  jackson:\n    serialization:\n      write-dates-as-timestamps: true\n  datasource:\n    username: root\n    password: barByWXG\n  redis:\n    database: 6 \n    host: 127.0.0.1\n    port: 6399\n    password: 6379@@6379\n    lettuce:\n      pool:\n        min-idle: 8\n        max-idle: 500\n        max-active: 2000\n        max-wait: 10000\n    timeout: 5000\n#请求处理的超时时间\nribbon:\n  ReadTimeout: 10000\n  ConnectTimeout: 10000\n\n# feign 配置\nfeign:\n  sentinel:\n    enabled: true\n  okhttp:\n    enabled: true\n  httpclient:\n    enabled: false\n  client:\n    config:\n      default:\n        connectTimeout: 10000\n        readTimeout: 10000\n  compression:\n    request:\n      enabled: true\n    response:\n      enabled: true\n\n# 暴露监控端点\nmanagement:\n  endpoints:\n    web:\n      exposure:\n        include: \'*\'\n\n# 认证配置\nsecurity:\n  oauth2:\n    client:\n      client-id: yshop\n      client-secret: 123456\n      scope: server\n    resource:\n      loadBalanced: true\n      token-info-uri: http://yshop-auth/oauth/check_token\n    ignore:\n      urls:\n        - /v2/api-docs\n        - /actuator/**\n        - /user/info/*\n        - /api/auth/**\n        - /weixinOauth/**\n        - /shopindex/**\n        - /operlog\n        - /auth/token/logout\n        - /logininfor\n        - /wechat/notify\n        - /notify/refund\n        - /wechat/serve\n        - /wechat/auth\n        - /wxapp/config\n        - /wechat/config\n        - /logs/save\n        - /file/**\n        - /upload/**\n        - /localStorage/**\n        - /admin/order/detail/*\n        - /order/paySuccess\n        - /article/**\n        - /order/retrunStock\n        - /canvas/**\n        - /products/integral\n        - /yxSystemConfig/getData\n# 文件存储路径\nfile:\n  path: /home/yshop/file/\n  avatar: /home/yshop/avatar/\n  # 文件大小 /M\n  maxSize: 100\n  avatarMaxSize: 5\n\nmybatis-plus:\n  check-config-location: true\n  mapperLocations: classpath:mapper/**/*.xml\n  configuration:\n    map-underscore-to-camel-case: true\n    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl\n  global-config:\n    db-config:\n      id-type: auto\n      logic-delete-value: 1\n      logic-not-delete-value: 0\n\n\n#是否允许生成代码，生产环境设置为false\ngenerator:\n  enabled: true\n\n\n# swagger 配置\nswagger:\n  title: 系统模块接口文档\n  license: Powered By yshop\n  licenseUrl: https://www.yixiang.co\n  authorization:\n    name: Yshop OAuth\n    auth-regex: ^.*$\n    authorization-scope-list:\n      - scope: server\n        description: 客户端授权范围\n    token-url-list:\n      - http://localhost:8081/auth/oauth/token\n\ntask:\n  pool:\n    # 核心线程池大小\n    core-pool-size: 10\n    # 最大线程数\n    max-pool-size: 30\n    # 活跃时间\n    keep-alive-seconds: 60\n    # 队列容量\n    queue-capacity: 50', '0613a2bfa3c30c452a613d66824fe61c', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', '通用配置', NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (25, 'yshop-gateway-dev.yml', 'DEFAULT_GROUP', 'spring:\n  cloud:\n    gateway:\n      discovery:\n        locator:\n          lowerCaseServiceId: true\n          enabled: true\n      routes:\n        # 认证中心\n        - id: yshop-auth\n          uri: lb://yshop-auth\n          predicates:\n            - Path=/auth/**\n          filters:\n            # 验证码处理\n            - ValidateCodeFilter\n            - StripPrefix=1\n            - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin Access-Control-Allow-Methods Access-Control-Allow-Headers Vary\n        # 系统模块\n        - id: yshop-upms\n          uri: lb://yshop-upms\n          predicates:\n            - Path=/system/**\n          filters:\n            - name: BlackListUrlFilter\n              args:\n                blacklistUrl:\n                  - /user/info/*\n            - StripPrefix=1\n            - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin Access-Control-Allow-Methods Access-Control-Allow-Headers Vary\n       # 系统模块\n        - id: yshop-gen\n          uri: lb://yshop-gen\n          predicates:\n            - Path=/gen/**\n          filters:\n            - name: BlackListUrlFilter\n              args:\n                blacklistUrl:\n                  - /user/info/*\n            - StripPrefix=1\n            - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin Access-Control-Allow-Methods Access-Control-Allow-Headers Vary\n         # 系统模块\n        - id: yshop-mall\n          uri: lb://yshop-mall\n          predicates:\n            - Path=/mall/**\n          filters:\n            - name: BlackListUrlFilter\n              args:\n                blacklistUrl:\n                  - /user/info/*\n            - StripPrefix=1\n            - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin Access-Control-Allow-Methods Access-Control-Allow-Headers Vary\n           # 系统模块\n        - id: yshop-weixin\n          uri: lb://yshop-weixin\n          predicates:\n            - Path=/weixin/**\n          filters:\n            - name: BlackListUrlFilter\n              args:\n                blacklistUrl:\n                  - /user/info/*\n            - StripPrefix=1\n            - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin Access-Control-Allow-Methods Access-Control-Allow-Headers Vary\n   \n  \n', '67f1760476191c791e9321a6d45ce3b5', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', NULL, NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (26, 'yshop-auth-dev.yml', 'DEFAULT_GROUP', 'spring: \n  datasource:\n    driver-class-name: com.mysql.cj.jdbc.Driver\n    url: jdbc:mysql://127.0.0.1:3306/yshop_cloud_upms?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8\n\n#seata配置\nseata:\n  enabled: true\n  application-id: ${spring.application.name}\n  tx-service-group: mall_service_group_dev\n  config:\n    type: nacos\n    nacos:\n      namespace:\n      serverAddr: 127.0.0.1:8848\n      group: SEATA_GROUP_DEV\n      userName: \"nacos\"\n      password: \"nacos\"\n  registry:\n    type: nacos\n    nacos:\n      application: seata-server\n      serverAddr: 127.0.0.1:8848\n      group: SEATA_GROUP_DEV\n      namespace:\n      userName: \"nacos\"\n      password: \"nacos\"', 'bca9f30e623a679fce96b3a55d836287', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', '', NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (27, 'yshop-monitor-dev.yml', 'DEFAULT_GROUP', 'spring: \n  security:\n    user:\n      name: yshop\n      password: 123456\n  boot:\n    admin:\n      client:\n        url: http://localhost:8088\n      ui:\n        title: yshop服务状态监控\n      notify:\n        dingtalk:\n          enabled: true\n          webhook-token: https://oapi.dingtalk.com/robot/send?access_token=bc1a1982dd41473b56371f1e7fe5acfc72c48b2038712bdcef8755c7705af228', 'c41428ecdb170fad620ebcaf35455afe', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', NULL, NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (28, 'yshop-upms-dev.yml', 'DEFAULT_GROUP', '# Spring\nspring:\n  datasource:\n    driver-class-name: com.mysql.cj.jdbc.Driver\n    url: jdbc:mysql://127.0.0.1:3306/yshop_cloud_upms?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8\n\nmybatis-plus:\n  typeAliasesPackage: co.yixiang.upms.common.system,co.yixiang.upms.common.gen.domain,co.yixiang.upms.common.job.domain\n\n\n#密码加密传输，前端公钥加密，后端私钥解密\nrsa:\n  private_key: MIIBUwIBADANBgkqhkiG9w0BAQEFAASCAT0wggE5AgEAAkEA0vfvyTdGJkdbHkB8mp0f3FE0GYP3AYPaJF7jUd1M0XxFSE2ceK3k2kw20YvQ09NJKk+OMjWQl9WitG9pB6tSCQIDAQABAkA2SimBrWC2/wvauBuYqjCFwLvYiRYqZKThUS3MZlebXJiLB+Ue/gUifAAKIg1avttUZsHBHrop4qfJCwAI0+YRAiEA+W3NK/RaXtnRqmoUUkb59zsZUBLpvZgQPfj1MhyHDz0CIQDYhsAhPJ3mgS64NbUZmGWuuNKp5coY2GIj/zYDMJp6vQIgUueLFXv/eZ1ekgz2Oi67MNCk5jeTF2BurZqNLR3MSmUCIFT3Q6uHMtsB9Eha4u7hS31tj1UWE+D+ADzp59MGnoftAiBeHT7gDMuqeJHPL4b+kC+gzV4FGTfhR9q3tTbklZkD2A==\n\n#七牛云\nqiniu:\n  # 文件大小 /M\n  max-size: 15\n\n#邮箱验证码有效时间/分钟\ncode:\n  expiration: 5\n\n#seata配置\nseata:\n  enabled: true\n  application-id: ${spring.application.name}\n  tx-service-group: mall_service_group_dev\n  config:\n    type: nacos\n    nacos:\n      namespace:\n      serverAddr: 127.0.0.1:8848\n      group: SEATA_GROUP_DEV\n      userName: \"nacos\"\n      password: \"nacos\"\n  registry:\n    type: nacos\n    nacos:\n      application: seata-server\n      serverAddr: 127.0.0.1:8848\n      group: SEATA_GROUP_DEV\n      namespace:\n      userName: \"nacos\"\n      password: \"nacos\"', '066b5844754093e5942e932ae1d4a17d', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', '', NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (29, 'yshop-mall-dev.yml', 'DEFAULT_GROUP', '# Spring\nspring:\n  datasource:\n    driver-class-name: com.mysql.cj.jdbc.Driver\n    url: jdbc:mysql://127.0.0.1:3306/yshop_cloud_mall?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8\n\nmybatis-plus:\n  typeAliasesPackage: co.yixiang.mall.common.domain\n  \nyshop:\n  security:\n    jwt-key: yshopmini\n    token-expired-in: 86400000\n\n#seata配置\nseata:\n  enabled: true\n  application-id: ${spring.application.name}\n  tx-service-group: mall_service_group_dev\n  config:\n    type: nacos\n    nacos:\n      namespace:\n      serverAddr: 127.0.0.1:8848\n      group: SEATA_GROUP_DEV\n      userName: \"nacos\"\n      password: \"nacos\"\n  registry:\n    type: nacos\n    nacos:\n      application: seata-server\n      serverAddr: 127.0.0.1:8848\n      group: SEATA_GROUP_DEV\n      namespace:\n      userName: \"nacos\"\n      password: \"nacos\"', '8069a71197e9af050208990242a3ceb9', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', '', NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (30, 'yshop-weixin-dev.yml', 'DEFAULT_GROUP', '# Spring\nspring:\n  datasource:\n    driver-class-name: com.mysql.cj.jdbc.Driver\n    url: jdbc:mysql://127.0.0.1:3306/yshop_cloud_weixin?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8\n\nmybatis-plus:\n  typeAliasesPackage: co.yixiang.weixin.common.domain\n\n#seata配置\nseata:\n  enabled: true\n  application-id: ${spring.application.name}\n  tx-service-group: mall_service_group_dev\n  config:\n    type: nacos\n    nacos:\n      namespace:\n      serverAddr: 127.0.0.1:8848\n      group: SEATA_GROUP_DEV\n      userName: \"nacos\"\n      password: \"nacos\"\n  registry:\n    type: nacos\n    nacos:\n      application: seata-server\n      serverAddr: 127.0.0.1:8848\n      group: SEATA_GROUP_DEV\n      namespace:\n      userName: \"nacos\"\n      password: \"nacos\"', 'e20ca200bf259b49ef58b9fae622eb48', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', '', NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (31, 'xxl-job-admin-dev.yml', 'DEFAULT_GROUP', '# xxl\nxxl:\n  job:\n    i18n: zh_CN\n    logretentiondays: 30\n    triggerpool:\n      fast.max: 200\n      slow.max: 200\n\n# mybatis\nmybatis:\n  mapper-locations: classpath:/mybatis-mapper/*Mapper.xml\n\n# spring\nspring:\n  datasource:\n    url: jdbc:mysql://127.0.0.1:3306/yshop_cloud_xxljob?characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=false&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=GMT%2B8&nullCatalogMeansCurrent=true&allowPublicKeyRetrieval=true\n    driver-class-name: com.mysql.cj.jdbc.Driver\n    username: root\n    password: 123456\n  mvc:\n    static-path-pattern: /static/**\n  freemarker:\n    suffix: .ftl\n    request-context-attribute: request\n    settings:\n      number_format: 0.##########\n  mail:\n    host: smtp.mxhichina.com\n    port: 465\n    from: xxxx@gitee.wang\n    username: xxxx@gitee.wang\n    password: xxxx\n    properties:\n      mail:\n        smtp:\n          auth: true\n          ssl.enable: true\n          starttls.enable: false\n          required: false\n# spring boot admin 配置\n\nmanagement:\n  health:\n    mail:\n      enabled: false\n  endpoints:\n    web:\n      exposure:\n        include: \'*\'\n  endpoint:\n    health:\n      show-details: ALWAYS\n', '62e22333890cd403234a532a74ae4e34', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', '', NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (32, 'xxl-job-admin-prod.yml', 'DEFAULT_GROUP', '# xxl\nxxl:\n  job:\n    i18n: zh_CN\n    logretentiondays: 30\n    triggerpool:\n      fast.max: 200\n      slow.max: 200\n\n# mybatis\nmybatis:\n  mapper-locations: classpath:/mybatis-mapper/*Mapper.xml\n\n# spring\nspring:\n  datasource:\n    url: jdbc:mysql://127.0.0.1:3306/yshop_cloud_xxljob?characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=false&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=GMT%2B8&nullCatalogMeansCurrent=true&allowPublicKeyRetrieval=true\n    driver-class-name: com.mysql.cj.jdbc.Driver\n    username: root\n    password: barByWXG\n  mvc:\n    static-path-pattern: /static/**\n  freemarker:\n    suffix: .ftl\n    request-context-attribute: request\n    settings:\n      number_format: 0.##########\n  mail:\n    host: smtp.mxhichina.com\n    port: 465\n    from: xxxx@gitee.wang\n    username: xxxx@gitee.wang\n    password: xxxx\n    properties:\n      mail:\n        smtp:\n          auth: true\n          ssl.enable: true\n          starttls.enable: false\n          required: false\n# spring boot admin 配置\n\nmanagement:\n  health:\n    mail:\n      enabled: false\n  endpoints:\n    web:\n      exposure:\n        include: \'*\'\n  endpoint:\n    health:\n      show-details: ALWAYS\n', '5ee9c887c5ea2d69722c00f886d42c88', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', '', NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (33, 'yshop-gateway-prod.yml', 'DEFAULT_GROUP', 'spring:\n  cloud:\n    gateway:\n      discovery:\n        locator:\n          lowerCaseServiceId: true\n          enabled: true\n      routes:\n        # 认证中心\n        - id: yshop-auth\n          uri: lb://yshop-auth\n          predicates:\n            - Path=/auth/**\n          filters:\n            # 验证码处理\n            - ValidateCodeFilter\n            - StripPrefix=1\n            - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin Access-Control-Allow-Methods Access-Control-Allow-Headers Vary\n        # 系统模块\n        - id: yshop-upms\n          uri: lb://yshop-upms\n          predicates:\n            - Path=/system/**\n          filters:\n            - name: BlackListUrlFilter\n              args:\n                blacklistUrl:\n                  - /user/info/*\n            - StripPrefix=1\n            - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin Access-Control-Allow-Methods Access-Control-Allow-Headers Vary\n       # 系统模块\n        - id: yshop-gen\n          uri: lb://yshop-gen\n          predicates:\n            - Path=/gen/**\n          filters:\n            - name: BlackListUrlFilter\n              args:\n                blacklistUrl:\n                  - /user/info/*\n            - StripPrefix=1\n            - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin Access-Control-Allow-Methods Access-Control-Allow-Headers Vary\n         # 系统模块\n        - id: yshop-mall\n          uri: lb://yshop-mall\n          predicates:\n            - Path=/mall/**\n          filters:\n            - name: BlackListUrlFilter\n              args:\n                blacklistUrl:\n                  - /user/info/*\n            - StripPrefix=1\n            - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin Access-Control-Allow-Methods Access-Control-Allow-Headers Vary\n           # 系统模块\n        - id: yshop-weixin\n          uri: lb://yshop-weixin\n          predicates:\n            - Path=/weixin/**\n          filters:\n            - name: BlackListUrlFilter\n              args:\n                blacklistUrl:\n                  - /user/info/*\n            - StripPrefix=1\n            - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin Access-Control-Allow-Methods Access-Control-Allow-Headers Vary\n   \n  \n', '67f1760476191c791e9321a6d45ce3b5', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', NULL, NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (34, 'yshop-auth-prod.yml', 'DEFAULT_GROUP', 'spring: \n  datasource:\n    driver-class-name: com.mysql.cj.jdbc.Driver\n    url: jdbc:mysql://127.0.0.1:3366/yshop_cloud_upms?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8\n\n#seata配置\nseata:\n  enabled: true\n  application-id: ${spring.application.name}\n  tx-service-group: auth_service_group_prod\n  config:\n    type: nacos\n    nacos:\n      namespace:\n      serverAddr: 127.0.0.1:8848\n      group: SEATA_GROUP_PROD\n      userName: \"nacos\"\n      password: \"nacos\"\n  registry:\n    type: nacos\n    nacos:\n      application: seata-server\n      serverAddr: 127.0.0.1:8848\n      group: SEATA_GROUP_PROD\n      namespace:\n      userName: \"nacos\"\n      password: \"nacos\"', '27b91922b47e5522db758068e921794d', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', '', NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (35, 'yshop-monitor-prod.yml', 'DEFAULT_GROUP', 'spring: \n  security:\n    user:\n      name: yshop\n      password: 123456\n  boot:\n    admin:\n      client:\n        url: http://localhost:8088\n      ui:\n        title: yshop服务状态监控\n      notify:\n        dingtalk:\n          enabled: true\n          webhook-token: https://oapi.dingtalk.com/robot/send?access_token=bc1a1982dd41473b56371f1e7fe5acfc72c48b2038712bdcef8755c7705af228', 'c41428ecdb170fad620ebcaf35455afe', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', NULL, NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (36, 'yshop-upms-prod.yml', 'DEFAULT_GROUP', '# Spring\nspring:\n  datasource:\n    driver-class-name: com.mysql.cj.jdbc.Driver\n    url: jdbc:mysql://127.0.0.1:3366/yshop_cloud_upms?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8\n\nmybatis-plus:\n  typeAliasesPackage: co.yixiang.upms.common.system,co.yixiang.upms.common.gen.domain,co.yixiang.upms.common.job.domain\n\n\n#密码加密传输，前端公钥加密，后端私钥解密\nrsa:\n  private_key: MIIBUwIBADANBgkqhkiG9w0BAQEFAASCAT0wggE5AgEAAkEA0vfvyTdGJkdbHkB8mp0f3FE0GYP3AYPaJF7jUd1M0XxFSE2ceK3k2kw20YvQ09NJKk+OMjWQl9WitG9pB6tSCQIDAQABAkA2SimBrWC2/wvauBuYqjCFwLvYiRYqZKThUS3MZlebXJiLB+Ue/gUifAAKIg1avttUZsHBHrop4qfJCwAI0+YRAiEA+W3NK/RaXtnRqmoUUkb59zsZUBLpvZgQPfj1MhyHDz0CIQDYhsAhPJ3mgS64NbUZmGWuuNKp5coY2GIj/zYDMJp6vQIgUueLFXv/eZ1ekgz2Oi67MNCk5jeTF2BurZqNLR3MSmUCIFT3Q6uHMtsB9Eha4u7hS31tj1UWE+D+ADzp59MGnoftAiBeHT7gDMuqeJHPL4b+kC+gzV4FGTfhR9q3tTbklZkD2A==\n\n#seata配置\nseata:\n  enabled: true\n  application-id: ${spring.application.name}\n  tx-service-group: upms_service_group_prod\n  config:\n    type: nacos\n    nacos:\n      namespace:\n      serverAddr: 127.0.0.1:8848\n      group: SEATA_GROUP_PROD\n      userName: \"nacos\"\n      password: \"nacos\"\n  registry:\n    type: nacos\n    nacos:\n      application: seata-server\n      serverAddr: 127.0.0.1:8848\n      group: SEATA_GROUP_PROD\n      namespace:\n      userName: \"nacos\"\n      password: \"nacos\"\n\n#七牛云\nqiniu:\n  # 文件大小 /M\n  max-size: 15\n\n#邮箱验证码有效时间/分钟\ncode:\n  expiration: 5', '19c80ca399e87e3ebf58e7969db860e2', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', '', NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (37, 'yshop-mall-prod.yml', 'DEFAULT_GROUP', '# Spring\nspring:\n  datasource:\n    driver-class-name: com.mysql.cj.jdbc.Driver\n    url: jdbc:mysql://127.0.0.1:3366/yshop_cloud_mall?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8\n\nmybatis-plus:\n  typeAliasesPackage: co.yixiang.mall.common.domain\n\n#seata配置\nseata:\n  enabled: true\n  application-id: ${spring.application.name}\n  tx-service-group: mall_service_group_prod\n  config:\n    type: nacos\n    nacos:\n      namespace:\n      serverAddr: 127.0.0.1:8848\n      group: SEATA_GROUP_PROD\n      userName: \"nacos\"\n      password: \"nacos\"\n  registry:\n    type: nacos\n    nacos:\n      application: seata-server\n      serverAddr: 127.0.0.1:8848\n      group: SEATA_GROUP_PROD\n      namespace:\n      userName: \"nacos\"\n      password: \"nacos\"\n\nyshop:\n  security:\n    jwt-key: yshopmini\n    token-expired-in: 86400000', '28f922026214da8074387a6dd4bf3122', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', '', NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (38, 'yshop-weixin-prod.yml', 'DEFAULT_GROUP', '# Spring\nspring:\n  datasource:\n    driver-class-name: com.mysql.cj.jdbc.Driver\n    url: jdbc:mysql://127.0.0.1:3366/yshop_cloud_weixin?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8\n\n#seata配置\nseata:\n  enabled: true\n  application-id: ${spring.application.name}\n  tx-service-group: weixin_service_group_prod\n  config:\n    type: nacos\n    nacos:\n      namespace:\n      serverAddr: 127.0.0.1:8848\n      group: SEATA_GROUP_PROD\n      userName: \"nacos\"\n      password: \"nacos\"\n  registry:\n    type: nacos\n    nacos:\n      application: seata-server\n      serverAddr: 127.0.0.1:8848\n      group: SEATA_GROUP_PROD\n      namespace:\n      userName: \"nacos\"\n      password: \"nacos\"\n\nmybatis-plus:\n  typeAliasesPackage: co.yixiang.weixin.common.domain\n', '8686c5161753560d00326a2e1572d3e0', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', '', NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (39, 'sentinel-yshop-gateway-prod', 'DEFAULT_GROUP', '[\n    {\n        \"resource\": \"yshop-auth\",\n        \"count\": 500,\n        \"grade\": 1,\n        \"limitApp\": \"default\",\n        \"strategy\": 0,\n        \"controlBehavior\": 0\n    },\n	{\n        \"resource\": \"yshop-upms\",\n        \"count\": 1000,\n        \"grade\": 1,\n        \"limitApp\": \"default\",\n        \"strategy\": 0,\n        \"controlBehavior\": 0\n    },\n	{\n        \"resource\": \"yshop-gen\",\n        \"count\": 200,\n        \"grade\": 1,\n        \"limitApp\": \"default\",\n        \"strategy\": 0,\n        \"controlBehavior\": 0\n    },\n	{\n        \"resource\": \"yshop-job\",\n        \"count\": 300,\n        \"grade\": 1,\n        \"limitApp\": \"default\",\n        \"strategy\": 0,\n        \"controlBehavior\": 0\n    }\n]', 'f482645f19e7c4be5dabce1861a93e6f', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', '', NULL, NULL, 'json', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (40, 'yshop-gen-prod.yml', 'DEFAULT_GROUP', '# Spring\nspring:\n  datasource:\n    driver-class-name: com.mysql.cj.jdbc.Driver\n    url: jdbc:mysql://localhost:3366/yshop_cloud_upms?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8\n\nmybatis-plus:\n  typeAliasesPackage: co.yixiang.upms.common.system,co.yixiang.upms.common.gen.domain,co.yixiang.upms.common.job.domain', 'ff03eb68b849980d1aa4e7ad46755328', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', NULL, NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (41, 'sentinel-yshop-gateway-dev', 'DEFAULT_GROUP', '[\n    {\n        \"resource\": \"yshop-auth\",\n        \"count\": 500,\n        \"grade\": 1,\n        \"limitApp\": \"default\",\n        \"strategy\": 0,\n        \"controlBehavior\": 0\n    },\n	{\n        \"resource\": \"yshop-upms\",\n        \"count\": 1000,\n        \"grade\": 1,\n        \"limitApp\": \"default\",\n        \"strategy\": 0,\n        \"controlBehavior\": 0\n    },\n	{\n        \"resource\": \"yshop-gen\",\n        \"count\": 200,\n        \"grade\": 1,\n        \"limitApp\": \"default\",\n        \"strategy\": 0,\n        \"controlBehavior\": 0\n    },\n	{\n        \"resource\": \"yshop-job\",\n        \"count\": 300,\n        \"grade\": 1,\n        \"limitApp\": \"default\",\n        \"strategy\": 0,\n        \"controlBehavior\": 0\n    }\n]', 'f482645f19e7c4be5dabce1861a93e6f', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', '', NULL, NULL, 'json', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (42, 'yshop-gen-dev.yml', 'DEFAULT_GROUP', '# Spring\nspring:\n  datasource:\n    driver-class-name: com.mysql.cj.jdbc.Driver\n    url: jdbc:mysql://localhost:3306/yshop_cloud_upms?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8\n\nmybatis-plus:\n  typeAliasesPackage: co.yixiang.upms.common.system,co.yixiang.upms.common.gen.domain,co.yixiang.upms.common.job.domain', 'c4bf89ed9ed68f07cd81050b95f1fddf', '2022-05-15 14:31:18', '2022-05-15 14:31:18', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', '', NULL, NULL, 'yaml', NULL, '');
INSERT INTO `config_info` (`id`, `data_id`, `group_id`, `content`, `md5`, `gmt_create`, `gmt_modified`, `src_user`, `src_ip`, `app_name`, `tenant_id`, `c_desc`, `c_use`, `effect`, `type`, `c_schema`, `encrypted_data_key`) VALUES (45, 'seataServer.properties', 'SEATA_GROUP', '#For details about configuration items, see https://seata.io/zh-cn/docs/user/configurations.html\n#Transport configuration, for client and server\ntransport.type=TCP\ntransport.server=NIO\ntransport.heartbeat=true\ntransport.enableTmClientBatchSendRequest=false\ntransport.enableRmClientBatchSendRequest=true\ntransport.enableTcServerBatchSendResponse=false\ntransport.rpcRmRequestTimeout=30000\ntransport.rpcTmRequestTimeout=30000\ntransport.rpcTcRequestTimeout=30000\ntransport.threadFactory.bossThreadPrefix=NettyBoss\ntransport.threadFactory.workerThreadPrefix=NettyServerNIOWorker\ntransport.threadFactory.serverExecutorThreadPrefix=NettyServerBizHandler\ntransport.threadFactory.shareBossWorker=false\ntransport.threadFactory.clientSelectorThreadPrefix=NettyClientSelector\ntransport.threadFactory.clientSelectorThreadSize=1\ntransport.threadFactory.clientWorkerThreadPrefix=NettyClientWorkerThread\ntransport.threadFactory.bossThreadSize=1\ntransport.threadFactory.workerThreadSize=default\ntransport.shutdown.wait=3\ntransport.serialization=seata\ntransport.compressor=none\n\n#Transaction routing rules configuration, only for the client\nservice.vgroupMapping.default_tx_group=default\n#If you use a registry, you can ignore it\nservice.default.grouplist=127.0.0.1:8091\nservice.enableDegrade=false\nservice.disableGlobalTransaction=false\n\n#Transaction rule configuration, only for the client\nclient.rm.asyncCommitBufferLimit=10000\nclient.rm.lock.retryInterval=10\nclient.rm.lock.retryTimes=30\nclient.rm.lock.retryPolicyBranchRollbackOnConflict=true\nclient.rm.reportRetryCount=5\nclient.rm.tableMetaCheckEnable=true\nclient.rm.tableMetaCheckerInterval=60000\nclient.rm.sqlParserType=druid\nclient.rm.reportSuccessEnable=false\nclient.rm.sagaBranchRegisterEnable=false\nclient.rm.sagaJsonParser=fastjson\nclient.rm.tccActionInterceptorOrder=-2147482648\nclient.tm.commitRetryCount=5\nclient.tm.rollbackRetryCount=5\nclient.tm.defaultGlobalTransactionTimeout=60000\nclient.tm.degradeCheck=false\nclient.tm.degradeCheckAllowTimes=10\nclient.tm.degradeCheckPeriod=2000\nclient.tm.interceptorOrder=-2147482648\nclient.undo.dataValidation=true\nclient.undo.logSerialization=jackson\nclient.undo.onlyCareUpdateColumns=true\nserver.undo.logSaveDays=7\nserver.undo.logDeletePeriod=86400000\nclient.undo.logTable=undo_log\nclient.undo.compress.enable=true\nclient.undo.compress.type=zip\nclient.undo.compress.threshold=64k\n#For TCC transaction mode\ntcc.fence.logTableName=tcc_fence_log\ntcc.fence.cleanPeriod=1h\n\n#Log rule configuration, for client and server\nlog.exceptionRate=100\n\n#Transaction storage configuration, only for the server. The file, DB, and redis configuration values are optional.\nstore.mode=DB\nstore.lock.mode=file\nstore.session.mode=file\n#Used for password encryption\nstore.publicKey=\n\n#If `store.mode,store.lock.mode,store.session.mode` are not equal to `file`, you can remove the configuration block.\nstore.file.dir=file_store/data\nstore.file.maxBranchSessionSize=16384\nstore.file.maxGlobalSessionSize=512\nstore.file.fileWriteBufferCacheSize=16384\nstore.file.flushDiskMode=async\nstore.file.sessionReloadReadSize=100\n\n#These configurations are required if the `store mode` is `db`. If `store.mode,store.lock.mode,store.session.mode` are not equal to `db`, you can remove the configuration block.\nstore.db.datasource=druid\nstore.db.dbType=mysql\nstore.db.driverClassName=com.mysql.jdbc.Driver\nstore.db.url=jdbc:mysql://172.31.0.12:3306/seata?useUnicode=true&rewriteBatchedStatements=true\nstore.db.user=root\nstore.db.password=xuzhijun256\nstore.db.minConn=5\nstore.db.maxConn=30\nstore.db.globalTable=global_table\nstore.db.branchTable=branch_table\nstore.db.distributedLockTable=distributed_lock\nstore.db.queryLimit=100\nstore.db.lockTable=lock_table\nstore.db.maxWait=5000\n\n#These configurations are required if the `store mode` is `redis`. If `store.mode,store.lock.mode,store.session.mode` are not equal to `redis`, you can remove the configuration block.\nstore.redis.mode=single\nstore.redis.single.host=127.0.0.1\nstore.redis.single.port=6379\nstore.redis.sentinel.masterName=\nstore.redis.sentinel.sentinelHosts=\nstore.redis.maxConn=10\nstore.redis.minConn=1\nstore.redis.maxTotal=100\nstore.redis.database=0\nstore.redis.password=\nstore.redis.queryLimit=100\n\n#Transaction rule configuration, only for the server\nserver.recovery.committingRetryPeriod=1000\nserver.recovery.asynCommittingRetryPeriod=1000\nserver.recovery.rollbackingRetryPeriod=1000\nserver.recovery.timeoutRetryPeriod=1000\nserver.maxCommitRetryTimeout=-1\nserver.maxRollbackRetryTimeout=-1\nserver.rollbackRetryTimeoutUnlockEnable=false\nserver.distributedLockExpireTime=10000\nserver.xaerNotaRetryTimeout=60000\nserver.session.branchAsyncQueueSize=5000\nserver.session.enableBranchAsyncRemove=false\n\n#Metrics configuration, only for the server\nmetrics.enabled=false\nmetrics.registryType=compact\nmetrics.exporterList=prometheus\nmetrics.exporterPrometheusPort=9898', 'c61c4c7ba13cab351379db7d598cb9c7', '2022-05-15 14:50:55', '2022-05-15 14:50:55', NULL, '120.229.78.27', '', 'e9234393-1ce4-4195-9937-5328ef05872b', NULL, NULL, NULL, 'properties', NULL, '');
COMMIT;

-- ----------------------------
-- Table structure for config_info_aggr
-- ----------------------------
DROP TABLE IF EXISTS `config_info_aggr`;
CREATE TABLE `config_info_aggr` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) COLLATE utf8_bin NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) COLLATE utf8_bin NOT NULL COMMENT 'group_id',
  `datum_id` varchar(255) COLLATE utf8_bin NOT NULL COMMENT 'datum_id',
  `content` longtext COLLATE utf8_bin NOT NULL COMMENT '内容',
  `gmt_modified` datetime NOT NULL COMMENT '修改时间',
  `app_name` varchar(128) COLLATE utf8_bin DEFAULT NULL,
  `tenant_id` varchar(128) COLLATE utf8_bin DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';

-- ----------------------------
-- Records of config_info_aggr
-- ----------------------------
BEGIN;
COMMIT;

-- ----------------------------
-- Table structure for config_info_beta
-- ----------------------------
DROP TABLE IF EXISTS `config_info_beta`;
CREATE TABLE `config_info_beta` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) COLLATE utf8_bin NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) COLLATE utf8_bin NOT NULL COMMENT 'group_id',
  `app_name` varchar(128) COLLATE utf8_bin DEFAULT NULL COMMENT 'app_name',
  `content` longtext COLLATE utf8_bin NOT NULL COMMENT 'content',
  `beta_ips` varchar(1024) COLLATE utf8_bin DEFAULT NULL COMMENT 'betaIps',
  `md5` varchar(32) COLLATE utf8_bin DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COLLATE utf8_bin COMMENT 'source user',
  `src_ip` varchar(50) COLLATE utf8_bin DEFAULT NULL COMMENT 'source ip',
  `tenant_id` varchar(128) COLLATE utf8_bin DEFAULT '' COMMENT '租户字段',
  `encrypted_data_key` text COLLATE utf8_bin NOT NULL COMMENT '秘钥',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';

-- ----------------------------
-- Records of config_info_beta
-- ----------------------------
BEGIN;
COMMIT;

-- ----------------------------
-- Table structure for config_info_tag
-- ----------------------------
DROP TABLE IF EXISTS `config_info_tag`;
CREATE TABLE `config_info_tag` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) COLLATE utf8_bin NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) COLLATE utf8_bin NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) COLLATE utf8_bin DEFAULT '' COMMENT 'tenant_id',
  `tag_id` varchar(128) COLLATE utf8_bin NOT NULL COMMENT 'tag_id',
  `app_name` varchar(128) COLLATE utf8_bin DEFAULT NULL COMMENT 'app_name',
  `content` longtext COLLATE utf8_bin NOT NULL COMMENT 'content',
  `md5` varchar(32) COLLATE utf8_bin DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COLLATE utf8_bin COMMENT 'source user',
  `src_ip` varchar(50) COLLATE utf8_bin DEFAULT NULL COMMENT 'source ip',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';

-- ----------------------------
-- Records of config_info_tag
-- ----------------------------
BEGIN;
COMMIT;

-- ----------------------------
-- Table structure for config_tags_relation
-- ----------------------------
DROP TABLE IF EXISTS `config_tags_relation`;
CREATE TABLE `config_tags_relation` (
  `id` bigint(20) NOT NULL COMMENT 'id',
  `tag_name` varchar(128) COLLATE utf8_bin NOT NULL COMMENT 'tag_name',
  `tag_type` varchar(64) COLLATE utf8_bin DEFAULT NULL COMMENT 'tag_type',
  `data_id` varchar(255) COLLATE utf8_bin NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) COLLATE utf8_bin NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) COLLATE utf8_bin DEFAULT '' COMMENT 'tenant_id',
  `nid` bigint(20) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`nid`),
  UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';

-- ----------------------------
-- Records of config_tags_relation
-- ----------------------------
BEGIN;
COMMIT;

-- ----------------------------
-- Table structure for group_capacity
-- ----------------------------
DROP TABLE IF EXISTS `group_capacity`;
CREATE TABLE `group_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `group_id` varchar(128) COLLATE utf8_bin NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表示整个集群',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数，，0表示使用默认值',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_group_id` (`group_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量信息表';

-- ----------------------------
-- Records of group_capacity
-- ----------------------------
BEGIN;
COMMIT;

-- ----------------------------
-- Table structure for his_config_info
-- ----------------------------
DROP TABLE IF EXISTS `his_config_info`;
CREATE TABLE `his_config_info` (
  `id` bigint(64) unsigned NOT NULL,
  `nid` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `data_id` varchar(255) COLLATE utf8_bin NOT NULL,
  `group_id` varchar(128) COLLATE utf8_bin NOT NULL,
  `app_name` varchar(128) COLLATE utf8_bin DEFAULT NULL COMMENT 'app_name',
  `content` longtext COLLATE utf8_bin NOT NULL,
  `md5` varchar(32) COLLATE utf8_bin DEFAULT NULL,
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `src_user` text COLLATE utf8_bin,
  `src_ip` varchar(50) COLLATE utf8_bin DEFAULT NULL,
  `op_type` char(10) COLLATE utf8_bin DEFAULT NULL,
  `tenant_id` varchar(128) COLLATE utf8_bin DEFAULT '' COMMENT '租户字段',
  `encrypted_data_key` text COLLATE utf8_bin NOT NULL COMMENT '秘钥',
  PRIMARY KEY (`nid`),
  KEY `idx_gmt_create` (`gmt_create`),
  KEY `idx_gmt_modified` (`gmt_modified`),
  KEY `idx_did` (`data_id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';



-- ----------------------------
-- Table structure for permissions
-- ----------------------------
DROP TABLE IF EXISTS `permissions`;
CREATE TABLE `permissions` (
  `role` varchar(50) NOT NULL,
  `resource` varchar(255) NOT NULL,
  `action` varchar(8) NOT NULL,
  UNIQUE KEY `uk_role_permission` (`role`,`resource`,`action`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Records of permissions
-- ----------------------------
BEGIN;
COMMIT;

-- ----------------------------
-- Table structure for roles
-- ----------------------------
DROP TABLE IF EXISTS `roles`;
CREATE TABLE `roles` (
  `username` varchar(50) NOT NULL,
  `role` varchar(50) NOT NULL,
  UNIQUE KEY `idx_user_role` (`username`,`role`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Records of roles
-- ----------------------------
BEGIN;
INSERT INTO `roles` (`username`, `role`) VALUES ('nacos', 'ROLE_ADMIN');
COMMIT;

-- ----------------------------
-- Table structure for tenant_capacity
-- ----------------------------
DROP TABLE IF EXISTS `tenant_capacity`;
CREATE TABLE `tenant_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `tenant_id` varchar(128) COLLATE utf8_bin NOT NULL DEFAULT '' COMMENT 'Tenant ID',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';

-- ----------------------------
-- Records of tenant_capacity
-- ----------------------------
BEGIN;
COMMIT;

-- ----------------------------
-- Table structure for tenant_info
-- ----------------------------
DROP TABLE IF EXISTS `tenant_info`;
CREATE TABLE `tenant_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `kp` varchar(128) COLLATE utf8_bin NOT NULL COMMENT 'kp',
  `tenant_id` varchar(128) COLLATE utf8_bin DEFAULT '' COMMENT 'tenant_id',
  `tenant_name` varchar(128) COLLATE utf8_bin DEFAULT '' COMMENT 'tenant_name',
  `tenant_desc` varchar(256) COLLATE utf8_bin DEFAULT NULL COMMENT 'tenant_desc',
  `create_source` varchar(32) COLLATE utf8_bin DEFAULT NULL COMMENT 'create_source',
  `gmt_create` bigint(20) NOT NULL COMMENT '创建时间',
  `gmt_modified` bigint(20) NOT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';

-- ----------------------------
-- Records of tenant_info
-- ----------------------------
BEGIN;
INSERT INTO `tenant_info` (`id`, `kp`, `tenant_id`, `tenant_name`, `tenant_desc`, `create_source`, `gmt_create`, `gmt_modified`) VALUES (1, '1', 'e9234393-1ce4-4195-9937-5328ef05872b', 'YSHOP_CLOUD', 'yshop', 'nacos', 1652624714921, 1652624714921);
COMMIT;

-- ----------------------------
-- Table structure for users
-- ----------------------------
DROP TABLE IF EXISTS `users`;
CREATE TABLE `users` (
  `username` varchar(50) NOT NULL,
  `password` varchar(500) NOT NULL,
  `enabled` tinyint(1) NOT NULL,
  PRIMARY KEY (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Records of users
-- ----------------------------
BEGIN;
INSERT INTO `users` (`username`, `password`, `enabled`) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', 1);
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;

```



## seata部署

⚠️ seata latest镜像有bug（`docker.io/seataio/seata-server:latest`,  `2de39e877c79`）无法注册到nacos，v1.4.2可正常注册

```yaml
apiVersion: v1
kind: Service
metadata:
  name: seata-server
  namespace: yshop-cloud
  labels:
    k8s-app: seata-server
spec:
  type: NodePort
  ports:
    - port: 8091
      nodePort: 30091
      protocol: TCP
      name: http
  selector:
    k8s-app: seata-server

---


apiVersion: apps/v1
kind: Deployment
metadata:
  name: seata-server
  namespace: yshop-cloud
  labels:
    k8s-app: seata-server
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: seata-server
  template:
    metadata:
      labels:
        k8s-app: seata-server
    spec:
      containers:
        - name: seata-server
          image: docker.io/seataio/seata-server:1.4.2
          imagePullPolicy: IfNotPresent
          env:
            - name: SEATA_CONFIG_NAME
              value: file:/root/seata-config/registry
          ports:
            - name: http
              containerPort: 8091
              protocol: TCP
          volumeMounts:
            - name: seata-config
              mountPath: /root/seata-config
      volumes:
        - name: seata-config
          configMap:
            name: seata-server-config

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: seata-server-config
data:
  registry.conf: |
    registry {
        type = "nacos"
        nacos {
          application = "seata-server"
          serverAddr = "nacos-0.nacos-headless.yshop-cloud.svc.cluster.local:8848,nacos-1.nacos-headless.yshop-cloud.svc.cluster.local:8848,nacos-2.nacos-headless.yshop-cloud.svc.cluster.local:8848"
          group = "SEATA_GROUP"
        	namespace = "e9234393-1ce4-4195-9937-5328ef05872b"
        }
    }
    config {
      type = "nacos"
      nacos {
        serverAddr = "nacos-0.nacos-headless.yshop-cloud.svc.cluster.local:8848,nacos-1.nacos-headless.yshop-cloud.svc.cluster.local:8848,nacos-2.nacos-headless.yshop-cloud.svc.cluster.local:8848"
        group = "SEATA_GROUP"
        namespace = "e9234393-1ce4-4195-9937-5328ef05872b"
        dataId = "seataServer.properties"
      }
    }
```


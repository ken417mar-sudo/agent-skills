# Skill: deep_clean.system

## 基本信息

-   名称: 深度系统清理
-   Skill ID: deep_clean.system
-   类别: 系统维护 / 性能优化
-   风险等级: 中
-   权限等级: 系统级（需用户授权）
-   是否支持自动化: 是
-   是否支持回滚: 是（部分）

------------------------------------------------------------------------

## 功能描述

对 Windows / Linux 系统进行多层级深度清理，包括：

-   系统垃圾文件
-   应用缓存
-   临时目录
-   残留安装包
-   日志文件
-   无效注册表项（Windows）
-   无用服务配置（可选）
-   回收站

并生成清理报告，支持"安全模式"和"深度模式"。

------------------------------------------------------------------------

## 使用场景

-   电脑长期使用后变慢
-   磁盘空间不足
-   系统异常排查前
-   新机初始化
-   游戏 / 重度工作前优化

------------------------------------------------------------------------

## 输入参数（Input Schema）

``` json
{
  "mode": {
    "type": "string",
    "enum": ["safe", "deep"],
    "default": "safe",
    "description": "清理模式：safe=安全清理，deep=深度清理"
  },
  "target": {
    "type": "array",
    "items": {
      "type": "string",
      "enum": [
        "system_cache",
        "app_cache",
        "temp_files",
        "log_files",
        "recycle_bin",
        "install_residue",
        "registry",
        "services"
      ]
    },
    "default": [
      "system_cache",
      "app_cache",
      "temp_files",
      "log_files",
      "recycle_bin",
      "install_residue"
    ]
  },
  "dry_run": {
    "type": "boolean",
    "default": false,
    "description": "是否只扫描不执行"
  },
  "backup_registry": {
    "type": "boolean",
    "default": true,
    "description": "清理注册表前是否备份"
  }
}
```

------------------------------------------------------------------------

## 输出结果（Output Schema）

``` json
{
  "status": "success | partial | failed",
  "freed_space_mb": 1532,
  "deleted_files_count": 12493,
  "affected_modules": [
    "system_cache",
    "app_cache",
    "registry"
  ],
  "duration_ms": 8321,
  "warnings": [
    "部分应用缓存正在被占用，未完全清理"
  ],
  "rollback_point_id": "rcp_20260113_001"
}
```

------------------------------------------------------------------------

## 执行流程（Execution Flow）

1.  系统环境检测\
2.  风险评估\
3.  可清理项扫描\
4.  执行清理\
5.  结果验证\
6.  生成报告 & 回滚点

执行顺序：

1.  temp_files\
2.  app_cache\
3.  system_cache\
4.  log_files\
5.  install_residue\
6.  recycle_bin\
7.  registry（仅 deep）

------------------------------------------------------------------------

## Agent 调用示例

``` json
{
  "skill_id": "deep_clean.system",
  "arguments": {
    "mode": "deep",
    "target": [
      "system_cache",
      "app_cache",
      "temp_files",
      "log_files",
      "recycle_bin",
      "install_residue",
      "registry"
    ],
    "backup_registry": true
  }
}
```

------------------------------------------------------------------------

## 权限声明（Permissions）

  权限             说明
  ---------------- --------------
  FILE_SYSTEM_RW   读写系统目录
  REGISTRY_RW      修改注册表
  SERVICE_QUERY    查询系统服务
  DISK_INFO        获取磁盘状态

------------------------------------------------------------------------

## 风险控制策略

-   注册表操作默认开启备份
-   系统目录白名单
-   黑名单路径保护：
    -   /Windows/System32
    -   /boot
    -   /usr/bin
-   清理前创建回滚点
-   高危操作需二次确认

------------------------------------------------------------------------

## 回滚机制

支持：

-   注册表恢复
-   部分系统缓存恢复
-   服务配置恢复

不支持：

-   用户主动删除的个人文件

------------------------------------------------------------------------

## UI 建议呈现

-   当前模块
-   已释放空间
-   预计剩余时间
-   当前调用 Skill

完成后：

-   空间释放统计
-   风险提示
-   建议下次清理时间
-   保存为任务模板

------------------------------------------------------------------------

## 版本信息

-   Version: 1.0.0
-   Author: PC Agent System
-   Updated: 2026-01-13

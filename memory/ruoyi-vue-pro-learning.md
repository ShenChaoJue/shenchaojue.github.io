# 若依(RuoYi-Vue-Pro) 框架学习笔记

**学习日期**: 2026-03-11
**来源**: GitHub https://github.com/YunaiV/ruoyi-vue-pro

## ⭐ Stars: 35,724 | Forks: 7,718

---

## 🏗️ 核心架构

### 模块化 Starter 模式
```
yudao-framework/
├── yudao-common                           # 通用工具
├── yudao-spring-boot-starter-security    # 权限认证
├── yudao-spring-boot-starter-tenant      # 多租户
├── yudao-spring-boot-starter-biz-data-permission  # 数据权限
├── yudao-spring-boot-starter-redis       # 缓存
├── yudao-spring-boot-starter-mybatis     # 数据库
└── yudao-spring-boot-starter-web         # Web层
```

### 多模块结构
```
yudao-module-system    # 系统模块(用户/角色/菜单)
yudao-module-bpm       # 工作流模块
yudao-module-pay       # 支付模块
yudao-module-member   # 会员模块
yudao-module-mall     # 商城模块
yudao-module-infra    # 基础设施
```

---

## 🔑 核心技术设计

### 1. 多租户 (Tenant)
- **实现**: TenantDatabaseInterceptor + MyBatis Plus TenantLineHandler
- **原理**: SQL 拦截，自动拼接 tenant_id
- **特点**: 
  - 支持忽略表配置 (@TenantIgnore)
  - 支持租户上下文 TenantContextHolder

### 2. 数据权限 (Data Permission)
- **实现**: DataPermissionRule 接口 + SQL 重写
- **原理**: 基于部门/角色过滤数据
- **特点**: 策略模式，易于扩展

### 3. 认证授权 (Security)
- **实现**: TokenAuthenticationFilter + Spring Security
- **原理**: Token 验证 → 构建 LoginUser → 放入 Security 上下文
- **特点**: 支持多用户类型

### 4. 全局异常处理
- **实现**: GlobalExceptionHandler
- **原理**: @RestControllerAdvice 统一处理
- **特点**: 统一返回 CommonResult

---

## 💡 设计模式学习

1. **Starter 模式** - 功能模块化，按需引入
2. **AOP 切面** - 无侵入增强
3. **责任链** - Filter 链处理请求
4. **策略模式** - DataPermissionRule 接口
5. **装饰器** - 拦截器增强

---

## 🛠️ 技术栈

- Spring Boot 2.7 / 3.2
- MyBatis Plus
- Spring Security
- Flowable (工作流)
- Redis + Redisson
- Vue3 + element-plus

---

## 📝 心得

这是一个非常适合学习的企业级 Java 全栈项目：
- 代码规范，注释详细
- 架构清晰，分层合理
- 文档完善，有视频教程
- 国产开源，文档中文友好

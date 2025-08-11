# Matrix Chat Server & Element Web Client

一个完整的Matrix聊天服务器和Web客户端部署项目。

## 项目架构

```
test1/
├── element-web/         # Element Web客户端
├── matrix-server/       # Matrix Synapse服务器
└── docker-compose配置文件
```

## 快速部署

### 1. 启动Matrix服务器
```bash
cd matrix-server
docker-compose -f docker-compose-simple.yml up -d
```

### 2. 启动Element Web客户端
```bash
cd element-web
docker-compose up -d
```

### 3. 访问地址
- **Element Web客户端**: http://localhost:8088
- **Matrix服务器API**: http://localhost:8008

## 功能特性

- ✅ 完整的Matrix聊天服务器
- ✅ Element Web前端界面
- ✅ 用户注册和认证
- ✅ 端到端加密聊天
- ✅ 房间创建和管理
- ✅ 文件分享
- ✅ 语音/视频通话支持

## 配置说明

### Matrix服务器配置
- 服务器名: `localhost`
- 端口: 8008
- 数据库: SQLite (开发用)
- 注册: 已启用，无需邮箱验证

### Element Web配置
- 端口: 8088
- 默认服务器: `http://localhost:8008`

## 测试账号

项目包含以下预设测试账号：
1. `test1:localhost` / 密码: `test123`
2. `test2:localhost` / 密码: `test123`
3. `test3:localhost` / 密码: `test123`
4. `test4:localhost` / 密码: `test123`
5. `admin:localhost` / 密码: `admin123` (管理员)

## 安全注意事项

⚠️ **重要**: 
- 此配置仅用于开发和测试环境
- 生产环境请更换所有默认密码和密钥
- 数据库文件和配置密钥已在.gitignore中排除

## 开发指南

### 目录结构
```
element-web/
├── config.json          # 客户端配置
├── docker-compose.yml   # Docker配置
└── src/                 # 源代码

matrix-server/
├── docker-compose-simple.yml  # 简化部署配置
├── docker-compose.yml        # 完整部署配置
└── homeserver.yaml          # 服务器配置模板
```

### 常用命令
```bash
# 查看容器状态
docker ps

# 查看服务日志
docker logs matrix-synapse
docker logs element-web

# 重启服务
docker-compose restart

# 停止所有服务
docker-compose down
```

## 故障排除

1. **端口被占用**: 修改docker-compose.yml中的端口映射
2. **服务无法连接**: 检查防火墙设置
3. **注册失败**: 确认homeserver.yaml中enable_registration: true

## 技术栈

- **后端**: Matrix Synapse (Python)
- **前端**: Element Web (React)
- **容器化**: Docker & Docker Compose
- **数据库**: SQLite/PostgreSQL

## License

MIT License
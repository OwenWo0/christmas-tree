# 生产环境部署指南

## 部署概述

本项目使用 Docker + Nginx 部署，支持 HTTPS 访问（443端口）。

## 部署前准备

### 1. 创建SSL证书目录

在项目根目录创建 `ssl` 目录，用于存放SSL证书：

```bash
mkdir ssl
```

### 2. 放置SSL证书文件

将你的SSL证书文件放置到 `ssl` 目录中：

```bash
ssl/
├── cert.pem         # SSL证书文件
└── key.pem          # SSL私钥文件
```

**注意：** 证书文件名必须为 `cert.pem` 和 `key.pem`，如果你的证书文件名不同，请重命名或修改 `nginx.conf` 中的配置。

### 3. 配置域名

编辑 `nginx.conf` 文件，将 `your-domain.com` 替换为你的实际域名：

```bash
# 使用文本编辑器修改
vim nginx.conf

# 或使用sed命令批量替换
sed -i 's/your-domain.com/你的实际域名/g' nginx.conf
```

### 4. 创建日志目录（可选）

```bash
mkdir -p logs/nginx
```

## 部署步骤

### 1. 确保已构建生产版本

```bash
npm run build
```

构建完成后，会在 `dist` 目录生成静态文件。

### 2. 启动Docker容器

```bash
docker-compose up -d
```

### 3. 查看容器状态

```bash
docker-compose ps
```

### 4. 查看容器日志

```bash
docker-compose logs -f nginx
```

## 常用管理命令

### 重启服务

```bash
docker-compose restart
```

### 停止服务

```bash
docker-compose down
```

### 更新代码后重新部署

```bash
# 1. 重新构建
npm run build

# 2. 重启容器
docker-compose restart
```

### 查看nginx配置是否正确

```bash
docker-compose exec nginx nginx -t
```

### 重新加载nginx配置

```bash
docker-compose exec nginx nginx -s reload
```

## 验证部署

### 1. 检查HTTP自动跳转HTTPS

访问：`http://你的域名`

应该自动跳转到：`https://你的域名`

### 2. 检查HTTPS证书

在浏览器中访问 `https://你的域名`，点击地址栏的锁图标，确认证书有效。

### 3. 检查服务健康状态

```bash
docker-compose ps
```

确保容器状态为 `healthy` 或 `Up`。

## 故障排查

### 1. 证书问题

如果遇到SSL证书错误：

- 检查证书文件是否正确放置在 `ssl/` 目录
- 检查证书文件权限：`chmod 644 ssl/cert.pem ssl/key.pem`
- 检查证书是否过期
- 检查证书域名是否匹配

### 2. 容器无法启动

```bash
# 查看详细日志
docker-compose logs nginx

# 检查端口是否被占用
sudo lsof -i :443
sudo lsof -i :80
```

### 3. 访问404错误

- 确认 `dist` 目录存在且包含构建文件
- 检查 `index.html` 是否存在于 `dist` 目录

### 4. 配置修改不生效

```bash
# 重新加载nginx配置
docker-compose exec nginx nginx -s reload

# 如果还不生效，重启容器
docker-compose restart
```

## 安全建议

1. **定期更新SSL证书**：在证书过期前及时更新
2. **保护私钥文件**：确保 `key.pem` 文件权限为 600
3. **定期查看日志**：监控异常访问和错误
4. **启用防火墙**：只开放必要的端口（80, 443）
5. **定期更新Docker镜像**：保持nginx版本最新

## 配置说明

### nginx.conf 主要配置

- **SSL/TLS配置**：使用TLS 1.2和1.3协议
- **Gzip压缩**：优化传输性能
- **静态资源缓存**：缓存1年，提升加载速度
- **SPA路由支持**：所有路由都返回index.html
- **安全头部**：HSTS、XSS保护等

### docker-compose.yml 配置

- **端口映射**：443（HTTPS）和 80（HTTP）
- **健康检查**：每30秒检查一次服务状态
- **自动重启**：容器异常退出后自动重启
- **只读挂载**：静态文件和配置以只读方式挂载，提升安全性

## 性能优化建议

1. **启用CDN**：将静态资源托管到CDN
2. **开启HTTP/2**：nginx配置中已启用
3. **Gzip压缩**：nginx配置中已启用
4. **浏览器缓存**：静态资源缓存1年
5. **定期清理日志**：防止日志文件过大

## 联系支持

如有问题，请查看：
- Nginx官方文档：https://nginx.org/en/docs/
- Docker官方文档：https://docs.docker.com/

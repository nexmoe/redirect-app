# 轻量级域名重定向服务开源发布

各位小伙伴们好！今天给大家安利一个我刚刚撸完的域名重定向服务工具。起因是手头闲置了不少域名，想用来做跳转服务。之前用 Caddy 虽然不错，但配置略显繁琐，功能也不够趁手，索性自己用 Golang 撸了个轻量级方案！

✨ 项目亮点：

- 🚀 轻量化设计：Docker 镜像仅 4.2MiB，内存占用约 1.277 MiB
- ⚡ 即开即用：支持 Docker 一键部署，配置简单直观
- 🔗 智能跳转：支持多目标轮询、来源追踪、路径保留等实用功能
- 📊 数据分析：自带来源参数，轻松对接 Google Analytics 统计流量

现已开源并发布到 Docker Hub，欢迎各位大佬在 GitHub 点个 Star 支持一下！你的星星就是我爆肝的动力～

## 配置说明

服务通过环境变量进行配置，支持多个域名映射规则。每个映射规则使用 `DOMAIN_MAPPING_*` 格式的环境变量进行配置。

### 环境变量格式

```
DOMAIN_MAPPING_<任意名称>=<域名>-><目标地址1>,<目标地址2>,...
```

例如：

```
DOMAIN_MAPPING_1=example.com->https://target1.com,https://target2.com
```

### 其他环境变量

- `PORT`: 服务监听端口，默认为 8080
- `PRESERVE_PATH`: 是否保持原始路径，默认为 false。设置为 "true" 时，重定向时会保留原始请求路径。
- `INCLUDE_REFERRAL`: 是否在重定向时携带来源域名信息，默认为 false。设置为 "true" 时，重定向时会添加 ref 参数，值为来源域名。
- `ENABLE_TIMESTAMP`: 是否启用时间戳参数，默认为 false。设置为 "true" 时，重定向时会添加 _t 参数。

## 部署方式

### Docker 部署

1. 拉取镜像：

```bash
docker pull nexmoe/domain-redirect
```

2. 构建镜像（可选）：

```bash
docker build -t domain-redirect .
```

3. 运行容器：

```bash
docker run -d \
  -p 8080:8080 \
  -e DOMAIN_MAPPING_1=example.com->https://target1.com,https://target2.com \
  nexmoe/domain-redirect
```

### Docker Compose 部署

1. 创建 `docker-compose.yml` 文件：

```yaml
version: '3'
services:
  domain-redirect:
    image: nexmoe/domain-redirect
    container_name: domain-redirect
    ports:
      - "8080:8080"
    environment:
      - DOMAIN_MAPPING_1=example.com->https://target1.com,https://target2.com
    restart: unless-stopped
```

2. 启动服务：

```bash
docker-compose up -d
```

3. 停止服务：

```bash
docker-compose down
```

### 直接运行

1. 编译：

```bash
go build -o domain-redirect main.go
```

2. 运行：

```bash
export DOMAIN_MAPPING_1=example.com->https://target1.com,https://target2.com
./domain-redirect
```

## 使用示例

假设配置了以下映射：

```
DOMAIN_MAPPING_1=example.com->https://target1.com,https://target2.com
```

当访问 `http://example.com/any/path` 时，服务会：

1. 在目标地址之间轮询选择
2. 将请求重定向到选中的目标地址
3. 保持原始路径（如果配置了 PRESERVE_PATH 为 true）
4. 添加时间戳参数防止缓存（如果配置了 ENABLE_TIMESTAMP 为 true）
5. 添加来源域名信息（如果配置了 INCLUDE_REFERRAL 为 true）

## 注意事项

- 确保配置的域名映射规则格式正确
- 目标地址必须是有效的 URL
- 服务默认监听 8080 端口，可以通过 PORT 环境变量修改

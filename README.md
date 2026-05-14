# New API 使用说明

这个目录现在改成以 [new-api](./new-api/README.md) 为主。

前面为了 `LiteLLM` 单独搭的 FastAPI / Proxy 骨架已经删除，当前你应该只关注：

- [new-api](./new-api/README.md)：主项目
- 本文件：中文使用说明

## 这套项目现在能做什么

`new-api` 本身就包含前后端，可以用来做：

- 管理后台
- 用户管理
- 额度管理
- 令牌管理
- 渠道管理
- 给用户提供你自己的 OpenAI 兼容 API 地址

也就是说，你可以：

1. 在后台配置你自己购买的上游模型 `API URL` 和 `API Key`
2. 给用户充值、分配额度
3. 给用户生成平台自己的 `API Key`
4. 让用户通过你的域名和你的 key 来调用模型

## 目录说明

- [new-api](./new-api/README.md)：官方仓库代码
- [new-api/docker-compose.yml](./new-api/docker-compose.yml:1)：官方 compose 配置
- [new-api/.env.example](./new-api/.env.example:1)：官方环境变量模板

## 推荐启动方式

当前 [new-api/docker-compose.yml](/c:/Users/Admin/Desktop/中专站/new-api/docker-compose.yml:1) 已经改成更适合上线的版本，包含：

- `PostgreSQL`
- `Redis`
- `Nginx` 反向代理
- 容器健康检查
- 数据持久化
- 应用服务不再直接暴露到公网

先进入项目目录：

```powershell
Set-Location .\new-api
```

然后复制生产环境变量模板：

```powershell
Copy-Item .env.production.example .env
```

至少修改这些值：

- `FRONTEND_BASE_URL`
- `SESSION_SECRET`
- `CRYPTO_SECRET`
- `POSTGRES_PASSWORD`
- `REDIS_PASSWORD`
- `SQL_DSN`
- `REDIS_CONN_STRING`
- `TRUSTED_REDIRECT_DOMAINS`

再启动：

```powershell
docker compose up -d
```

启动后默认通过 Nginx 访问：

```text
http://localhost
```

生产环境建议把 `80` 换成反向代理网关后的 `443`，由 `Nginx`、`Caddy` 或云负载均衡负责 TLS。

## 你最常用的操作

### 1. 配置你自己的上游模型

登录后台后进入“渠道管理”，新增渠道：

- `渠道类型`：通常选 `OpenAI`
- `Base URL`：填上游地址
- `Key`：填你的上游 API Key
- `Models`：填你要开放的模型名

如果你后面还想把 `LiteLLM` 接进来，也是在这里把 `LiteLLM` 当成一个 `OpenAI` 渠道来配。

### 2. 给用户额度

你可以通过后台：

- 创建用户
- 给用户充值或分配额度
- 限制用户可用模型或分组

### 3. 给用户专属 API Key

你给用户发的是 `new-api` 平台自己的 key，不是上游厂商原始 key。

用户最终使用的是：

- `Base URL`：`http://你的域名/v1`
- `API Key`：你在 `new-api` 里给他生成的 token

## 给用户的调用示例

Python:

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://your-domain/v1",
    api_key="your-new-api-token",
)

resp = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "你好"}],
)

print(resp.choices[0].message.content)
```

## 使用建议

- 如果你只是想尽快做一个给用户用的平台，先直接用 `new-api`
- 如果你后面想增强多模型路由、fallback、统一网关能力，再把 `LiteLLM` 接到 `new-api` 后面
- 商用前认真检查 `new-api` 的许可证、支付、内容安全、日志留存和合规要求
- 当前这套生产底稿在本仓库里主要看：
  - [new-api/docker-compose.yml](/c:/Users/Admin/Desktop/中专站/new-api/docker-compose.yml:1)
  - [new-api/.env.production.example](/c:/Users/Admin/Desktop/中专站/new-api/.env.production.example:1)
  - [new-api/deploy/nginx/nginx.conf](/c:/Users/Admin/Desktop/中专站/new-api/deploy/nginx/nginx.conf:1)

## 下一步建议

你现在最合适的下一步是：

1. 先把 `new-api` 跑起来
2. 先配置一个你自己的上游渠道
3. 先创建一个测试用户和一个测试 token
4. 用 OpenAI SDK 实测一遍你的平台接口

如果你愿意，我下一步可以继续直接帮你做：

1. 给你写一份“`new-api` 最小上线配置清单”
2. 帮你把 `new-api` 的 `.env` 和 `docker-compose` 按你的机器改好
3. 帮你设计“用户购买额度 -> 发 token -> 调用模型”的完整流程

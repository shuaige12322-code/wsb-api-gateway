

## 这套项目现在能做什么

`new-api` 本身就包含前后端，可以用来做：

1. 在后台配置你自己购买的上游模型 `API URL` 和 `API Key`
2. 给用户充值、分配额度
3. 给用户生成平台自己的 `API Key`
4. 让用户通过你的域名和你的 key 来调用模型


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


## Linux 上怎么配

下面按“已经买好一台云服务器”的最短路径来：

### 1. 服务器准备

至少准备好这些：

- 一台 Linux 云服务器
- 一个公网 IP
- 一个域名（可选，但正式用强烈建议有）
- 已安装 `Docker` 和 `Docker Compose`

如果你暂时还没有域名，也能先用：

```text
http://你的服务器IP
```

比如你的机器是：

```text
http://70.189.44.6:20001
```

### 2. 拉代码

```bash
git clone <你的仓库地址>
cd new-api
```

如果你现在这份仓库已经在服务器上了，就直接进入：

```bash
cd /你的项目目录/new-api
```

### 3. 复制生产环境配置

```bash
cp .env.production.example .env
```

至少改这些配置：

- `FRONTEND_BASE_URL`
- `SESSION_SECRET`
- `CRYPTO_SECRET`
- `POSTGRES_PASSWORD`
- `REDIS_PASSWORD`
- `SQL_DSN`
- `REDIS_CONN_STRING`
- `TRUSTED_REDIRECT_DOMAINS`

一个最常见的思路是：

- `FRONTEND_BASE_URL` 填你的域名，或者测试时先填 `http://70.189.44.6:20001`
- `SESSION_SECRET` 随机长字符串
- `CRYPTO_SECRET` 随机长字符串
- `POSTGRES_PASSWORD` 自己的数据库密码
- `REDIS_PASSWORD` 自己的 Redis 密码
- `TRUSTED_REDIRECT_DOMAINS` 填你的域名，测试时可加服务器 IP

### 4. 启动服务

```bash
docker compose up -d
```

查看状态：

```bash
docker compose ps
```

查看日志：

```bash
docker compose logs -f
```

### 5. 防火墙和安全组

如果你部署后打不开，优先检查：

- 阿里云安全组是否放行了端口
- Linux 防火墙是否放行了端口
- `docker compose` 映射端口是否正确

如果你当前就是通过 `70.189.44.6:20001` 访问，那至少要确保：

- 阿里云安全组放行 `20001`
- 系统防火墙放行 `20001`

### 6. 正式环境建议

正式用建议改成：

- 域名访问
- `443` HTTPS
- 反向代理
- 有证书

也就是说，最好最终让用户访问：

```text
https://你的域名
```

而不是长期直接暴露一个裸 IP 加端口。

## 部署后你怎么管理额度和兑换码

部署完成后，管理员最常用的就是这几块：

### 1. 给用户直接加额度

可以通过后台用户管理操作，也可以通过接口：

- 管理接口：`POST /api/user/manage`
- 用途：给指定用户增加、减少、覆盖额度

### 2. 创建兑换码

可以通过后台兑换码管理页面，也可以通过接口：

- 管理接口：`POST /api/redemption/`
- 用途：批量生成可兑换额度的兑换码

### 3. 用户自己兑换

用户登录后可以在钱包/充值页面输入兑换码，也可以走接口：

- 用户接口：`POST /api/user/topup`

### 4. 常见后台入口

如果你部署在：

```text
http://70.189.44.6:20001
```

可以试这些入口：

- 登录：`http://70.189.44.6:20001/sign-in`
- 新版用户管理：`http://70.189.44.6:20001/users`
- 新版钱包：`http://70.189.44.6:20001/wallet`
- 新版兑换码管理：`http://70.189.44.6:20001/redemption-codes`

如果你那边加载的是 classic 页面，再试：

- 登录：`http://70.189.44.6:20001/login`
- classic 用户管理：`http://70.189.44.6:20001/console/user`
- classic 兑换码管理：`http://70.189.44.6:20001/console/redemption`
- classic 充值页：`http://70.189.44.6:20001/console/topup`

### 5. 兑换码功能为什么可能看不到

通常是这几个原因：

- 你登录的不是管理员账号
- 还没确认支付/合规声明
- 你当前访问的是另一套前端入口

如果兑换码相关功能被锁，先去系统设置里的支付/计费相关配置，确认合规声明后再试。

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

## 下一步建议

你现在最合适的下一步是：

1. 先把 `new-api` 跑起来
2. 先配置一个你自己的上游渠道
3. 先创建一个测试用户和一个测试 token
4. 用 OpenAI SDK 实测一遍你的平台接口



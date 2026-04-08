# OpenClaw Docker 安装、访问与卸载指南（Arch Linux）

## 一、适用说明

本文档适用于在 **Arch Linux** 上使用 **Docker** 安装 OpenClaw，并从：

- 本机浏览器访问 Control UI
- 另一台电脑通过 SSH 隧道访问 Control UI

本文档同时包含：

- Docker 代理配置
- 首次 `pairing required` 的处理方法
- 日常启动 / 停止 / 重启命令
- Docker 版卸载步骤

---

## 二、安装 Docker

先确保系统已经安装 Docker，并能正常运行：

```bash
sudo pacman -Syu
sudo pacman -S docker docker-compose
sudo systemctl enable --now docker
sudo usermod -aG docker "$USER"
```

重新登录一次 shell 后检查：

```bash
docker --version
docker compose version
```

---

## 三、克隆 OpenClaw 仓库

```bash
git clone https://github.com/openclaw/openclaw.git ~/openclaw
cd ~/openclaw
```

---

## 四、如果拉镜像慢：配置 Docker 代理

如果直接拉取镜像很慢，或出现连接被重置，可给 Docker daemon 配置代理。

创建配置目录和配置文件：

```bash
sudo mkdir -p /etc/docker
sudo nano /etc/docker/daemon.json
```

写入：

```json
{
  "proxies": {
    "http-proxy": "http://127.0.0.1:7890",
    "https-proxy": "http://127.0.0.1:7890",
    "no-proxy": "127.0.0.1,localhost"
  }
}
```

然后重启 Docker：

```bash
sudo systemctl restart docker
```

测试是否能拉取官方镜像：

```bash
docker pull ghcr.io/openclaw/openclaw:latest
```

---

## 五、使用官方预构建镜像安装

建议直接使用官方预构建镜像，而不是本地 build。

在仓库目录中执行：

```bash
cd ~/openclaw
export OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"
export OPENCLAW_GATEWAY_BIND="lan"
./docker-setup.sh
```

说明：

- `OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"`：使用官方 GHCR 镜像
- `OPENCLAW_GATEWAY_BIND="lan"`：Docker 场景建议用 `lan`
- `docker-setup.sh` 会自动执行：
  - onboarding
  - 生成 gateway token
  - 写入 `.env`
  - 启动 `openclaw-gateway` 容器


## 可选：改脚本，让它“本地有镜像就跳过 pull”

把脚本里这段：

```bash
else
  echo "==> Pulling Docker image: $IMAGE_NAME"
  if ! docker pull "$IMAGE_NAME"; then
    echo "ERROR: Failed to pull image $IMAGE_NAME. Please check the image name and your access permissions." >&2
    exit 1
  fi
fi
```

改成：

```bash
else
  if docker image inspect "$IMAGE_NAME" >/dev/null 2>&1; then
    echo "==> Using existing local image: $IMAGE_NAME"
  else
    echo "==> Pulling Docker image: $IMAGE_NAME"
    if ! docker pull "$IMAGE_NAME"; then
      echo "ERROR: Failed to pull image $IMAGE_NAME. Please check the image name and your access permissions." >&2
      exit 1
    fi
  fi
fi
```

这样本地已有镜像时，就不会再 pull。


---

## 六、onboarding 推荐选项

进入向导后，可按下面选择：

### 1. Security

选：

- `Yes`

### 2. Onboarding mode

选：

- `QuickStart`

### 3. Model/auth provider

如果暂时还没有配置模型 API key，可选：

- `Skip for now`

后面再手动补。

### 4. Channels

选：

- `Skip for now`

### 5. Web search

选：

- `Skip for now`

### 6. Skills

选：

- `No`

### 7. Hooks

建议先启用：

- `boot-md`
- `session-memory`

**1）🚀 boot-md（Run BOOT.md on gateway startup）**
 意思是：**当 OpenClaw 的 gateway 启动时，自动执行工作区里的 `BOOT.md`**。官方 `BOOT.md` 模板说明里写得很直接：这个文件用来放“启动时要执行的简短明确指令”。所以您可以把它理解成“开机自启动脚本”，只是内容是给 OpenClaw 的行为指令，不是 shell 脚本。

适合的场景例如：

- 启动后自动做一次初始化检查
- 自动发一条启动通知
- 自动读取/整理某些固定上下文

如果您**没有专门写 `BOOT.md`**，那这个通常不是必开项。

------

**2）📎 bootstrap-extra-files（Inject additional workspace bootstrap files via glob/path patterns）**
 意思是：**除了默认会注入的那些工作区文件外，再额外把您指定的文件也塞进启动上下文里**。官方文档说明，OpenClaw 默认会自动引入一批 bootstrap 文件，例如 `AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md`、`BOOTSTRAP.md`；而这个 hook 的作用，就是让您再补充更多文件进去。官方也说明了 bootstrap 文件有单文件和总字符数上限，过大时会被截断。

您可以把它理解成：
 **“启动会话时，额外喂给模型看的参考资料。”**

适合的场景例如：

- 您想额外注入某个项目规范文件
- 想把自己写的补充说明、流程文档也一并载入
- 想通过 glob/path 把某类文件自动带进去

如果您目前只是普通使用，**没有额外 bootstrap 文件需求**，这个也可以先不开。

------

**3）📝 command-logger（Log all command events to a centralized audit file）**
 意思是：**把所有命令事件统一记录到一个审计日志文件里**。官方 hooks 页面和 CLI 页面都写明了，它就是用来做 command auditing 的。

您可以把它理解成：
 **“OpenClaw 的命令操作留痕器。”**

适合的场景例如：

- 您想回头查它执行过什么命令
- 想审计自动化行为
- 想排查“它刚才到底动了什么”

如果您比较重视可追踪性、排错，这个挺有用。
 缺点就是会多一些日志记录。

------

**4）💾 session-memory（Save session context to memory when /new or /reset command is issued）**
 意思是：**当您执行 `/new` 或 `/reset` 时，把当前会话上下文保存到 memory**。官方 hooks 列表和运行日志都明确写了，这个 hook 绑定的是 `command:new, command:reset`。

您可以把它理解成：
 **“在您开新会话或重置时，自动把刚才的重要上下文存进记忆。”**

这对连续工作很有帮助，尤其适合：

- 长项目
- 多轮调试
- 您不想每次 `/new` 后都重新解释背景

不过要注意一点：官方研究文档和相关 issue 也说明了，“保存到 memory”不等于“新会话一定自动完整注入这些 memory”；memory 的保存和后续召回，是两个相关但不完全相同的机制。

------

### 您现在怎么选更合适

如果您是 **普通 Linux Docker 用户，先求稳、少折腾**，我建议：

- **可开**：`command-logger`
- **可开**：`session-memory`
- **先不开也行**：`boot-md`
- **先不开也行**：`bootstrap-extra-files`

也就是更偏向这套：

- 想保留操作痕迹 → 开 `command-logger`
- 想保留会话连续性 → 开 `session-memory`
- 没写 `BOOT.md` → 不必开 `boot-md`
- 没额外 bootstrap 文件 → 不必开 `bootstrap-extra-files`

### 一句话版

- `boot-md`：启动时自动执行 `BOOT.md`
- `bootstrap-extra-files`：额外注入自定义文件到启动上下文
- `command-logger`：记录命令日志
- `session-memory`：`/new`、`/reset` 时自动存会话记忆

### 8. Systemd

Docker 安装下会提示：

- `Systemd user services are unavailable`

这是正常的，因为 Docker 版不需要本机 systemd daemon。

---

## 七、修正 Docker 网关配置并获取 Dashboard 地址

### 7.1 先说明：初次安装后容器短暂不健康，通常是正常的

Docker 版 OpenClaw 在 `onboard` 完成后，`openclaw-gateway` 容器可能会先进入 `starting`、短暂 `unhealthy`，甚至在配置未完整写入时出现重启；而 `docker compose run --rm openclaw-cli ...` 又依赖 `openclaw-gateway` 的网络命名空间，所以在 Gateway 尚未稳定前，`dashboard --no-open` 可能报错。

另外，Docker 版 OpenClaw 的配置和工作区会写到宿主机：

- `~/.openclaw/`
- `~/.openclaw/workspace/`

因此，Docker 部署时的关键配置应直接在宿主机修改，而不是优先进入容器内部修改。

### 7.2 这一步必须做：把 Docker 的 `gateway.mode` 和 `gateway.bind` 改正确

Docker 场景下，必须确保：

- `gateway.mode = "local"`
- `gateway.bind = "lan"`

这是因为 Docker 快速安装默认就是 `OPENCLAW_GATEWAY_BIND=lan`，而 `lan` 允许宿主机浏览器和宿主机 CLI 访问已映射端口；如果错误地使用 `loopback`，宿主机通过已发布端口访问时可能出现异常。

由于配置挂载在宿主机 `~/.openclaw/openclaw.json`，所以直接改宿主机配置文件即可。

执行：

```
python - <<'PY'
import json, pathlib
p = pathlib.Path.home() / ".openclaw" / "openclaw.json"
cfg = json.loads(p.read_text())
cfg.setdefault("gateway", {})
cfg["gateway"]["mode"] = "local"
cfg["gateway"]["bind"] = "lan"
p.write_text(json.dumps(cfg, indent=2, ensure_ascii=False))
print(f"updated: {p}")
PY
```

### 7.3 重建并只启动 Gateway

```
cd ~/openclaw
docker compose down
docker compose up -d openclaw-gateway
docker compose ps
docker compose logs --tail=100 openclaw-gateway
```

如果 `docker compose ps` 中看到 `openclaw-gateway` 已经不是 `Restarting`，而是 `Up ...` 或 `health: starting / healthy`，就可以继续下一步。

### 7.4 获取 Dashboard 地址

```
cd ~/openclaw
docker compose run --rm openclaw-cli dashboard --no-open
```

会输出类似：

```
Dashboard URL: http://127.0.0.1:18789/#token=xxxxxxxx
```

然后在浏览器打开：

```
http://127.0.0.1:18789/
```

如果页面要求 token，可以：

- 直接打开上面带 `#token=` 的完整链接
- 或把 token 粘贴到网页 UI 设置中

如果看到 `unauthorized` 或 `pairing required`，先重新获取 Dashboard 链接，再按下面的 pairing 处理方案操作。

------

## 八、Docker 下 pairing required

### Docker 下为什么会出现 `pairing required`

Control UI 的规则可以简单理解为：

- 本地 `127.0.0.1` 连接通常会自动批准
- 远程连接（LAN、Tailnet 等）需要显式批准
- 每个浏览器 profile 都会生成唯一设备 ID

但 Docker 场景里，即使宿主机浏览器打开的是：

```
http://127.0.0.1:18789/
```

从 **Gateway 容器** 的视角看，请求来源往往不是容器自己的 `127.0.0.1`，而是 Docker bridge 地址，例如：

```
172.17.0.1
```

因此 Gateway 会把这次连接识别为“远程设备”，从而触发：

```
pairing required
```

可以这样理解：

- **源码安装**：浏览器 → 宿主机 Gateway，通常表现为本地回环连接
- **Docker 安装**：浏览器 → 宿主机端口 → Docker NAT → 容器 Gateway，容器看到的可能是 bridge 地址

所以 Docker 环境下首次出现 `pairing required`，通常不是安装失败，而是设备信任模型的正常表现。

------

### token 和 pairing 不是一回事

这是 Docker 场景里最容易混淆的点。

####  token 的作用

Gateway token 的作用是：

- 证明客户端知道这个 Gateway 的访问口令
- 通过 WebSocket 认证层

#### pairing 的作用

pairing 的作用是：

- 批准“这台新的浏览器/设备”成为受信任设备

因此：

- **有 token，不等于不需要 pairing**
- **新设备第一次接入时，即使 token 正确，也仍然可能要求 pairing**

也就是说：

- `token` = 你知道门禁密码
- `pairing` = 管理员批准这台具体设备

------

### pairing 的三种处理方案

当 Docker 部署的 OpenClaw 出现：

```
disconnected (1008): pairing required
```

可以按以下三种方式处理。

### 方案 A：手动批准设备（官方默认方式，推荐）

这是默认方式，也是最安全的方式。

#### 查看待批准设备

```
cd ~/openclaw
docker compose run --rm openclaw-cli devices list
```

如果有待批准请求，会看到类似：

```
Pending (1)
Request: xxxxx
Device: xxxxx
```

#### 批准设备

```
cd ~/openclaw
docker compose run --rm openclaw-cli devices approve <requestId>
```

批准后刷新浏览器页面即可。

#### 批准最近一个待配对请求

```
cd ~/openclaw
docker compose run --rm openclaw-cli devices approve --latest
```

#### 优点

- 安全性高
- 每个浏览器/设备独立管理
- 后续可 revoke / rotate

### 方案 A - 进入容器操作版本

#### 1. **进入 `openclaw-gateway` 容器**

首先，我们进入 `openclaw-gateway` 容器：

```
docker exec -it openclaw-openclaw-gateway-1 /bin/sh
```

#### 2. **查看待批准的设备**

在容器内，使用 `node dist/index.js` 查看待批准的设备请求：

```
node dist/index.js devices list
```

如果有待批准的设备，会看到类似如下输出：

```
Pending (1)
Request: c3f69c5d-b047-4065-b5b5-2a2aefd4184a
Device: e578999b59d6923d4a90852f17a0790ef5c0870af7f4bcb7b8c5d1bc5f7a9a93
```

#### 3. **批准设备**

你可以手动批准设备（根据上一步的 `requestId` 和 `deviceId`）：

```
node dist/index.js devices approve c3f69c5d-b047-4065-b5b5-2a2aefd4184a
```

或者，你可以批准最近一个待配对请求：

```
node dist/index.js devices approve --latest
```

#### 4. **退出容器**

完成设备批准后，退出容器：

```
exit
```

然后刷新浏览器页面，已经批准的设备应该会显示为已配对。

------

#### 总结：

你已经可以通过以下步骤直接进入容器来操作设备批准：

1. 进入 `openclaw-gateway` 容器。
2. 使用 `node dist/index.js devices list` 查看待批准设备。
3. 使用 `node dist/index.js devices approve` 批准设备。

这样你就避免了 `openclaw-cli` 容器的问题，直接在 `openclaw-gateway` 容器内完成操作。

------

### 方案 B：信任 Docker 网络（推荐的自动化方案）

这个方案仍然保留 token 和 device 体系，只是把 Docker bridge 网络视为可信来源。

适用于 Docker 默认桥接网络，例如：

```
172.17.0.0/16
```

#### 一键执行代码

```
python - <<'PY'
import json, pathlib

p = pathlib.Path.home() / ".openclaw" / "openclaw.json"
cfg = json.loads(p.read_text())

cfg.setdefault("gateway", {})
trusted = cfg["gateway"].setdefault("trustedProxies", [])

for net in ["172.17.0.0/16"]:
    if net not in trusted:
        trusted.append(net)

p.write_text(json.dumps(cfg, indent=2, ensure_ascii=False))
print("Updated trustedProxies:", trusted)
PY
```

然后重启 Gateway：

```
cd ~/openclaw
docker compose restart
```

#### 配置示例

```
{
  "gateway": {
    "mode": "local",
    "bind": "lan",
    "port": 18789,
    "auth": {
      "mode": "token",
      "token": "your-token"
    },
    "trustedProxies": [
      "172.17.0.0/16"
    ]
  }
}
```

#### 优点

- 通常不需要每个浏览器都手动 approve
- 仍然保留 token / 设备安全体系
- 比较适合固定的 Docker 本地部署场景

------

#### 方案 C：关闭设备认证（不推荐，仅调试）

这是最省事但安全性最低的方案。
 它会关闭设备认证，仅依赖 token。

#### 一键执行代码

```
python - <<'PY'
import json, pathlib

p = pathlib.Path.home() / ".openclaw" / "openclaw.json"
cfg = json.loads(p.read_text())

cfg.setdefault("gateway", {})
cfg["gateway"].setdefault("controlUi", {})

cfg["gateway"]["controlUi"]["dangerouslyDisableDeviceAuth"] = True

p.write_text(json.dumps(cfg, indent=2, ensure_ascii=False))
print("Device auth disabled")
PY
```

然后重启：

```
cd ~/openclaw
docker compose restart
```

#### 配置示例

```
{
  "gateway": {
    "mode": "local",
    "bind": "lan",
    "controlUi": {
      "dangerouslyDisableDeviceAuth": true
    },
    "auth": {
      "mode": "token",
      "token": "your-token"
    }
  }
}
```

#### 优点

- 不会再出现 pairing
- 调试最方便

#### 缺点

- 安全性最低
- 不建议长期使用
- 不建议暴露到公网或多人环境

------

####  三种方案对比

| 方案                   | 是否需要 approve | 安全性 | 建议                |
| ---------------------- | ---------------- | ------ | ------------------- |
| 方案 A：手动批准设备   | 是               | 高     | 官方默认，推荐      |
| 方案 B：trustedProxies | 通常否           | 高     | Docker 本地部署推荐 |
| 方案 C：关闭设备认证   | 否               | 低     | 仅调试使用          |

#### 推荐选择

Docker 场景建议优先选择：

- **方案 A**：最标准、最安全
- **方案 B**：更适合你这种固定 Docker 本地访问场景

不建议长期使用方案 C。

------

## 十一、本机访问 OpenClaw

### 11.1 查看容器状态

```
cd ~/openclaw
docker compose ps
```

### 11.2 获取 Dashboard 地址

```
cd ~/openclaw
docker compose run --rm openclaw-cli dashboard --no-open
```

浏览器访问：

```
http://127.0.0.1:18789/
```

如果页面要求 token，就使用带 token 的完整链接。

------

## 十二、另一台电脑访问 OpenClaw（推荐使用 SSH 隧道）

不建议直接访问：

```
http://Linux主机IP:18789/
```

更稳妥的方式是从另一台电脑建立 SSH 隧道：

```
ssh -N -L 18789:127.0.0.1:18789 用户名@Linux主机IP
```

然后在你自己的电脑浏览器打开：

```
http://127.0.0.1:18789/
```

如果要求 token，就在 Linux 主机上执行：

```
cd ~/openclaw
docker compose run --rm openclaw-cli dashboard --no-open
```

拿到带 token 的完整链接后，在另一台电脑浏览器打开。

如果首次连接出现 `pairing required`，同样在 Linux 主机上执行：

```
cd ~/openclaw
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

------

## 十三、日常启动、关闭、重启、查看日志

### 13.1 启动

```
cd ~/openclaw
docker compose up -d openclaw-gateway
```

### 13.2 停止

```
cd ~/openclaw
docker compose stop openclaw-gateway
```

### 13.3 重启

```
cd ~/openclaw
docker compose restart openclaw-gateway
```

### 13.4 关闭并删除当前 compose 容器

```
cd ~/openclaw
docker compose down --remove-orphans
```

### 13.5 查看状态

```
cd ~/openclaw
docker compose ps
```

### 13.6 查看日志

```
cd ~/openclaw
docker compose logs -f openclaw-gateway
```

查看最近 100 行：

```
cd ~/openclaw
docker compose logs --tail=100 openclaw-gateway
```

------

## 十四、常用设备管理命令

### 查看 pending / paired 设备

```
cd ~/openclaw
docker compose run --rm openclaw-cli devices list
```

### 批准最近一个待配对请求

```
cd ~/openclaw
docker compose run --rm openclaw-cli devices approve --latest
```

### 按 requestId 批准

```
cd ~/openclaw
docker compose run --rm openclaw-cli devices approve <requestId>
```

### 调整某个设备的 scope

```
cd ~/openclaw
docker compose run --rm openclaw-cli devices rotate \
  --device <deviceId> \
  --role operator \
  --scope operator.read \
  --scope operator.write
```

### 撤销某个设备

```
cd ~/openclaw
docker compose run --rm openclaw-cli devices revoke --device <deviceId> --role operator
```

------

## 十五、常见排查命令

### 查看端口占用

```
ss -ltnp | grep 18789
sudo ss -ltnp | grep 18789
sudo lsof -nP -iTCP:18789 -sTCP:LISTEN
```

### 查看容器运行情况

```
cd ~/openclaw
docker compose ps
```

### 查看 Gateway 日志

```
cd ~/openclaw
docker compose logs --tail=200 openclaw-gateway
```

### 查看待批准设备

```
cd ~/openclaw
docker compose run --rm openclaw-cli devices list
```

### 批准设备

```
cd ~/openclaw
docker compose run --rm openclaw-cli devices approve <requestId>
```

------

## 十六、卸载 Docker 版 OpenClaw

下面分为：

- 基础卸载
- 清理配置与数据
- 可选删除镜像
- 可选删除 Docker 代理配置

### 16.1 停止并删除 OpenClaw 容器

```
cd ~/openclaw
docker compose down --remove-orphans
```

### 16.2 删除 OpenClaw 配置与工作区（可选）

如果你想彻底删掉 OpenClaw 的配置、token、workspace、agent 数据：

```
rm -rf ~/.openclaw
```

这一步会删除：

- `openclaw.json`
- gateway token
- 配对设备记录
- workspace
- sessions
- agents 配置

### 16.3 删除仓库目录（可选）

```
rm -rf ~/openclaw
```

如果您还想把 Docker 仓库里的本地环境文件也一起清掉，再补这两句：

```
cd ~/openclaw || exit 1
rm -f .env docker-compose.extra.yml
```

### 16.4 删除镜像（可选）

先查看：

```
docker images | grep -E 'openclaw|ghcr.io/openclaw/openclaw'
```

删除官方镜像：

```
docker rmi ghcr.io/openclaw/openclaw:latest
```

如果有本地 tag，也一并删除：

```
docker rmi openclaw:local
```

### 16.5 删除残留容器（可选）

```
docker ps -a | grep openclaw
```

如果有残留，删除：

```
docker rm -f <容器ID>
```

### 16.6 删除 Docker 代理配置（可选）

如果你之前为了拉镜像配置过 Docker 代理，并且现在不想保留：

```
sudo rm -f /etc/docker/daemon.json
sudo systemctl restart docker
```

如果你想保留 Docker 代理供以后继续使用，则不要删除这一步。

------

## 十七、最常用的四条命令

### 启动

```
cd ~/openclaw
docker compose up -d openclaw-gateway
```

### 查看状态

```
cd ~/openclaw
docker compose ps
```

### 获取 Dashboard 地址

```
cd ~/openclaw
docker compose run --rm openclaw-cli dashboard --no-open
```

### 停止

```
cd ~/openclaw
docker compose down --remove-orphans
```

------

## 十八、补充说明

- Docker 安装时，Gateway 运行在容器里
- 网页 UI 运行在你的浏览器里
- 宿主机浏览器访问容器映射端口时，首次可能会触发 device pairing
- 出现 `pairing required` 时，不代表安装失败，只需要批准设备即可







# OpenClaw Docker 代理配置(clash)与回退说明

## 一、当前环境概况

- OpenClaw 安装方式：Docker 版
- 项目目录：`~/openclaw`
- 代理软件：Clash
- 代理端口：`7890`
- 额外 Compose 文件：`docker-compose.extra.yml`
- 代理覆盖文件：`docker-compose.proxy.yml`

当前已经确认：

1. `openclaw-gateway` 容器中已经成功注入了代理环境变量；
2. 通过临时 `curl` 容器访问 `https://www.google.com` 已经返回 `HTTP/1.1 200 Connection established` 与 `HTTP/2 200`；
3. 这说明“容器通过宿主机代理出网”这条链路已经打通；
4. 但 Web UI 中仍出现 `LLM request rejected: ***.***.***.name: Field required`，该问题更像是模型接口兼容性或请求体校验问题，不一定是代理问题本身。

---

## 二、为什么最初容器无法走代理

最初检查结果显示：

```bash
ss -ltnp | grep -E '7890|7891' || true
```

输出为：

```text
127.0.0.1:7890
```

这表示 Clash 只监听本机回环地址，宿主机自己能访问，但 Docker 容器通常无法通过 `172.17.0.1:7890` 或 `host.docker.internal:7890` 访问它。

另外，Docker 容器默认不会自动继承宿主机的代理设置，所以需要在 Compose 配置里显式注入：

- `HTTP_PROXY`
- `HTTPS_PROXY`
- `ALL_PROXY`
- `NO_PROXY`

---

## 三、最终采用的代理方案

### 1. 只给 `openclaw-gateway` 配代理

由于 `openclaw-cli` 使用的是共享 `openclaw-gateway` 网络命名空间的方式运行，因此不能再单独给它加 `extra_hosts`，否则会报：

```text
conflicting options: custom host-to-IP mapping and the network mode
```

所以最终只给 `openclaw-gateway` 添加代理配置。

### 2. 创建代理覆盖文件

文件路径：

```text
~/openclaw/docker-compose.proxy.yml
```

文件内容如下：

```yaml
services:
  openclaw-gateway:
    extra_hosts:
      - "host.docker.internal:host-gateway"
    environment:
      HTTP_PROXY: "http://host.docker.internal:7890"
      HTTPS_PROXY: "http://host.docker.internal:7890"
      ALL_PROXY: "socks5://host.docker.internal:7890"
      NO_PROXY: "localhost,127.0.0.1,openclaw-gateway,openclaw-cli"
```

### 3. 用代码自动生成该文件

```bash
cd ~/openclaw || exit 1

cat > docker-compose.proxy.yml <<'YAML'
services:
  openclaw-gateway:
    extra_hosts:
      - "host.docker.internal:host-gateway"
    environment:
      HTTP_PROXY: "http://host.docker.internal:7890"
      HTTPS_PROXY: "http://host.docker.internal:7890"
      ALL_PROXY: "socks5://host.docker.internal:7890"
      NO_PROXY: "localhost,127.0.0.1,openclaw-gateway,openclaw-cli"
YAML

sed -n '1,120p' docker-compose.proxy.yml
```

---

## 四、启用代理的启动命令

### 1. 启动前关闭旧容器并带代理覆盖文件重启

```bash
cd ~/openclaw || exit 1

if [ -f docker-compose.extra.yml ]; then
  docker compose -f docker-compose.yml -f docker-compose.extra.yml -f docker-compose.proxy.yml down
  docker compose -f docker-compose.yml -f docker-compose.extra.yml -f docker-compose.proxy.yml up -d
else
  docker compose -f docker-compose.yml -f docker-compose.proxy.yml down
  docker compose -f docker-compose.yml -f docker-compose.proxy.yml up -d
fi
```

以后你如果想每次改完都顺手验证**api**是否可用，直接用这组最清楚：

```
cd ~/openclaw || exit 1

if [ -f docker-compose.extra.yml ]; then
  docker compose -f docker-compose.yml -f docker-compose.extra.yml -f docker-compose.proxy.yml restart openclaw-gateway
  docker compose -f docker-compose.yml -f docker-compose.extra.yml -f docker-compose.proxy.yml run --rm openclaw-cli models status --probe
else
  docker compose -f docker-compose.yml -f docker-compose.proxy.yml restart openclaw-gateway
  docker compose -f docker-compose.yml -f docker-compose.proxy.yml run --rm openclaw-cli models status --probe
fi
```

这样就不会混淆“只是重启”还是“真的探测过 API”。

### 2. 检查容器中是否已经注入代理变量

```bash
cd ~/openclaw || exit 1
cid=$(docker compose ps -q openclaw-gateway)
docker inspect "$cid" --format '{{range .Config.Env}}{{println .}}{{end}}' | grep -iE '^(HTTP_PROXY|HTTPS_PROXY|ALL_PROXY|NO_PROXY)='
```

成功时应看到类似输出：

```text
ALL_PROXY=socks5://host.docker.internal:7890
NO_PROXY=localhost,127.0.0.1,openclaw-gateway,openclaw-cli
HTTP_PROXY=http://host.docker.internal:7890
HTTPS_PROXY=http://host.docker.internal:7890
```

---

## 五、验证代理是否真的可用

### 1. 通过临时 `curl` 容器验证 HTTP 代理

```bash
docker run --rm --add-host host.docker.internal:host-gateway curlimages/curl \
  -I -x http://host.docker.internal:7890 https://www.google.com --max-time 10
```

### 2. 当前验证结果

已经成功返回：

```text
HTTP/1.1 200 Connection established
HTTP/2 200
```

这说明：

- 容器已能访问宿主机代理端口；
- 宿主机代理已能转发外部 HTTPS 请求；
- OpenClaw 的 Docker 出网代理链路已经基本正常。

---

## 六、当前遗留问题

虽然代理链路已经通了，但 OpenClaw Web UI 里依然报错：

```text
LLM request rejected: ***.***.***.name: Field required
```

这更像是：

1. 第三方 OpenAI 兼容端点与 OpenClaw 聊天请求格式不完全兼容；
2. 某个模型路由对请求体字段要求更严格；
3. 并不一定是代理本身导致的问题。

因此，当前尝试“先去掉代理再测试一次”是合理的排查步骤。

---

## 七、如何删除 / 停用代理配置

这里有两种方式。

### 方式 A：临时停用代理（推荐）

不删除文件，只是在启动时**不加载** `docker-compose.proxy.yml`。

执行：

```bash
cd ~/openclaw || exit 1

if [ -f docker-compose.extra.yml ]; then
  docker compose -f docker-compose.yml -f docker-compose.extra.yml down
  docker compose -f docker-compose.yml -f docker-compose.extra.yml up -d
else
  docker compose -f docker-compose.yml down
  docker compose -f docker-compose.yml up -d
fi
```

这样做的效果是：

- OpenClaw 仍然保留原来的 `docker-compose.proxy.yml` 文件；
- 但本次启动不会加载代理环境变量；
- 适合做“有代理 / 无代理”的来回测试。

### 方式 B：直接删除代理覆盖文件

如果你确认以后都不需要这层代理覆盖，可以删除文件：

```bash
cd ~/openclaw || exit 1
rm -f docker-compose.proxy.yml
```

然后按不带代理覆盖文件的方式重启：

```bash
cd ~/openclaw || exit 1

if [ -f docker-compose.extra.yml ]; then
  docker compose -f docker-compose.yml -f docker-compose.extra.yml down
  docker compose -f docker-compose.yml -f docker-compose.extra.yml up -d
else
  docker compose -f docker-compose.yml down
  docker compose -f docker-compose.yml up -d
fi
```

---

## 八、如何确认代理已经被删除

重启完成后执行：

```bash
cd ~/openclaw || exit 1
cid=$(docker compose ps -q openclaw-gateway)
docker inspect "$cid" --format '{{range .Config.Env}}{{println .}}{{end}}' | grep -iE '^(HTTP_PROXY|HTTPS_PROXY|ALL_PROXY|NO_PROXY)=' || echo "未检测到代理变量"
```

如果输出：

```text
未检测到代理变量
```

就说明这次启动已经不再带代理。

---

## 九、建议的排查顺序

建议你按下面顺序继续测试：

1. 先保留当前文档；
2. 临时停用代理，用不带 `docker-compose.proxy.yml` 的方式重启 OpenClaw；
3. 再进入 Web UI 测试一次最简单的对话；
4. 如果仍然报 `name: Field required`，则基本可判断与代理无关，更可能是第三方端点兼容性问题；
5. 如果去掉代理后反而恢复正常，再继续分析是否是代理地区、路由或第三方风控导致。

---

## 十、一组最简“停用代理测试”命令

如果你现在只是想快速测试“去掉代理后会不会正常”，直接执行这一组即可：

```bash
cd ~/openclaw || exit 1

if [ -f docker-compose.extra.yml ]; then
  docker compose -f docker-compose.yml -f docker-compose.extra.yml down
  docker compose -f docker-compose.yml -f docker-compose.extra.yml up -d
else
  docker compose -f docker-compose.yml down
  docker compose -f docker-compose.yml up -d
fi

cid=$(docker compose ps -q openclaw-gateway)
docker inspect "$cid" --format '{{range .Config.Env}}{{println .}}{{end}}' | grep -iE '^(HTTP_PROXY|HTTPS_PROXY|ALL_PROXY|NO_PROXY)=' || echo "未检测到代理变量"
```

如果最后显示：

```text
未检测到代理变量
```

就说明你已经成功切回“无代理模式”，可以再去 Web UI 里重新测试。

---

## 十一、第三方端点对 OpenClaw 工具请求兼容不完整

经过本次排查，已经形成较清晰的结论：

1. **网络链路正常**
   - 在启用代理时，临时 `curl` 容器已经能够通过 `host.docker.internal:7890` 成功访问 `https://www.google.com`；
   - 返回了 `HTTP/1.1 200 Connection established` 与 `HTTP/2 200`，说明 Docker 容器经由宿主机 Clash 代理出网没有问题。

2. **基础鉴权正常**
   - `openclaw-cli models status --probe` 对 `openai365/gemini-2.5-pro` 返回 `ok`；
   - 说明 API key、基础模型探测与第三方端点连接本身没有问题。

3. **第三方端点的基础文本聊天也正常**
   - 直接执行：

```bash
source ~/.openclaw/.env

curl -s https://max.openai365.top/v1/chat/completions \
  -H "Authorization: Bearer $CUSTOM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemini-2.5-pro",
    "messages": [
      {"role": "user", "content": "你好"}
    ]
  }'
```

   - 已经成功返回正常回答：`你好！有什么可以帮助你的吗？`

4. **只要禁用 OpenClaw 工具，请求就能正常对话**
   - 执行：

```bash
cd ~/openclaw || exit 1

if [ -f docker-compose.extra.yml ]; then
  docker compose -f docker-compose.yml -f docker-compose.extra.yml run --rm openclaw-cli \
    config set tools.deny '["*"]' --strict-json
  docker compose -f docker-compose.yml -f docker-compose.extra.yml restart openclaw-gateway
else
  docker compose -f docker-compose.yml run --rm openclaw-cli \
    config set tools.deny '["*"]' --strict-json
  docker compose -f docker-compose.yml restart openclaw-gateway
fi
```

   - 之后 Web UI 就恢复正常对话。

### 结论

本次问题的根因更可能是：

**第三方 OpenAI 兼容端点虽然支持基础文本聊天，但对 OpenClaw 发出的“带工具信息的请求体”兼容不完整。**

因此：

- 不是单纯的代理问题；
- 也不是基础 API key 无效；
- 更像是第三方服务端对某些工具相关字段校验更严格，导致返回：

```text
LLM request rejected: ***.***.***.name: Field required
```

这说明该端点更适合作为“纯聊天模型端点”使用，而不适合完整承接 OpenClaw 的工具增强模式。

---

## 十二、如何恢复工具配置

### 方式 A：恢复工具（取消全禁用）

如果你已经执行过：

```bash
config set tools.deny '["*"]'
```

那么恢复的方法就是把 `tools.deny` 清空。

执行：

```bash
cd ~/openclaw || exit 1

if [ -f docker-compose.extra.yml ]; then
  docker compose -f docker-compose.yml -f docker-compose.extra.yml run --rm openclaw-cli \
    config set tools.deny '[]' --strict-json
  docker compose -f docker-compose.yml -f docker-compose.extra.yml restart openclaw-gateway
else
  docker compose -f docker-compose.yml run --rm openclaw-cli \
    config set tools.deny '[]' --strict-json
  docker compose -f docker-compose.yml restart openclaw-gateway
fi
```

### 方式 B：继续保留“纯聊天模式”（推荐给第三方端点）

如果你确认当前第三方端点不兼容 OpenClaw 工具请求，那么建议继续保留：

```bash
tools.deny = ["*"]
```

这样最稳定，适合把该端点作为：

- 普通聊天
- 文本问答
- 纯文本写作

而不要用来跑 OpenClaw 的完整工具链。

---

## 十三、如何恢复代理配置

### 1. 恢复“带代理启动”

如果你之前只是临时停用了代理，而 `docker-compose.proxy.yml` 文件还在，那么重新启用代理只需要重新按“带代理覆盖文件”的方式启动：

```bash
cd ~/openclaw || exit 1

if [ -f docker-compose.extra.yml ]; then
  docker compose -f docker-compose.yml -f docker-compose.extra.yml -f docker-compose.proxy.yml down
  docker compose -f docker-compose.yml -f docker-compose.extra.yml -f docker-compose.proxy.yml up -d
else
  docker compose -f docker-compose.yml -f docker-compose.proxy.yml down
  docker compose -f docker-compose.yml -f docker-compose.proxy.yml up -d
fi
```

### 2. 恢复后检查代理变量

```bash
cd ~/openclaw || exit 1
cid=$(docker compose ps -q openclaw-gateway)
docker inspect "$cid" --format '{{range .Config.Env}}{{println .}}{{end}}' | grep -iE '^(HTTP_PROXY|HTTPS_PROXY|ALL_PROXY|NO_PROXY)='
```

如果能看到：

```text
ALL_PROXY=socks5://host.docker.internal:7890
HTTP_PROXY=http://host.docker.internal:7890
HTTPS_PROXY=http://host.docker.internal:7890
NO_PROXY=...
```

就说明代理配置已经恢复。

---

## 十四、一组“全部还原”命令

如果你想把：

- 代理重新启用；
- 工具重新恢复；

可以直接执行这一组：

```bash
cd ~/openclaw || exit 1

if [ -f docker-compose.extra.yml ]; then
  docker compose -f docker-compose.yml -f docker-compose.extra.yml run --rm openclaw-cli \
    config set tools.deny '[]' --strict-json
  docker compose -f docker-compose.yml -f docker-compose.extra.yml -f docker-compose.proxy.yml down
  docker compose -f docker-compose.yml -f docker-compose.extra.yml -f docker-compose.proxy.yml up -d
else
  docker compose -f docker-compose.yml run --rm openclaw-cli \
    config set tools.deny '[]' --strict-json
  docker compose -f docker-compose.yml -f docker-compose.proxy.yml down
  docker compose -f docker-compose.yml -f docker-compose.proxy.yml up -d
fi
```

### 还原后验证

```bash
cd ~/openclaw || exit 1
cid=$(docker compose ps -q openclaw-gateway)

echo "== 代理变量 =="
docker inspect "$cid" --format '{{range .Config.Env}}{{println .}}{{end}}' | grep -iE '^(HTTP_PROXY|HTTPS_PROXY|ALL_PROXY|NO_PROXY)=' || echo "未检测到代理变量"
```

如果你恢复工具后又再次出现 `LLM request rejected: ... name: Field required`，那么就说明：

- 代理本身是正常的；
- 第三方端点的问题仍然是“工具请求兼容性”；
- 这时就应该重新把 `tools.deny` 设回 `["*"]`。





# docker 配置、api

## openclaw onboard

如果您现在**已经初始化好了**，再运行 `openclaw onboard`，**默认不会直接把东西清空**。当前文档写得很明确：如果 `~/.openclaw/openclaw.json` 已经存在，向导会先检测现有配置，然后让您选择 **Keep / Modify / Reset**；只有您明确选了 **Reset**，或者命令里带了 `--reset`，才会做重置。

更具体一点：

- **Keep**：基本保留现有配置，不会主动抹掉。
- **Modify**：进入重新配置流程，您改到哪里，它就更新哪里；常见的是重新选 Gateway 模式、认证、模型提供方、channels、workspace 默认项。
- **Reset**：才会重置现有内容。CLI 的 `--reset` 默认作用于 `config + creds + sessions`；如果再加 `--reset-scope full`，连 workspace 也会一起移除。文档还说明 reset 用的是 `trash`，不是直接 `rm`。

所以对您现在这种“已经配好了 Docker 版”的情况，**继续跑 `openclaw onboard` 最常见的结果，就是再次进入初始化向导，让您决定保留、修改还是重置**。它不是一个“日常必须运行”的命令，更像是“重新做安装向导 / 改初始设置”的入口。



# OpenClaw 更新说明（Linux Docker）

## 一、先分清两类更新

在 OpenClaw 的 Docker 使用场景里，`git pull` 和 Docker 镜像更新不是一回事。

### 1. `git pull`

用于更新您本机 `~/openclaw` 仓库里的文件，例如：

- `docker-setup.sh`
- `docker-compose.yml`
- 其他辅助脚本与配置文件

### 2. `docker compose pull` / `docker pull`

用于更新容器里的 OpenClaw 程序本体，也就是 Docker 镜像。

可以这样理解：

- **仓库文件**：宿主机侧的安装与启动逻辑
- **Docker 镜像**：容器里真正运行的 OpenClaw 程序

因此，**镜像更新了，不代表本地脚本也更新了；本地脚本更新了，也不代表容器镜像已经更新。**

------

## 二、是否需要重新 clone 仓库

通常**不需要重新 clone**。

如果您已经有 `~/openclaw` 目录，正常情况下只需要：

```
cd ~/openclaw
https_proxy=http://127.0.0.1:7890
HTTPS_PROXY=http://127.0.0.1:7890
HTTP_PROXY=http://127.0.0.1:7890
http_proxy=http://127.0.0.1:7890
git fetch --refetch origin
git pull
```

这就相当于更新本地仓库中的安装脚本和 compose 文件。

只有在以下情况，才建议重新 clone 一份新的仓库：

- 当前仓库改动很多，长期 `git pull` 冲突
- 本地文件已经比较混乱，不想继续清理
- 想保留旧仓库，同时测试一份全新环境
- 怀疑本地仓库有损坏或缺文件

------

## 三、环境变量说明

如果您当前的运行方式依赖以下设置，更新前应先确认：

```
export OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"
export OPENCLAW_GATEWAY_BIND="lan"
```

其中：

- `OPENCLAW_IMAGE`：指定使用官方镜像
- `OPENCLAW_GATEWAY_BIND="lan"`：让 gateway 绑定到局域网可访问模式

如果您平时就是这样运行，更新时也应继续保留。

------

## 四、已完成初始化后的日常更新方式

如果您已经完成过 OpenClaw 的初始化配置，后续**大多数普通更新**建议直接更新镜像，不必每次重新跑安装向导。

推荐流程：

```
cd ~/openclaw

export OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"
export OPENCLAW_GATEWAY_BIND="lan"

docker compose pull
#或者
#docker pull --progress=plain ghcr.io/openclaw/openclaw:latest
```

然后重新拉起容器时，按您当前的 compose 结构处理：

```
if [ -f docker-compose.extra.yml ]; then
  docker compose -f docker-compose.yml -f docker-compose.extra.yml -f docker-compose.proxy.yml down
  docker compose -f docker-compose.yml -f docker-compose.extra.yml -f docker-compose.proxy.yml up -d
else
  docker compose -f docker-compose.yml -f docker-compose.proxy.yml down
  docker compose -f docker-compose.yml -f docker-compose.proxy.yml up -d
fi
```

这套方式的优点是：

- 更新镜像版本
- 重建容器
- 保留原有持久化配置与数据
- 不重新进入 onboarding 向导
- 自动兼容 `docker-compose.extra.yml` 是否存在的情况

------

## 五、什么时候需要 `git pull`

只有在以下情况，才建议额外执行一次 `git pull`：

- 官方安装脚本或 compose 逻辑可能有变化
- 您需要拿到最新版 `docker-setup.sh`
- 您要修改或重新生成 Docker 相关配置
- 您改动了以下环境变量，并需要重新运行 `docker-setup.sh`：
  - `OPENCLAW_HOME_VOLUME`
  - `OPENCLAW_EXTRA_MOUNTS`
  - `OPENCLAW_DOCKER_APT_PACKAGES`
  - `OPENCLAW_EXTENSIONS`

这类情况通常是：

```
cd ~/openclaw
git pull
# 然后按需要再运行 docker-setup.sh
```

------

## 六、什么时候两个都要用

如果您既想：

- 更新 OpenClaw 容器镜像
- 又想同步官方最新脚本与 compose 配置

那就两个都用：

```
cd ~/openclaw
git pull

export OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"
export OPENCLAW_GATEWAY_BIND="lan"

docker compose pull

if [ -f docker-compose.extra.yml ]; then
  docker compose -f docker-compose.yml -f docker-compose.extra.yml -f docker-compose.proxy.yml down
  docker compose -f docker-compose.yml -f docker-compose.extra.yml -f docker-compose.proxy.yml up -d
else
  docker compose -f docker-compose.yml -f docker-compose.proxy.yml down
  docker compose -f docker-compose.yml -f docker-compose.proxy.yml up -d
fi
```

------

## 七、什么时候需要运行 `docker-setup.sh`

`docker-setup.sh` 的主要作用不是“普通更新”，而是处理安装、镜像选择、compose 生成、挂载等宿主机侧逻辑。

如果您已经配置完成，**不建议每次更新都重新运行它**，因为这样可能再次触发配置覆盖或向导流程。

更适合运行 `docker-setup.sh` 的场景是：

- 第一次安装
- 需要切换镜像来源
- 需要重新生成 compose 配置
- 修改了挂载、volume、扩展、APT 包等相关设置
- 需要重新执行官方 setup 流程

例如：

```
cd ~/openclaw
git pull

export OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"
export OPENCLAW_GATEWAY_BIND="lan"

./docker-setup.sh
```

------

## 八、关于配置被覆盖的问题

如果每次都通过官方脚本重新进入 setup 流程，可能会出现类似提示：

```
Config overwrite: /home/node/.openclaw/openclaw.json ...
```

这表示配置文件被重写，同时系统通常会生成一个 `.bak` 备份。

因此，对于**已经完成初始化**的 Docker 部署，后续更新应尽量避免反复跑向导型脚本。更稳妥的方式是直接：

```
export OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"
export OPENCLAW_GATEWAY_BIND="lan"

docker compose pull
```

然后按您的 compose 组合重新拉起容器。

如果当前配置可正常使用，建议顺手备份一份：

```
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.good
```

------

## 九、`git pull` 时遇到本地修改冲突怎么办

如果执行 `git pull` 时提示某个文件有本地修改，例如 `package.json`，说明本地文件与远程更新发生了冲突，Git 为了防止覆盖，自动中止了更新。

先查看状态：

```
cd ~/openclaw
git status
git diff -- package.json
```

### 1）本地改动不需要保留

直接丢弃：

```
cd ~/openclaw
git restore package.json
git pull
```

### 2）想先临时保存本地改动

使用 stash：

```
cd ~/openclaw
git stash push -m "before openclaw update"
git pull
```

之后如果需要恢复：

```
git stash list
git stash pop
```

### 3）本地改动本来就是有意保留的

那就先提交，再更新：

```
cd ~/openclaw
git add package.json
git commit -m "keep local package.json changes"
git pull --rebase
```

------

## 十、推荐的实际使用策略

### 1. 普通日常更新

大多数时候只需要：

```
cd ~/openclaw

export OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"
export OPENCLAW_GATEWAY_BIND="lan"

docker compose pull

if [ -f docker-compose.extra.yml ]; then
  docker compose -f docker-compose.yml -f docker-compose.extra.yml -f docker-compose.proxy.yml down
  docker compose -f docker-compose.yml -f docker-compose.extra.yml -f docker-compose.proxy.yml up -d
else
  docker compose -f docker-compose.yml -f docker-compose.proxy.yml down
  docker compose -f docker-compose.yml -f docker-compose.proxy.yml up -d
fi
```

### 2. 官方脚本或 compose 逻辑可能变化时

再执行：

```
cd ~/openclaw
git pull
```

### 3. 需要重新生成 setup 配置时

再运行：

```
./docker-setup.sh
```

------

## 十一、一句话总节

- **更新仓库脚本**：用 `git pull`
- **更新 Docker 镜像**：用 `docker compose pull`
- **普通更新**：通常只需要更新镜像后重新拉起容器
- **需要跟进官方 setup/compose 变化时**：再加 `git pull`
- **不要把每次更新都等同于重新跑安装向导**
- **当前环境下，更新时记得保留：**
  - `export OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"`
  - `export OPENCLAW_GATEWAY_BIND="lan"`
  - 您自己的 `docker-compose.proxy.yml`
  - 若存在则同时带上 `docker-compose.extra.yml`



# 配置详解

## 一、官方文档入口

最核心的两页是：

- **Configuration Reference**：完整逐字段参考，覆盖 `~/.openclaw/openclaw.json` 所有字段。
- **Configuration**：偏使用说明，讲最小配置、如何编辑、热加载、CLI 修改等。

------

## 二、和你脚本最相关的字段，官方怎么定义

### 1）`gateway`

官方示例中，`gateway` 主要包含这些字段：`mode`、`port`、`bind`、`auth`、`tailscale`、`controlUi`、`remote`、`trustedProxies`、`allowRealIpFallback`、`tools`、`push`。

你现在脚本里实际碰到的重点有：

- `gateway.mode`：`local` 或 `remote`。官方说明是，Gateway 只有在 `local` 时才真正启动；`remote` 是客户端去连接远端 Gateway。
- `gateway.port`：Gateway 的统一端口，WebSocket 和 HTTP 共用，默认 `18789`。优先级是 `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`。
- `gateway.bind`：可用值是 `auto`、`loopback`、`lan`、`tailnet`、`custom`。官方特别提醒：这里应该写**绑定模式**，不是直接写 `0.0.0.0` 或 `127.0.0.1` 这类 host alias。
- Docker 场景下，如果容器内还是默认 `loopback`，桥接网络通过 `-p 18789:18789` 访问时，Gateway 可能不可达；官方建议这时改成 `bind: "lan"`，或者 `customBindHost: "0.0.0.0"`，或者使用 host network。

这正好对应你之前脚本里把 `BIND` 设成 `lan` 的用法——这个方向和官方文档是一致的。

------

### 2）`gateway.auth`

官方说明：

- `gateway.auth.mode` 可为 `none`、`token`、`password`、`trusted-proxy`。
- 如果同时配置了 `token` 和 `password`，则必须显式写 `gateway.auth.mode`，否则启动和安装/修复流程会失败。
- `mode: "none"` 只适合可信本机 loopback 环境。
- `gateway.auth.allowTailscale` 为真时，Tailscale Serve 的身份头可以满足 Control UI / WebSocket 的认证；但 HTTP API 仍需要 token/password。
- `gateway.auth.rateLimit` 是失败认证限速；其中 `exemptLoopback` 默认真。

------

### 3）`models`

这是你脚本最值得注意的部分，因为官方文档里**确实有** `models.mode`。官方定义如下：

- `models.mode`：provider catalog 行为，取值 `merge` 或 `replace`。
- `models.providers`：自定义 provider 映射表，以 provider id 为 key。
- `models.providers.*.api`：请求适配器，例如 `openai-completions`、`openai-responses`、`anthropic-messages`、`google-generative-ai` 等。
- `models.providers.*.apiKey`：provider 凭证；官方建议优先用 SecretRef 或环境变量替换。
- `models.providers.*.auth`：认证策略，例如 `api-key`、`token`、`oauth`、`aws-sdk`。
- `models.providers.*.baseUrl`：上游 API 根地址。
- `models.providers.*.headers`：额外静态请求头，用于代理或租户路由。
- `models.providers.*.models`：显式 provider 模型目录。
- `models.providers.*.injectNumCtxForOpenAICompat`：官方特别写到，给 Ollama + `openai-completions` 兼容时，可自动注入 `options.num_ctx`，默认真。
- `models.providers.*.authHeader`：强制把凭证放到 `Authorization` header。

这里有一个关键点：
 **官方语义里，`models.mode` 本身就是配置字段。** 官方甚至明确写了：想让 config 完全重写 `models.json` 时，用 `models.mode: "replace"`。

所以如果你现在脚本是“把 `MODE` 只当脚本行为开关，但主动删除 `data["models"]["mode"]`”，那是**你脚本的设计选择**，不是官方字段语义。官方配置里，这个字段是存在且有正式含义的。

------

### 4）`agents.defaults`

官方在 `agents.defaults` 下面给了很多细项。与你脚本最相关的是：

- `agents.defaults.model`：可写成字符串 `"provider/model"`，也可写成对象 `{ primary, fallbacks }`。字符串只设主模型；对象可同时设主模型和失败回退模型。
- `agents.defaults.models`：配置模型目录与 allowlist，供 `/model` 使用；每个条目可以带 `alias` 和 `params`。
- `agents.defaults.imageModel`：图片/视觉模型路由，可写字符串或 `{ primary, fallbacks }`。
- `agents.defaults.pdfModel`：PDF 工具使用的模型路由；如未设置，会回退到 `imageModel`，再回到 provider 默认策略。
- `agents.defaults.pdfMaxBytesMb`、`pdfMaxPages`：PDF 工具默认大小与页数限制。
- `agents.defaults.maxConcurrent`：并发 agent run 上限，默认 1。
- `agents.defaults.userTimezone`：系统提示里使用的用户时区；默认回退主机时区。
- `agents.defaults.timeFormat`：时间格式，`auto | 12 | 24`。

这意味着你脚本里写的：

- `agents.defaults.model.primary`
- `agents.defaults.models`

都是**官方推荐结构的一部分**，没有问题。

------

### 5）`env`

官方对环境变量的描述很清楚：

- OpenClaw 会从多个来源读环境变量，规则是**不覆盖已有值**。
- 优先级从高到低是：
  1. 进程环境
  2. 当前工作目录 `.env`
  3. `~/.openclaw/.env`
  4. `openclaw.json` 里的 `env` 块
  5. 可选的 shellEnv 登录 shell 导入。
- `env.shellEnv.enabled` 可开启从登录 shell 导入缺失变量；`timeoutMs` 控制超时。
- 配置里的字符串支持 `${VAR_NAME}` 形式的环境变量替换；变量缺失或为空会在加载时直接报错。

所以你把 API key 放进 `~/.openclaw/.env`，再在配置里引用环境变量，这条路是**完全符合官方思路**的。

------

### 6）`auth`

官方这里指的是**认证资料存储**，和 `gateway.auth` 不是同一个层次。

- `auth.profiles`：按 profile id 存储 provider + mode + 关联元数据。
- `auth.order`：定义同一 provider 下 profile 的优先顺序。
- 每个 agent 目录还可以有 `<agentDir>/auth-profiles.json`。

你当前脚本没直接写这一块，但如果以后要兼容官方 onboard / oauth / 多账号切换，这块要注意不要和你自己手写的 `models.providers` 混为一谈。

------

## 三、你现在这类手写 provider 配置，对应官方字段关系

按官方字段语义，你这类自定义 provider 大致应该理解成：

- `models.mode`：决定是与现有目录合并，还是整体替换。
- `models.providers.<providerId>.api`：告诉 OpenClaw 用哪套协议适配。
- `models.providers.<providerId>.apiKey`：凭证来源。
- `models.providers.<providerId>.baseUrl`：你自己的转发站或 API 根地址。
- `models.providers.<providerId>.models`：这个 provider 暴露哪些模型。
- `agents.defaults.models`：把哪些模型放进默认模型目录/allowlist。
- `agents.defaults.model.primary`：当前默认主模型是谁。

------

## 四、我替你做一个结论：哪些地方官方已明确，哪些不是

### 官方已明确的

- `openclaw.json` 是 JSON5，字段都可选。
- 有完整逐字段参考页。
- `gateway.bind` 应写绑定模式，不是直接写 `0.0.0.0` 这类 host alias。
- Docker bridge 下若用 `loopback` 可能不可达，`lan` 是官方认可方案之一。
- `models.mode` 是正式配置字段，值为 `merge` / `replace`。
- `agents.defaults.model` 与 `agents.defaults.models` 都是正式字段。

### 不是官方写法，而是你脚本自己的策略

- 把 `MODE` 仅作为脚本内部行为开关，不写回 `models.mode`。这一点和官方参考并不相同。官方是承认 `models.mode` 这个配置字段的。
- 你脚本里用 `providers.clear()` 来模拟“replace providers”，这是你脚本的实现逻辑，不等于官方对 `models.mode: "replace"` 的完整内部实现说明。

------

## 五、 Docker openclaw-cli 指令查看，修改配置/onboard

在 Docker 场景里，这几条要改成 **通过 `openclaw-cli` 容器执行**，因为官方 Docker 文档就是这样用 CLI 的；而且官方说明 `openclaw-cli` 共享 `openclaw-gateway` 的网络命名空间，并且配置/工作区写在宿主机 `~/.openclaw/`。

你原来这几条：

```
openclaw config get gateway
openclaw config get models
openclaw config get agents.defaults
openclaw config get auth
```

在 Docker 下对应写成：

```
docker compose run -T --rm openclaw-cli config get gateway
docker compose run -T --rm openclaw-cli config get models
docker compose run -T --rm openclaw-cli config get agents.defaults
docker compose run -T --rm openclaw-cli config get auth
```

如果你仓库里还有 `docker-compose.extra.yml`、`docker-compose.proxy.yml`，那就要和你前面脚本一样叠加这些 compose 文件，否则有时会和实际运行环境不一致。官方文档也明确说了：如果 setup 生成了额外 compose 文件，手动运行 compose 时要把它们一起带上。

你可以直接用这段小脚本：

```
#!/usr/bin/env bash
set -Eeuo pipefail

REPO_DIR="${REPO_DIR:-$HOME/openclaw}"
cd "$REPO_DIR" || { echo "找不到目录: $REPO_DIR"; exit 1; }

dc() {
  local files=(-f docker-compose.yml)
  [ -f docker-compose.extra.yml ] && files+=(-f docker-compose.extra.yml)
  [ -f docker-compose.proxy.yml ] && files+=(-f docker-compose.proxy.yml)
  docker compose "${files[@]}" "$@"
}

dc run -T --rm openclaw-cli config get gateway
echo
dc run -T --rm openclaw-cli config get models
echo
dc run -T --rm openclaw-cli config get agents.defaults
echo
dc run -T --rm openclaw-cli config get auth
```

如果你是想“打开 onboard”，这里要分两种情况。

第一种，你是想重新跑**初始化向导**，官方 Docker 手动流程写的是：

```
docker compose run --rm openclaw-cli onboard
```

然后再启动 gateway：

```
docker compose up -d openclaw-gateway
```

这是官方文档给出的 Docker manual flow。

如果你也要兼容 extra/proxy compose 文件，我建议你用下面这个版本：

```
#!/usr/bin/env bash
set -Eeuo pipefail

REPO_DIR="${REPO_DIR:-$HOME/openclaw}"
cd "$REPO_DIR" || { echo "找不到目录: $REPO_DIR"; exit 1; }

dc() {
  local files=(-f docker-compose.yml)
  [ -f docker-compose.extra.yml ] && files+=(-f docker-compose.extra.yml)
  [ -f docker-compose.proxy.yml ] && files+=(-f docker-compose.proxy.yml)
  docker compose "${files[@]}" "$@"
}

dc run --rm openclaw-cli onboard
dc up -d openclaw-gateway
```

第二种，你其实是想“打开控制面板/dashboard”，那官方 Docker 文档给的是：

```
docker compose run --rm openclaw-cli dashboard --no-open
```

它会打印 dashboard URL；文档还写了，装完后通常可以直接打开 `http://127.0.0.1:18789/`，需要的话再把 token 粘到 Control UI 里。

你可以用这个脚本：

```
#!/usr/bin/env bash
set -Eeuo pipefail

REPO_DIR="${REPO_DIR:-$HOME/openclaw}"
cd "$REPO_DIR" || { echo "找不到目录: $REPO_DIR"; exit 1; }

dc() {
  local files=(-f docker-compose.yml)
  [ -f docker-compose.extra.yml ] && files+=(-f docker-compose.extra.yml)
  [ -f docker-compose.proxy.yml ] && files+=(-f docker-compose.proxy.yml)
  docker compose "${files[@]}" "$@"
}

dc run -T --rm openclaw-cli dashboard --no-open
```

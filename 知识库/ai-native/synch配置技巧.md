# Syncthing 同步配置完整文档

> 创建时间：2026-04-01
> 状态：✅ 已连接并同步中

---

## 一、前置条件

### 1.1 本地环境要求

| 项目 | 要求 |
|------|------|
| 操作系统 | Windows 10 Pro 10.0.19045 |
| Shell | Git Bash (Unix shell 语法) |
| Python | 需安装 `paramiko` 库（用于 SSH 自动化） |
| Syncthing | 已下载至 `D:\Users\User\okteto\syncthing-windows-amd64-v1.27.9\syncthing.exe` |
| 同步目录 | `D:\Users\User\workspace\go\src` （需确保目录存在） |
| 网络 | 可访问内网 `10.107.95.219:22001`，无需外网 |

**安装 paramiko（如果没有）：**
```bash
pip install paramiko
```

**确认 Syncthing 存在：**
```bash
ls /d/Users/User/okteto/syncthing-windows-amd64-v1.27.9/syncthing.exe
```

### 1.2 远程环境要求

| 项目 | 信息 |
|------|------|
| 操作系统 | PlatOS 1.3 (LTS), Kernel 4.18.0 |
| 主机名 | xos-ekpc9f64 |
| IP 地址 | 10.107.95.219 |
| SSH 用户 | gaoyifeng |
| SSH 密码 | sase@123 |
| Syncthing | v2.0.15，安装在 `/usr/local/bin/syncthing` |
| Syncthing Home | `/xaasos/data/gaoyifeng-syncthing/syncthing` |
| 同步目录 | `/xaasos/data/gaoyifeng-src`（约 21GB，59 个子目录） |
| 监听端口 | `tcp://0.0.0.0:22001`（**非默认 22000**） |
| Web UI | `http://127.0.0.1:8385`（仅本机可访问） |
| API Key | `f7Nh6PVWJEur52eGgWce5H3UsaStk95X` |
| 进程启动方式 | `sudo -u gaoyifeng /usr/local/bin/syncthing serve --home=... --no-browser` |

**远程 Syncthing 自 2026-03-24 持续运行，无需手动管理。**

### 1.3 网络条件

- 本地与远程通过**公司内网**连通（本地可直接访问 `10.107.95.219`）
- **无法访问外网**，因此 Syncthing 的全局发现、中继服务、NAT 穿越均不可用
- 必须使用 **TCP 直连内网 IP + 端口** 方式连接

---

## 二、设备信息

| 角色 | 设备名 | 设备 ID | 地址 |
|------|--------|---------|------|
| 本地 (Windows) | yftx0097 | `HOKJOVI-SNEZIHW-42Q3S4X-4WSSMQY-2PLSNZJ-ZHX2RJA-RJDLD3W-4VBHBA3` | dynamic (监听 :22000) |
| 远程 (Linux) | xos-ekpc9f64 | `RB5N74A-2CKNM4S-CCPLQZW-KBKTTRI-P3GKVZV-J5SHOIE-NQH5EQK-5Q5YUA2` | `tcp://10.107.95.219:22001` |

**共享文件夹：**

| 项目 | 值 |
|------|------|
| 文件夹 ID | `go-src-sync` |
| 文件夹标签 | `go-src` |
| 同步类型 | `sendreceive`（双向同步） |
| 扫描间隔 | 30 秒 |
| 文件监听 | 启用（延迟 1 秒） |

---

## 三、连接拓扑

```
┌──────────────────────────────┐          ┌──────────────────────────────┐
│  本地 Windows (yftx0097)     │          │  远程 Linux (xos-ekpc9f64)   │
│                              │          │                              │
│  Syncthing v1.27.9           │   TCP    │  Syncthing v2.0.15           │
│  Home: D:\...\syncthing-v1.27│◄────────►│  Home: /xaasos/data/...      │
│  Listen: :22000              │  直连    │  Listen: :22001              │
│  Web UI: :8384               │          │  Web UI: :8385               │
│                              │          │                              │
│  同步目录:                    │  双向    │  同步目录:                    │
│  D:\Users\User\workspace\    │◄═══════►│  /xaasos/data/               │
│    go\src                    │  同步    │    gaoyifeng-src             │
│                              │          │                              │
│  API Key:                    │  TLS1.3  │  API Key:                    │
│  wVwVdVs6HbTb...             │  加密    │  f7Nh6PVWJEur...             │
└──────────────────────────────┘          └──────────────────────────────┘
```

- 加密方式：TLS1.3-TLS_CHACHA20_POLY1305_SHA256
- 全局发现：❌ 关闭
- 本地发现：❌ 关闭
- 中继服务：❌ 关闭
- NAT 穿越：❌ 关闭

---

## 四、配置文件

### 4.1 本地 config.xml

路径：`D:\Users\User\okteto\syncthing-windows-amd64-v1.27.9\config.xml`

```xml
<configuration version="37">
    <folder id="go-src-sync" label="go-src" path="D:\Users\User\workspace\go\src"
            type="sendreceive" rescanIntervalS="30" fsWatcherEnabled="true"
            fsWatcherDelayS="1" ignorePerms="true" autoNormalize="true">
        <filesystemType>basic</filesystemType>
        <device id="HOKJOVI-SNEZIHW-42Q3S4X-4WSSMQY-2PLSNZJ-ZHX2RJA-RJDLD3W-4VBHBA3"
                introducedBy=""><encryptionPassword></encryptionPassword></device>
        <device id="RB5N74A-2CKNM4S-CCPLQZW-KBKTTRI-P3GKVZV-J5SHOIE-NQH5EQK-5Q5YUA2"
                introducedBy=""><encryptionPassword></encryptionPassword></device>
        <!-- ... 其他默认配置 ... -->
    </folder>

    <!-- 本地设备 -->
    <device id="HOKJOVI-..." name="yftx0097">
        <address>dynamic</address>
    </device>

    <!-- 远程设备 — 关键：指定直连地址和端口 -->
    <device id="RB5N74A-..." name="xos-ekpc9f64">
        <address>tcp://10.107.95.219:22001</address>
    </device>

    <gui enabled="true" tls="false">
        <address>127.0.0.1:8384</address>
        <apikey>wVwVdVs6HbTbFVn5xNt7zfa7uwXFgpRS</apikey>
    </gui>

    <options>
        <listenAddress>default</listenAddress>
        <globalAnnounceEnabled>false</globalAnnounceEnabled>
        <localAnnounceEnabled>false</localAnnounceEnabled>
        <relaysEnabled>false</relaysEnabled>
        <startBrowser>false</startBrowser>
        <natEnabled>false</natEnabled>
        <urAccepted>-1</urAccepted>
        <autoUpgradeIntervalH>0</autoUpgradeIntervalH>
        <crashReportingEnabled>false</crashReportingEnabled>
        <reconnectionIntervalS>10</reconnectionIntervalS>
    </options>
</configuration>
```

### 4.2 远程 config.xml

路径：`/xaasos/data/gaoyifeng-syncthing/syncthing/config.xml`

```xml
<configuration version="51">
    <folder id="go-src-sync" label="go-src" path="/xaasos/data/gaoyifeng-src"
            type="sendreceive" rescanIntervalS="30" fsWatcherEnabled="true"
            fsWatcherDelayS="1" caseSensitiveFS="true">
        <device id="RB5N74A-..." />  <!-- 远程自身 -->
        <device id="HOKJOVI-..." />  <!-- 本地设备 -->
    </folder>

    <!-- 远程自身 -->
    <device id="RB5N74A-..." name="xos-ekpc9f64">
        <address>dynamic</address>
    </device>

    <!-- 本地设备 -->
    <device id="HOKJOVI-..." name="yftx0097-new">
        <address>tcp://10.107.95.219:22001</address>
    </device>

    <gui enabled="true" tls="false">
        <address>127.0.0.1:8385</address>
        <apikey>f7Nh6PVWJEur52eGgWce5H3UsaStk95X</apikey>
    </gui>

    <options>
        <listenAddress>tcp://0.0.0.0:22001</listenAddress>
        <globalAnnounceEnabled>false</globalAnnounceEnabled>
        <localAnnounceEnabled>false</localAnnounceEnabled>
        <relaysEnabled>false</relaysEnabled>
        <natEnabled>false</natEnabled>
        <urAccepted>-1</urAccepted>
        <autoUpgradeIntervalH>0</autoUpgradeIntervalH>
        <crashReportingEnabled>false</crashReportingEnabled>
    </options>
</configuration>
```

---

## 五、操作手册

### 5.1 启动本地 Syncthing

```bash
cd /d/Users/User/okteto/syncthing-windows-amd64-v1.27.9
./syncthing.exe --no-browser --home=. > syncthing.log 2>&1 &
```

### 5.2 停止本地 Syncthing

```bash
taskkill //F //IM syncthing.exe
```

### 5.3 检查本地进程

```bash
tasklist | grep -i syncthing
```

### 5.4 检查连接状态

```bash
curl -s -H "X-API-Key: wVwVdVs6HbTbFVn5xNt7zfa7uwXFgpRS" \
  http://127.0.0.1:8384/rest/system/connections | python -m json.tool
```

关键字段：`"connected": true` 表示连接正常。

### 5.5 检查同步进度

```bash
curl -s -H "X-API-Key: wVwVdVs6HbTbFVn5xNt7zfa7uwXFgpRS" \
  http://127.0.0.1:8384/rest/db/status?folder=go-src-sync | python -m json.tool
```

关键字段：
- `state`: `idle` 表示同步完成，`syncing`/`scanning` 表示进行中
- `needFiles`: 0 表示无需同步的文件
- `globalFiles` vs `localFiles`: 对比判断同步完整性

### 5.6 SSH 到远程服务器

```bash
ssh gaoyifeng@10.107.95.219
# 密码：sase@123
cd /xaasos/data/gaoyifeng-src
```

### 5.7 SSH 端口转发访问远程 Web UI

```bash
ssh -L 8385:127.0.0.1:8385 gaoyifeng@10.107.95.219
# 浏览器打开 http://127.0.0.1:8385
```

### 5.8 通过 paramiko 操作远程（脚本方式）

```python
import paramiko
ssh = paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect('10.107.95.219', username='gaoyifeng', password='sase@123', timeout=10)

# 执行命令
stdin, stdout, stderr = ssh.exec_command('你的命令')
print(stdout.read().decode())

ssh.close()
```

### 5.9 重启远程 Syncthing（通过 API）

```bash
# 在远程服务器上或通过 paramiko 执行：
curl -s -X POST -H "X-API-Key: f7Nh6PVWJEur52eGgWce5H3UsaStk95X" \
  http://127.0.0.1:8385/rest/system/restart
```

### 5.10 查看本地日志

```bash
tail -50 /d/Users/User/okteto/syncthing-windows-amd64-v1.27.9/syncthing.log
```

---

## 六、踩坑记录

### 坑 1：Windows 无 sshpass / expect，SSH 密码登录无法自动化

- **现象**：直接 `ssh gaoyifeng@10.107.95.219` 需要交互输入密码，`sshpass` 和 `expect` 均未安装
- **错误信息**：`bash: sshpass: command not found`
- **尝试**：
  - `sshpass` — 不存在
  - `expect` — 不存在
  - `ssh-copy-id` — 也需要密码，同样失败
  - `pip install sshpass` — 不是 Python 包
- **最终解决**：使用 Python `paramiko` 库，它原生支持密码认证
  ```python
  import paramiko
  ssh = paramiko.SSHClient()
  ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
  ssh.connect('10.107.95.219', username='gaoyifeng', password='sase@123')
  ```

### 坑 2：内网无法连接 Syncthing 外网服务（Relay/全局发现/STUN）

- **现象**：启动后日志反复报错，每隔几秒重试一次：
  ```
  listenerSupervisor@dynamic+https://relays.syncthing.net/endpoint:
  dial tcp 94.130.164.126:443: connectex: No connection could be made because
  the target machine actively refused it.
  ```
- **原因**：公司内网环境，无法访问外网 `relays.syncthing.net`、`data.syncthing.net` 等
- **解决**：在 config.xml 的 `<options>` 中关闭所有外网功能：
  ```xml
  <globalAnnounceEnabled>false</globalAnnounceEnabled>
  <localAnnounceEnabled>false</localAnnounceEnabled>
  <relaysEnabled>false</relaysEnabled>
  <natEnabled>false</natEnabled>
  <autoUpgradeIntervalH>0</autoUpgradeIntervalH>
  <crashReportingEnabled>false</crashReportingEnabled>
  <urAccepted>-1</urAccepted>
  ```

### 坑 3：远程 Syncthing 监听端口不是默认的 22000

- **现象**：本地配置远程设备地址为 `tcp://10.107.95.219:22000` 时连接不上
- **原因**：远程 Syncthing 配置了 `<listenAddress>tcp://0.0.0.0:22001</listenAddress>`
- **解决**：本地配置中远程设备地址必须使用 `tcp://10.107.95.219:22001`
- **排查方法**：查看远程 config.xml 中的 `<listenAddress>` 字段

### 坑 4：远程 Syncthing Home 目录不在默认位置
- **现象**：`cat ~/.config/syncthing/config.xml` 看到的配置与实际运行的不一致
- **原因**：远程 Syncthing 使用 `--home=/xaasos/data/gaoyifeng-syncthing/syncthing` 参数启动
- **解决**：通过 `ps aux | grep syncthing` 查看进程参数确认实际 Home 目录
- **实际配置文件位置**：`/xaasos/data/gaoyifeng-syncthing/syncthing/config.xml`
- **注意**：`~/.config/syncthing/config.xml` 是旧配置，**不要以它为准**

### 坑 5：API 调用返回 CSRF Error

- **现象**：`curl http://127.0.0.1:8384/rest/system/status` 返回 `CSRF Error`
- **原因**：Syncthing REST API 默认启用 CSRF 保护
- **解决**：请求头中添加 API Key：
  ```bash
  curl -s -H "X-API-Key: wVwVdVs6HbTbFVn5xNt7zfa7uwXFgpRS" \
    http://127.0.0.1:8384/rest/system/status
  ```
- **API Key 来源**：config.xml 中 `<gui>` → `<apikey>` 字段

### 坑 6：本地与远程 Syncthing 版本差异（v1.27.9 vs v2.0.15）

- **现象**：本地 config version=37，远程 config version=51
- **影响**：配置文件格式有差异，不能直接互拷
- **结论**：Syncthing 协议向后兼容，**连接和同步正常**，无需担心
- **建议**：有机会升级本地到 v2.x（本地 okteto 目录下有 `syncthing-linux-amd64-v2.0.15.tar.gz`，但那是 Linux 版）

### 坑 7：远程配置中绑定的是旧的本地设备 ID

- **现象**：远程 config.xml 中 folder 和 device 部分使用的设备 ID 是 `RESZRFX-M5SAMKU-...`，与当前本地设备 ID `HOKJOVI-SNEZIHW-...` 不匹配
- **原因**：之前可能使用过另一台电脑或重新生成过 Syncthing 密钥
- **解决**：通过 paramiko 修改远程 config.xml，全局替换旧 ID 为新 ID，再通过 API 重启远程 Syncthing
  ```python
  new_config = config.replace('RESZRFX-M5SAMKU-B3JTU4B-...', 'HOKJOVI-SNEZIHW-42Q3S4X-...')
  # 写回远程文件后调用重启 API
  ```

### 坑 8：Syncthing 后台启动后 shell 提示 exit code 0

- **现象**：`./syncthing.exe ... &` 后台启动，shell 报告命令完成 exit code 0，容易误以为 Syncthing 已退出
- **原因**：`&` 将进程放入后台，shell 任务本身正常结束
- **验证方法**：用 `tasklist | grep -i syncthing` 确认进程仍在运行

### 坑 9：`--home=.` 参数让配置和数据都在 Syncthing 安装目录

- **现象**：使用 `--home=.` 后，config.xml、证书、数据库都在 `D:\Users\User\okteto\syncthing-windows-amd64-v1.27.9\` 下
- **注意**：如果不加 `--home=.`，Syncthing 会使用默认位置 `%LOCALAPPDATA%\Syncthing`，配置和数据就不在一起
- **好处**：便于管理和备份，所有配置集中在一个目录

---

## 七、同步验证记录 (2026-04-01)

### 操作时间线

| 时间 | 操作 |
|------|------|
| 11:24 | 首次启动本地 Syncthing（默认配置） |
| 11:24 | 发现 Relay 连接失败，确认内网环境 |
| 11:33 | 通过 paramiko SSH 连接远程，获取远程配置信息 |
| 11:35 | 停止本地 Syncthing，重写 config.xml |
| 11:36 | 更新远程 config.xml（替换旧设备 ID），API 重启远程 |
| 11:37 | 重新启动本地 Syncthing |
| 11:37:30 | 连接建立成功，TLS1.3 加密 |
| 11:37:31 | 开始索引交换和文件同步 |

### 连接验证

```
Remote: RB5N74A... (xos-ekpc9f64)
Connected: True
Address: 10.107.95.219:22001
Type: tcp-client
Crypto: TLS1.3-TLS_CHACHA20_POLY1305_SHA256
Client Version: v2.0.15
```

### 同步进度快照（约 11:42）

```
State: scanning
Global:  62,488 files, 16,577 dirs, 20.61 GB
Local:   21,082 files, 16,565 dirs, 4.61 GB
Need:    42,748 files, 8,047 dirs, 16.17 GB
InSync:  19,740 files, 4.44 GB
Progress: 21.5%
Errors: 0
```

### 同步方向确认

- **双向同步** (`sendreceive`)
- 远程 `/xaasos/data/gaoyifeng-src` (21GB, 59个子目录) ↔ 本地 `D:\Users\User\workspace\go\src`
- 本地新增/修改的文件会同步到远程，远程新增/修改的文件也会同步到本地
- 冲突处理：最多保留 10 个冲突副本 (`maxConflicts: 10`)

---

## 八、日常维护

### 常见状态判断

| state 值 | 含义 |
|----------|------|
| `idle` | 同步完成，空闲等待 |
| `scanning` | 正在扫描文件变更 |
| `syncing` | 正在传输文件 |
| `error` | 出错，查看 `error` 字段 |

### 重新配对流程（如更换本地电脑）

1. 在新电脑上安装 Syncthing
2. 获取新设备 ID
3. 修改远程 config.xml 中的设备 ID（通过 paramiko）
4. 在本地 config.xml 中配置远程设备地址 `tcp://10.107.95.219:22001`
5. 两端重启 Syncthing

### 推荐的 .stignore 配置

在同步目录下创建 `.stignore` 文件排除不需要的内容：
```
// 版本控制
.git

// 构建产物
**/vendor
**/node_modules
**/*.exe
**/*.o
**/*.a

// IDE 临时文件
.idea
.vscode
*.swp
*.swo

// Syncthing 临时文件
.stversions
```


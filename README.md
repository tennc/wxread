
## 项目介绍 📚

开发这个脚本的主要目的是为了在微信读书的阅读挑战赛中刷时长和保持天数。由于本人偶尔看书时未能及时签到，导致入场费打了水漂。网上找了一些，发现高赞的自动阅读需要挂虚拟器模拟或者用ADB模拟，实现一点也不优雅。因此，我决定编写一个自动化脚本。通过对官网接口的抓包和JS逆向分析实现。

该脚本具备以下功能：

- **自动刷阅读时长**：默认计入排行榜和挑战赛，时长可调节，默认为60分钟。
- **定时运行**：可部署在GitHub Action/服务器上，支持每天定时运行并推送结果到微信。
- **Cookie自动更新**：一次抓包后，脚本能自动获取并更新Cookie，支持长时间使用。
- **轻量化设计**：相比市面上的ADB调试器和自动阅读器，本脚本实现了轻量化的编写，部署服务器即可运行，无需额外环境条件。

***
## 操作步骤（v3.0） 🛠️

### 1. 抓包准备（适合本地运行）

脚本逻辑还是比较简单的，`main.py`与`push.py`代码不需要改动。在微信阅读官网 [微信读书](https://weread.qq.com/) 搜索【三体】点开阅读点击下一页进行抓包，抓到`read`接口 `https://weread.qq.com/web/book/read`，如果返回格式正常（如：

```json
{
  "succ": 1,
  "synckey": 564589834
}
```

右键复制为Bash格式（Action部署到这里即可）；然后在 [Convert](https://curlconverter.com/python/) 转化为Python脚本，复制需要的headers与cookies字段替换到`config.py`（data字段保留），运行`main.py`即可，依赖自行安装。

### 2. GitHub Action部署运行（GitHub运行）


- Fork这个仓库，在仓库 **Settings** -> 左侧列表中的 **Secrets and variables** -> **Actions**，然后在右侧的 **Repository secrets** 中添加如下值：
  - `WXREAD_CURL_BASH`：复制read接口转换为bath的数据。
  - `PUSH_METHOD`：推送方法，可以填写你想使用的推送方式（pushplus、wxpusher、telegram）。
  - `PUSHPLUS_TOKEN`/`WXPUSHER_SPT`/`TELEGRAM_BOT_TOKEN`&`TELEGRAM_CHAT_ID`: 推送key值。
  
- 在 **Variables** 部分，最下方添加变量：
  - `READ_NUM`：设定每次阅读的目标次数。


- 基本属性

|      secrets       |                       Value                       |            说明            |
|:------------------:|:-------------------------------------------------:|:--------------------------:|
| `WXREAD_CURL_BASH` |          抓到的 read 接口的 curl_bash 命令 (必填)           |      必须提供有效的指令    |
|     `READ_NUM`     |                阅读次数，每次代表 30 秒 (可选)                |    控制阅读时长，默认 60 分钟 |
|   `PUSH_METHOD`    | 推送方式，可选值为 `pushplus`、`wxpusher` 或 `telegram` (可选) |    选择推送方式，默认不推送  |

- 推送方式

|         item         |                                     varables                                      |                      说明                      |
|:--------------------:|:---------------------------------------------------------------------------------:|:--------------------------------------------:|
| **当选择 `pushplus` 时** |                                 `PUSHPLUS_TOKEN`                                  |      https://www.pushplus.plus/uc.html       |
| **当选择 `wxpusher` 时** |                                  `WXPUSHER_SPT`                                   | https://wxpusher.zjiecode.com/docs/#/?id=spt |
| **当选择 `telegram` 时** | 配置项：`TELEGRAM_BOT_TOKEN` & `TELEGRAM_CHAT_ID` <br>代理可选：`http_proxy`&`https_proxy` |                   telagram                   |






<img src="https://github.com/user-attachments/assets/69694f8a-e6be-4c3a-820a-ac79ec2a22e5" alt="微信截图_17309442135616" width="700px"/>

### 3. 服务器运行（可选）

在你的服务器上有Python运行环境即可，使用`cron`定义自动运行。（如：

```bash
0 2 * * * /www/server/pyproject_env/wxread_venv/bin/python3 /root/wxread/main.py >> /root/wxread/logs/$(date +\%y-\%m.\%d)_sout.log 2>&1
```

意思为：【在每天两点时刻使用python所在位置编译器运行某个路径下的`main.py`脚本，同时将输出按每天的日期格式输出到对应日志中】可供参考）。


***

## 视频教程
<div style="text-align: center;">
  <a href="https://www.bilibili.com/video/BV1kJ6gY3En3/" target="_blank">
    <img src="https://pic1.imgdb.cn/item/67726c5ed0e0a243d4eca4f6.png" alt="Watch the video" style="width:900px;">
  </a>
</div>


***
## Attention 📢

1. **签到次数调整**：只需签到完成挑战赛可以将`num`次数从120调整为2，每次`num`为30秒，200即100分钟。
   
2. **解决阅读时间问题**：对于issue中提出的“阅读时间没有增加”，“增加时间与刷的时间不对等”建议保留`config.py`中的【data】字段，默认阅读三体，其它书籍自行测试。

3. **GitHub Action部署/本地部署**：主要配置config.py即可，Action部署使用环境变量，本地部署修改config.py里的阅读次数、headers、cookies即可。

4. **推送问题**：猜测是GitHub action环境问题，或者pushplus对github的ip限时墙导致偶尔推送出问题。现增加wxpusher的极简推送方式。


***
## 字段解释 🔍

| 字段 | 示例值 | 解释 |
| --- | --- | --- |
| `appId` | `"wbxxxxxxxxxxxxxxxxxxxxxxxx"` | 应用的唯一标识符。 |
| `b` | `"ce032b305a9bc1ce0b0dd2a"` | 书籍或章节的唯一标识符。 |
| `c` | `"0723244023c072b030ba601"` | 内容的唯一标识符，可能是页面或具体段落。 |
| `ci` | `60` | 章节或部分的索引。 |
| `co` | `336` | 内容的具体位置或页码。 |
| `sm` | `"[插图]威慑纪元61年，执剑人在一棵巨树"` | 当前阅读的内容描述或摘要。 |
| `pr` | `65` | 页码或段落索引。 |
| `rt` | `88` | 阅读时长或阅读进度。 |
| `ts` | `1727580815581` | 时间戳，表示请求发送的具体时间（毫秒级）。 |
| `rn` | `114` | 随机数或请求编号，用于标识唯一的请求。 |
| `sg` | `"bfdf7de2fe1673546ca079e2f02b79b937901ef789ed5ae16e7b43fb9e22e724"` | 安全签名，用于验证请求的合法性和完整性。 |
| `ct` | `1727580815` | 时间戳，表示请求发送的具体时间（秒级）。 |
| `ps` | `"xxxxxxxxxxxxxxxxxxxxxxxx"` | 用户标识符或会话标识符，用于追踪用户或会话。 |
| `pc` | `"xxxxxxxxxxxxxxxxxxxxxxxx"` | 设备标识符或客户端标识符，用于标识用户的设备或客户端。 |
| `s` | `"fadcb9de"` | 校验和或哈希值，用于验证请求数据的完整性。 |




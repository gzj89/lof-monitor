 # LOF 溢价监控

基于 GitHub Actions 的 LOF 基金场内溢价率自动监控工具，定时抓取数据并通过 **Server酱（微信）** 和/或 **飞书机器人** 推送提醒，同时将溢价历史数据自动写入仓库的 `history.csv`。

---

## 功能特性

- 监控 35 只主要跨境/商品 LOF 基金的实时溢价率
- 识别套利机会（正溢价 + 可申购），并标注限购状态与限额
- 检测 EST 日期是否滞后，显示参考溢价率
- 同时支持 Server酱（微信推送）与飞书群机器人两条推送通道
- 历史溢价数据自动追加至 `history.csv`，随仓库版本化保存
- 支持本地调试模式与模拟数据测试模式

---

## 快速部署

### 1. Fork 本仓库

点击右上角 **Fork**，将仓库复制到自己的 GitHub 账户下。

### 2. 配置 Secrets

进入你 Fork 后的仓库，点击 **Settings → Secrets and variables → Actions → New repository secret**，按需添加以下密钥：

| Secret 名称 | 说明 | 是否必填 |
|---|---|---|
| `SERVERCHAN_KEY` | Server酱 SendKey，用于微信推送 | 二选一 |
| `FEISHU_APP_ID` | 飞书自建应用的 App ID | 二选一 |
| `FEISHU_APP_SECRET` | 飞书自建应用的 App Secret | 二选一 |
| `FEISHU_CHAT_ID` | 飞书推送目标群的 Chat ID | 飞书必填 |

> 两个推送通道至少配置一个，也可同时配置。若均未配置，脚本将正常运行但不会发送推送。

### 3. 启用 Actions

Fork 后 GitHub 默认禁用 Actions，进入 **Actions** 标签页，点击 **I understand my workflows, go ahead and enable them** 即可。

### 4. 触发运行

Actions 默认在每个工作日的以下时间自动运行（UTC 时间，北京时间 +8）：

| Cron 表达式 | 对应北京时间 |
|---|---|
| `30 3 * * 1-5` | 周一至周五 11:30 |
| `30 7 * * 1-5` | 周一至周五 15:30 |

也可在 Actions 页面手动点击 **Run workflow** 立即执行。

---

## 推送通道配置

### Server酱（微信）

1. 前往 [sct.ftqq.com](https://sct.ftqq.com) 登录并获取 SendKey
2. 将 SendKey 填入仓库 Secret `SERVERCHAN_KEY`

### 飞书机器人

1. 在[飞书开发者后台](https://open.feishu.cn/app)创建一个自建应用
2. 为应用开启**发送消息**权限，并将应用发布至目标群
3. 将 App ID、App Secret、群 Chat ID 分别填入对应的 Secret

---

## 本地运行

### 安装依赖

```bash
pip install requests
```

### 配置本地密钥（可选）

在项目根目录创建 `.env` 文件：

```
SERVERCHAN_KEY=SCTxxxxxxxxxxxxxxxx
FEISHU_APP_ID=cli_xxxxxxxxxxxxxxxx
FEISHU_APP_SECRET=xxxxxxxxxxxxxxxxxxxxxxxx
FEISHU_CHAT_ID=oc_xxxxxxxxxxxxxxxxxxxxxxxx
```

> ⚠️ **重要：** 务必在 `.gitignore` 中添加 `.env`，防止密钥被意外提交到仓库。

`.gitignore` 示例：
```
.env
```

### 运行模式

```bash
# 正常模式：抓取真实数据、写入 history.csv、执行推送
python lof_notify.py

# 本地调试模式：抓取真实数据，终端表格展示，不写 CSV，不推送
python lof_notify.py --local

# 测试模式：使用模拟数据，测试消息格式与推送通道，不写 CSV
python lof_notify.py --test
```

---

## 文件说明

| 文件 | 说明 |
|---|---|
| `lof_notify.py` | 核心脚本：数据抓取、溢价计算、消息构建、推送 |
| `.github/workflows/lof.yml` | GitHub Actions 工作流配置 |
| `history.csv` | 溢价历史数据（由 Actions 自动生成并更新） |
| `.env` | 本地密钥文件（**不要提交到仓库**） |

---

## 数据来源

| 来源 | 用途 |
|---|---|
| [palmmicro.com](https://palmmicro.com/woody/res/lofcn.php?sort=premium) | EST 估值与官方溢价率 |
| 新浪财经行情接口 | 实时场内价格与涨跌幅 |
| 天天基金 App API / 网页 | 申购状态与限购额度 |

---

## 常见问题

**Q：Actions 运行后没有收到推送？**
检查 Secret 名称是否与脚本中一致（区分大小写），以及推送服务本身是否正常。可在 Actions 日志中查看详细输出。

**Q：history.csv 一直有新的 commit，正常吗？**
正常。每次 Actions 运行后，溢价数据会被追加到 `history.csv` 并自动提交到仓库，这是设计行为。若在公开仓库部署，此文件内容对外可见。

**Q：EST 日期非今日，溢价率准确吗？**
EST（估算净值）若非当日数据，溢价率计算会有滞后。脚本会在推送消息中标注警告，并在有参考数据时同步显示参考溢价率，请以实际判断为准。

**Q：如何增加或删减监控的基金？**
编辑 `lof_notify.py` 中的 `FUNDS` 列表，每项格式为 `("交易所代码", "六位代码", "基金名称")`。

---

## 免责声明

本工具仅供信息参考，不构成任何投资建议。溢价套利存在市场风险，请独立判断，自负盈亏。


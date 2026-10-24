# 中国节假日与二十四节气 ICS 日历（2026–2100）

中文 | [English](README.md)

一个可直接订阅/导入的 iCalendar (.ics) 日历数据集：覆盖 **2026-01-01 至 2100-12-31**，
共 **3900 个全天事件**，全部使用**纯英文命名**（文件不含任何中文字符），
符合 RFC 5545，并针对 **Tuta Calendar** 的导入器与订阅功能做了适配
（无循环规则、确定性 UID、纯 ASCII、CRLF）。

所有日期均由程序实际计算生成，并逐日与香港天文台官方历表交叉核对，
2026 年结果与《国务院办公厅关于 2026 年部分节假日安排的通知》逐项一致。

## 内容与统计

| 类别 (CATEGORIES) | 内容 | 事件数 |
|---|---|---|
| Public Holiday | 元旦；春节（正月初一至十四，Day 1–14 逐日拆分）；清明；劳动节；端午；中秋；国庆（10.1–10.3，Day 1–3 拆分） | 1650 |
| Traditional Festival | 除夕、元宵、七夕、中元、重阳、腊八 | 450 |
| Solar Term | 二十四节气，标准英文译名（Beginning of Spring / Spring Equinox / White Dew …） | 1800 |
| **合计** | 每公历年 52 个事件（腊八在个别公历年出现 2 次、次年 0 次，总数不变） | **3900** |

单日节假日不加 “Day N” 后缀；多天假期按天拆分为独立事件，如
`Chinese New Year (Day 1)` ~ `(Day 14)`、`National Day (Day 1)` ~ `(Day 3)`。

## 文件清单

| 文件 | 说明 |
|---|---|
| `chinese-holidays-and-solar-terms-2026-2100.ics` | 总文件（3900 事件，774 KB） |
| `cn-holidays-2026-2100.ics` | 仅法定节假日（1650） |
| `cn-traditional-festivals-2026-2100.ics` | 仅传统节日（450） |
| `cn-solar-terms-2026-2100.ics` | 仅二十四节气（1800） |
| `chinese-calendar-events-2026-2100.csv` | 全部事件的逐日审计清单（date / summary / category / uid） |
| `gen_cn_calendar.py` | 生成脚本：计算所有日期、执行全部校验、输出 ICS 与 CSV |
| `hko/T2026e.txt … T2100e.txt` | 香港天文台公历-农历换算表（2026–2100，交叉验证基准） |
| `site/` | 部署包（含上述文件与落地页，见“部署”一节） |

## 日期是如何计算的（无凭记忆）

- **二十四节气**：`pyephem` 对太阳**当日分点视黄经**做二分法求根
  （黄经到达 15° 整数倍的精确时刻），换算 UTC+8 后取北京日期。
  关键细节：必须 `sun.compute(date, epoch=date)` 归算到当日分点；
  若误用 J2000 黄经，岁差偏差约 0.36°（≈9 小时）会把大量节气推后一天。
- **农历节日**：`lunardate` 库（数据源自香港天文台）。春节 14 天取正月初一至十四；
  除夕 = 次年春节前一日（无论腊月是大月小月均成立）。
- **农历 2100 年**：`lunardate` 表止于 2099、`cnlunar` 止于 2100 年中，
  因此农历 2100 年（含 2100-02-09 春节、2100-02-08 除夕）直接由 HKO 表派生。
- **交叉验证**：每个农历日期经 `lunardate` ↔ `cnlunar` ↔ HKO 三方核对；
  每个节气日期经 `pyephem` ↔ HKO 核对。

### 验证结论

- 全部 2000+ 个农历日期三方一致（含 2033 闰十一月、2061 闰三月；
  HKO 以重复月序号标记闰月）。
- 节气 1800 个日期中 1788 个 `pyephem` 与 HKO 完全一致；其余 12 个
  `pyephem` 时刻均在午夜前 10 分钟内（如 2082 年大寒 23:59:46），
  属 ΔT 长期预测的固有不确定度，**采用 HKO 官方历表日期**
  （`cnlunar` 独立计算亦与 HKO 同侧）。
- 2026 年锚点与国办通知一致：春节 2/17、除夕 2/16、清明 4/5、
  端午 6/19、中秋 9/25、国庆 10/1–10/3。
- HKO 官方 `T2069e.txt` 本身漏印 2069-12-30 一行（非节气、非任何事件日，无影响）。

## ICS 格式规范

- `CALSCALE:GREGORIAN` / `VERSION:2.0` / `PRODID` / `METHOD:PUBLISH`
- 全天事件 `DTSTART;VALUE=DATE:YYYYMMDD`，无 `TZID`、无 `VTIMEZONE`、无 `DTEND`
- **无 `RRULE` / `RDATE`**：每个日期一个独立自包含的 `VEVENT`
- `TRANSP:TRANSPARENT`（不占用忙/闲），`DTSTAMP` 固定 `20260101T000000Z`
- 确定性 UID：`chinese-new-year-day2-2026@cn-holidays`，
  重复导入同一文件时客户端按 UID 去重；
  腊八在 2041/2060/2068/2079/2098 年出现两次，UID 追加月份后缀
  （`laba-festival-2041-01@…` / `laba-festival-2041-12@…`）以保持唯一
- 纯 ASCII 内容、CRLF 换行、每行 ≤75 八位字节（RFC 5545 折行规则）
- 事件字段固定为：`UID` / `DTSTAMP` / `DTSTART;VALUE=DATE` / `SUMMARY` /
  `CATEGORIES` / `TRANSP:TRANSPARENT`

## 使用方法

**订阅（推荐）**：桌面客户端或手机 App →
日历左侧 Your Calendars 旁 `＋` → `From URL` → 粘贴 .ics 的公网 URL →
`Subscribe`。订阅日历只读.

**导入（免费版可用）**：网页版/桌面客户端 → 目标日历右侧 `⋯` →
`Import` → 选择 .ics 文件。建议导入到专用日历（Tuta 无批量删除事件功能）；
重复导入相同文件不会产生重复事件（UID 去重）。

其他客户端（Google Calendar “From URL”、Apple/Outlook 订阅等）使用同一 URL。

## 部署（获得公网订阅 URL）

订阅要求 URL 公网可达、无需登录（Tuta 服务器端抓取）。`site/` 目录即部署包，
三种方式任选：

1. **GitHub**：公开仓库网页上传 `site/` 内文件，取 raw 链接或
   `https://cdn.jsdelivr.net/gh/<user>/<repo>@main/<file>.ics`；
2. **Cloudflare Pages**：Workers & Pages → Upload assets → 拖入 `site/` →
   `https://<project>.pages.dev/<file>.ics`；
3. **自建**：Caddy `file_server` 或 nginx（`text/calendar`，HTTPS）。

详细步骤见 `site/README.md`。

## 重新生成

```bash
pip install ephem lunardate cnlunar icalendar   # cnlunar/icalendar 用于交叉验证
python gen_cn_calendar.py
```

脚本会计算全部日期、执行上文全部校验（校验失败即非零退出）、
重写 ICS 与 CSV，并打印 2026 年明细、ephem 与 HKO 的 12 处差异清单及
边界年份（2026 / 2033 / 2100）春节日期。`sxtwl` 在 Python 3.14 下无预编译
wheel，故未采用（功能已由 lunardate + cnlunar + HKO 三方覆盖）。

## 已知限制

- 不含每年国办通知的实际**放假调休与补班日**（逐年 11 月公布，2027 年起需每年追加；
  如需要可单独制作“当年放假安排”日历）。
- 不含提醒（VALARM）；Tuta 订阅源亦不同步提醒，需提醒请导入带 VALARM 的版本。
- 12 个临界节气日期以 HKO 出版历表为准（见上文），远期 ΔT 不确定度在所难免。
- 本日历标注的是**节日日本身**（法定节日天数口径），非连休安排。

## 数据来源

- [国务院办公厅关于 2026 年部分节假日安排的通知](http://www.scio.gov.cn/zdgz/jj/202511/t20251110_938367.html)
- [香港天文台 公历-农历换算表（1901–2100）](https://www.hko.gov.hk/en/gts/time/conversion.htm)
  （本仓库使用其 `T{year}e.txt` 逐年文本表）
- Python 库：`pyephem`（天文历算）、`lunardate` / `cnlunar`（农历）、
  `icalendar`（RFC 5545 解析验证）

生成物可自由使用与再分发。

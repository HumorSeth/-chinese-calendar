# 中国 / 美国节假日 ICS 日历（2026–2100）

中文 | [English](README.md)

两个可直接导入或订阅的 iCalendar 数据集，覆盖 **2026-01-01 至 2100-12-31**：
一份中国节假日，一份美国节假日。所有事件均为**单日全天事件**、**纯英文命名**
（文件不含任何非 ASCII 字节），符合 RFC 5545，并针对 **Tuta Calendar** 的导入器与
URL 订阅做了适配：无循环规则、确定性 UID、纯 ASCII、CRLF 换行、每行 ≤75 八位字节。

所有日期均由规则计算得出（没有一个是手敲的），再用另一套独立实现交叉核对。

## 日历总览

| 文件 | 事件数 | 类别（`CATEGORIES`） | 每年事件数 |
|---|---|---|---|
| `chinese-holidays-2026-2100.ics` | 3,900 | `Public Holiday` 1,650 · `Traditional Festival` 450 · `Solar Term` 1,800 | 51–53 |
| `us-holidays-2026-2100.ics` | 1,835 | `US Federal Holiday` 1,160 · `US Traditional Holiday` 675 | 23–27 |

两份文件覆盖同样的 75 个公历年（2026–2100）。多天假期按天拆分为独立事件——
`Chinese New Year (Day 1)` … `(Day 14)`、`National Day (Day 1)` … `(Day 3)`、
`Thanksgiving Day (Day 1)` … `(Day 4)`；单日节假日不加后缀。

## 中国日历内容

| 类别 | 内容 | 事件数 |
|---|---|---|
| `Public Holiday` | 元旦；春节（正月初一至十四）；清明；劳动节；端午；中秋；国庆（10.1–10.3） | 1,650 |
| `Traditional Festival` | 除夕、元宵、七夕、中元、重阳、腊八 | 450 |
| `Solar Term` | 二十四节气，标准英文译名（Beginning of Spring / Spring Equinox / White Dew …） | 1,800 |

平均每个公历年 52 个事件——腊八在个别公历年出现两次、次年一次也没有，
因此单年事件数在 51–53 之间浮动，75 年总数不变。

## 美国日历内容

| 类别 | 内容 | 事件数 |
|---|---|---|
| `US Federal Holiday` | 元旦；马丁·路德·金纪念日；总统日；阵亡将士纪念日；六月节；独立日；劳动节；哥伦布日；退伍军人节；感恩节（Day 1–4）；圣诞节——固定日期假日遇周末时另加 `(Observed)` 补休事件 | 1,160 |
| `US Traditional Holiday` | 情人节、圣帕特里克节、耶稣受难日、复活节、母亲节、父亲节、万圣节、平安夜、跨年夜 | 675 |

### 美国联邦假日日期规则

| 假日 | 规则 |
|---|---|
| 元旦 New Year's Day | 1 月 1 日 |
| 马丁·路德·金纪念日 | 1 月第 3 个星期一 |
| 总统日（华盛顿诞辰） | 2 月第 3 个星期一 |
| 阵亡将士纪念日 | 5 月最后一个星期一 |
| 六月节 Juneteenth | 6 月 19 日 |
| 独立日 Independence Day | 7 月 4 日 |
| 劳动节 Labor Day | 9 月第 1 个星期一 |
| 哥伦布日 Columbus Day | 10 月第 2 个星期一 |
| 退伍军人节 Veterans Day | 11 月 11 日 |
| 感恩节 Thanksgiving Day | 11 月第 4 个星期四（其后周五至周日为 Day 2–4） |
| 圣诞节 Christmas Day | 12 月 25 日 |

**补休规则**（5 U.S.C. § 6103）：假日落在周六则提前到前一个周五，落在周日则顺延到
后一个周一。法定日期始终保留自己的事件，补休日期另生成一条 `(Observed)` 事件，
因此两个日期在日历上都能看到。由此会有一条事件落在所属年份之外：
`New Year's Day (Observed)` 出现在 2100-12-31，对应的是 2101 年 1 月 1 日。

## 校验

**美国日历**

- 9 个规则型假日用 `dateutil.rrule`（独立的循环展开实现）对 75 年逐条重算：**0 处差异**。
- 复活节与耶稣受难日用 `dateutil.easter`（匿名格里历算法）重算：**0 处差异**；
  耶稣受难日恒为复活节减 2 天。
- 补休事件按另一套独立实现的规则计数：元旦 22、六月节 22、独立日 23、退伍军人节 21、
  圣诞节 22——完全一致，且包含上文那条跨年事件。
- 感恩节：75 年均为 4 个连续事件，且首日恒为 11 月第 4 个星期四。
- 用 `icalendar` 7.3 解析：1,835 个事件、UID 唯一、全部为全天 `DATE` 值、
  无超过 75 八位字节的行、文件以 `END:VCALENDAR` 正常结束。

2026 年锚点（美国）：马丁·路德·金日 1/19、总统日 2/16、阵亡将士纪念日 5/25、
六月节 6/19（周五）、独立日 7/4（周六）+ 补休 7/3、劳动节 9/7、哥伦布日 10/12、
退伍军人节 11/11、感恩节 11/26–11/29、圣诞节 12/25。

**中国日历**

- 2026 年锚点与国办通知逐项一致：春节 2/17、除夕 2/16、清明 4/5、端午 6/19、
  中秋 9/25、国庆 10/1–10/3。
- 2,000+ 个农历日期在 `lunardate` ↔ `cnlunar` ↔ 香港天文台历表三方一致
  （含 2033 闰十一月、2061 闰三月）。
- 节气 1,800 个日期中 1,788 个与天文台历表完全一致；其余 12 个的天文时刻落在午夜前
  10 分钟内（如 2082 年大寒 23:59:46），属 ΔT 长期预测的固有不确定度，
  故采用天文台出版日期。

## ICS 格式规范

两份文件共用：

- `CALSCALE:GREGORIAN` / `VERSION:2.0` / `PRODID`
- 全天事件 `DTSTART;VALUE=DATE:YYYYMMDD`——无 `TZID`、无 `VTIMEZONE`、无 `DTEND`
  （按 RFC 5545 §3.6.1，只有 `DATE` 值而无 `DTEND` 即为一整天事件）。
  纯日期事件与时区无关，客户端处于任何时区都不会偏移。
- **无 `RRULE` / `RDATE`**：每个日期都是独立自包含的 `VEVENT`。
- `TRANSP:TRANSPARENT`（节假日不占用忙/闲）。
- `DTSTAMP` 固定为 `20260101T000000Z`，重复生成得到字节级一致的文件。
- 确定性 UID，如 `mlk-day-2026@us-holidays`、`chinese-new-year-day2-2026@cn-holidays`；
  重复导入同一文件时客户端按 UID 去重。
- 纯 ASCII、CRLF 换行、每行 ≤75 八位字节（RFC 5545 折行规则）。
- 事件字段固定为：`UID` / `DTSTAMP` / `DTSTART;VALUE=DATE` / `SUMMARY` /
  `CATEGORIES` / `TRANSP:TRANSPARENT`。

两处刻意的差异：

- 美国事件额外带 `DESCRIPTION`，写明日期规则、当年星期几，以及补休说明。
- 中国文件声明 `METHOD:PUBLISH`；美国文件不写，避免纯数据文件被当成日程邀约。

## 使用方法

**订阅（推荐）**：桌面或手机 App → 日历列表 Your Calendars 旁 `＋` → *From URL* →
粘贴下方公网 URL → *Subscribe*。订阅日历只读、由服务端定期刷新、不同步提醒。

**导入（免费版可用）**：网页版或桌面客户端 → 目标日历右侧 `⋯` → *Import* →
选择 `.ics`。建议导入到专用日历（Tuta 没有批量删除事件的功能）；
重复导入同一文件不会产生重复事件（UID 去重）。

**订阅地址**

```
https://raw.githubusercontent.com/HumorSeth/cn-us-calendar/main/us-holidays-2026-2100.ics
https://raw.githubusercontent.com/HumorSeth/cn-us-calendar/main/chinese-holidays-2026-2100.ics

# CDN 镜像（部分地区更快）
https://cdn.jsdelivr.net/gh/HumorSeth/cn-us-calendar@main/us-holidays-2026-2100.ics
https://cdn.jsdelivr.net/gh/HumorSeth/cn-us-calendar@main/chinese-holidays-2026-2100.ics
```

其他客户端（Google Calendar “From URL”、Apple/Outlook 订阅等）使用同一地址。

## 可复现性

这里没有隐藏数据：重建两份文件所需的规则全部写在上面的表格里，补休规则与复活节
算法也已说明。美国文件由一个约 200 行的 Python 脚本遍历 2026–2100、计算每个假日、
逐日输出一条 `VEVENT`、按 75 八位字节折行生成；中国文件另需星历库（`pyephem`）
计算节气、农历库（`lunardate`、`cnlunar`）计算节日，并与香港天文台历表交叉核对。
如需把生成脚本一并提交到仓库，开个 issue 即可。

## 已知限制

- **中国日历**：标注的是节日日本身，不含每年国办通知的放假调休与补班日
  （每年 11 月公布）。
- **美国日历**：仅联邦假日，不含州级假日，也不含提前闭市、半日等安排。
- 两份文件都不含 `VALARM`；Tuta 订阅源同样不同步提醒，需要提醒请导入带 VALARM 的版本。
- 远期日期：美国规则为法定且稳定；中国节气中距午夜不足 10 分钟的 12 个日期以
  天文台出版历表为准。

## 数据来源

- 美国：[5 U.S.C. § 6103](https://uscode.house.gov/view.xhtml?req=granuleid:USC-prelim-title5-section6103)、
  [OPM 联邦假日](https://www.opm.gov/policy-data-oversight/pay-leave/federal-holidays/)、
  `python-dateutil`（RRULE / 复活节交叉核对）、`icalendar`（RFC 5545 解析验证）
- 中国：[国务院办公厅关于 2026 年部分节假日安排的通知](http://www.scio.gov.cn/zdgz/jj/202511/t20251110_938367.html)、
  [香港天文台 公历-农历换算表（1901–2100）](https://www.hko.gov.hk/en/gts/time/conversion.htm)、
  `pyephem`、`lunardate`、`cnlunar`

## 许可证

MIT，见 [LICENSE](LICENSE)。

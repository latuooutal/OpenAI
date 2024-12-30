
# 用 Notion 和 Scriptable 高效管理信用卡与花呗还款

在现代生活中，信用卡与花呗等还款管理是一项不可忽视的任务。本文将向您详细展示如何结合 **Notion** 和 **Scriptable** 工具，打造一套高效的还款管理系统。通过本文，您可以轻松掌控每一张卡片的额度与还款日，避免因逾期还款而产生额外费用。

---

## 快速推荐：WildCard 虚拟卡管理更高效

在日常财务管理中，您还需要一张便捷的虚拟信用卡来解决海外订阅问题。**WildCard** 是解决国际支付难题的绝佳选择：  
一分钟注册，轻松订阅海外服务：[https://bit.ly/bewildcard](https://bit.ly/bewildcard)  
- 使用微信或支付宝充值即可使用。  
- 支持 ChatGPT、Google Play、Apple Store 等多种平台。  
- 使用邀请码：**ACCPAY**，立享 0 手续费，减免开卡费用！  

---

## 系统概述

本方案主要包含以下步骤：
1. 在 **Notion** 中建立信用卡信息表格，自动计算还款日期与提醒天数。
2. 使用 **Scriptable** 工具在 iOS 设备中创建屏幕小组件，实时展示即将到期的账单信息。
3. 结合快捷指令，实现还款日自动提醒功能。

---

## 系统效果预览

- **Notion**：管理信用卡额度与还款日期，支持日历和看板视图。  
  ![Notion 管理信用卡](https://img2020.cnblogs.com/blog/645649/202105/645649-20210530170230218-633347741.png)

- **Scriptable**：实时提醒还款日期，将关键信息显示在主屏幕。  
  ![Scriptable 显示提醒](https://img2020.cnblogs.com/blog/645649/202105/645649-20210530170305420-2026644351.png)

---

## 步骤详解

### 1. 在 Notion 中设置信用卡管理表格

#### 第一步：复制模板
打开 [Notion 信用卡管理模板](https://www.notion.so/0056ae1047944bd5a58c07fc80537884)，点击 **Duplicate** 复制到自己的账户中。

#### 第二步：填写卡片信息
将信用卡、花呗、白条等相关信息录入表格，重点填写 **还款日** 字段。  
- **剩余天数公式**：根据当前日期，自动计算还款剩余天数。  
```notion
dateBetween((prop("还款日") >= date(now())) ? dateSubtract(now(), date(now()) - prop("还款日"), "days") : dateSubtract(dateAdd(now(), 1, "months"), date(now()) - prop("还款日"), "days"), now(), "days")
```
- **下一还款日公式**：根据剩余天数计算下一个还款日。  
```notion
dateSubtract(dateSubtract(dateAdd(now(), prop("剩余天数"), "days"), toNumber(formatDate(dateAdd(now(), prop("剩余天数"), "days"), "HH")), "hours"), toNumber(formatDate(dateSubtract(dateAdd(now(), prop("剩余天数"), "days"), toNumber(formatDate(dateAdd(now(), prop("剩余天数"), "days"), "HH")), "hours"), "mm")), "minutes")
```

#### 第三步：创建私人 API 集成
1. 前往 [Notion Integrations](https://www.notion.so/my-integrations)，创建一个新的私人集成并命名为 **信用卡还款**。
2. 获取私人 API Token 并在 Notion 表格的共享设置中添加该集成。

#### 第四步：切换视图
Notion 支持 **日历视图** 和 **看板视图**，可根据需求调整查看方式。  
![日历视图](https://img2020.cnblogs.com/blog/645649/202105/645649-20210530170526984-1096690994.png)

---

### 2. 使用 Scriptable 创建屏幕组件

#### 第一步：下载 Scriptable
前往 [Scriptable App Store 页面](https://apps.apple.com/us/app/scriptable/id1405459188?uo=4) 并安装应用。

#### 第二步：编写脚本
1. 打开 Scriptable，创建一个名为 **信用卡** 的脚本。
2. 替换以下代码中的 `[你的私人Token]` 和 `[表格id]`：
```javascript
let url = "https://api.notion.com/v1/databases/[表格id]/query"
let req = new Request(url)
req.method = "POST"
req.headers = {
    "Authorization": "Bearer [你的私人Token]",
    "Content-Type": "application/json",
    "Notion-Version": "2021-05-13"
}
req.body = JSON.stringify({
    "filter":{
        "or": [
        {
            "property": "组织",
            "multi_select":{
                "contains":"银联"                
            }
        },
        {
            "property": "组织",
            "multi_select":{
                "contains":"互联网"                
            }
        }
        ]
    },
    "sorts": [
      {
        "property": "剩余天数",
        "direction": "ascending"
      }
    ]
})
let json = await req.loadJSON()
let results = json.results
const listView = new ListWidget()
results.forEach(item => {
    let name = item.properties.卡名.title[0].plain_text
    let days = item.properties.剩余天数.formula.number
    if (days < 7) listView.addText(`⚠️${name} 剩余 ${days} 天`)
})
Script.setWidget(listView)
Script.complete()
listView.presentMedium()
```

#### 第三步：添加组件到主屏幕
1. 长按主屏幕，添加一个中尺寸 Scriptable 组件。
2. 将组件关联至刚刚编写的脚本，并设置参数为 `7`（即提醒剩余 7 天内的账单）。  
![Scriptable 设置参数](https://img2020.cnblogs.com/blog/645649/202105/645649-20210530170630698-1069520617.png)

---

## 总结

通过本文介绍的 **Notion + Scriptable** 解决方案，您可以实现信用卡与花呗的高效管理，无需再为记忆还款日而烦恼。  
此外，配合 **WildCard 虚拟卡**，还能轻松解决海外订阅支付问题，进一步提升财务管理效率。

立即注册 WildCard：[https://bit.ly/bewildcard](https://bit.ly/bewildcard)  
使用邀请码：**ACCPAY**，尊享专属优惠！

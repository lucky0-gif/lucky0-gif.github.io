---
title: "n8n 笔记"
date: 2025-04-22T21:00:00+08:00
draft: false
weight: 3
tags: ["自动化工具", "效率提升", "实践记录"]
categories: ["latest-records"]
description: "记录我与自动化工具 n8n 的第一次亲密接触，从好奇到动手，关于探索、实现与创造的起点"
---

# 我的n8n学习笔记：从零开始的自动化之旅

## 初识n8n - 2024年2月5日

今天我终于决定尝试一下这个朋友一直在推荐的工具——n8n。据说它可以帮我自动化很多重复性工作，而且完全免费开源。作为一个经常被各种琐事淹没的普通人，我确实需要这样的工具来解放自己。

n8n全称是"node-eight-node"，读作"n-eight-n"。它是一个工作流自动化平台，可以连接各种应用和服务，让它们协同工作。我理解的简单说法是：它就像是一个数字世界的搭积木，你可以用它创建各种自动化流程，将不同的服务连接起来，让电脑自动完成原本需要人工干预的任务。

### 安装过程

今天尝试安装了n8n，比我想象的要简单：

1. 首先，我下载了桌面版（从官网n8n.io下载）
2. 安装过程很顺利，就像安装普通软件一样
3. 安装完成后，系统自动打开了浏览器，显示了n8n的界面

备注：也可以用Docker或npm安装，但作为新手，桌面版最简单。

```bash
# 如果你懂一点命令行，也可以用npm安装
npm install n8n -g
n8n start

# 或者用Docker
docker run -it --rm --name n8n -p 5678:5678 n8nio/n8n
```

## n8n基础概念 - 2024年2月6日

今天花了几个小时了解n8n的基本概念，记录一下便于我以后复习：

### 核心概念

1. **工作流(Workflow)**：整个自动化流程，由多个节点组成
2. **节点(Node)**：工作流中的一个操作单元，比如"发送邮件"、"获取数据"等
3. **触发器(Trigger)**：启动工作流的条件，如定时、收到邮件、网页事件等
4. **连接(Connection)**：节点之间的连线，表示数据流向

### 界面布局

- 左侧：节点库和创建的工作流列表
- 中间：工作流设计区域
- 右侧：节点配置面板

笔记：界面非常直观，拖拽操作让我这个新手也能快速上手。

## 我的第一个工作流：每日提醒 - 2024年2月7日

决定从一个简单的案例开始——创建一个每日提醒工作流。这个工作流会在每天早上8点发送一封包含今日待办事项的邮件给我。

### 步骤记录

1. 创建新工作流，取名"每日提醒"
2. 添加一个"Schedule"触发器节点
   - 设置为每天早上8:00触发
   - 时区设置为Asia/Shanghai

3. 添加一个"Google Sheets"节点
   - 连接我的Google账号（需要授权）
   - 选择包含我待办事项的表格
   - 配置为"读取"操作

4. 添加一个"Function"节点处理数据
   ```javascript
   // 筛选今天的待办事项
   const today = new Date().toISOString().split('T')[0];
   const todayTasks = items.filter(item => item.json.date === today);
   
   // 格式化邮件内容
   let emailBody = `<h2>今日待办事项 (${today})</h2><ul>`;
   
   if (todayTasks.length === 0) {
     emailBody += '<li>今天没有待办事项，享受轻松的一天吧！</li>';
   } else {
     todayTasks.forEach(task => {
       emailBody += `<li>${task.json.task} - 优先级: ${task.json.priority}</li>`;
     });
   }
   
   emailBody += '</ul>';
   
   return [{
     json: {
       subject: `今日待办事项 - ${today}`,
       content: emailBody
     }
   }];
   ```

5. 添加"Send Email"节点
   - 配置我的邮箱SMTP设置
   - 收件人设为我自己的邮箱
   - 主题使用Function节点输出的subject
   - 内容使用Function节点输出的content

6. 连接各节点：Schedule → Google Sheets → Function → Send Email

7. 保存并激活工作流

测试结果：成功！我收到了一封包含今天待办事项的邮件。这就是我的第一个工作流，虽然简单但非常实用。

## n8n的高级特性 - 2024年2月10日

今天深入研究了n8n的一些高级特性，发现它比我想象的强大得多：

### 条件执行

学会了使用"IF"节点创建条件分支。例如，我可以根据待办事项的优先级决定是发邮件还是发短信提醒：

```
IF节点配置：
条件：{{$node["Function"].json["priority"] === "高"}}
- 成立时：执行发送短信节点
- 不成立时：执行发送邮件节点
```

### 错误处理

发现了"Error Trigger"节点，可以在工作流出错时捕获错误并执行特定操作，比如发送通知。这对于维护稳定的工作流非常重要。

### 执行模式

n8n有多种执行模式：
- 单次执行：手动触发一次
- 定时执行：按计划定期执行
- Webhook执行：响应外部请求时执行

笔记：对于关键工作流，最好添加错误处理和通知机制。

## 实际案例：网站监控 - 2024年2月15日

今天我创建了一个更复杂的工作流——监控我关注的几个网站是否有更新，如果有，就发送通知给我。这个工作流包含了更多高级概念：

### 工作流设计

1. **Schedule节点**：每4小时执行一次

2. **HTTP Request节点**：访问目标网站
   - 方法：GET
   - URL：https://example.com/news
   - 认证：不需要（公开网页）

3. **HTML Extract节点**：提取页面内容
   - 选择器：article.news-item
   - 提取内容：标题、日期、链接

4. **Function节点**：检查是否有新内容
   ```javascript
   // 读取上次检查的结果
   const fs = require('fs');
   const path = require('path');
   
   // 数据存储路径
   const dataPath = path.join(__dirname, 'last_check.json');
   
   // 尝试读取上次的数据
   let lastResults = [];
   try {
     if (fs.existsSync(dataPath)) {
       lastResults = JSON.parse(fs.readFileSync(dataPath, 'utf8'));
     }
   } catch (error) {
     console.error('Error reading last results:', error);
   }
   
   // 当前结果
   const currentResults = items.map(item => item.json);
   
   // 查找新内容
   const newItems = currentResults.filter(current => {
     return !lastResults.some(last => last.title === current.title);
   });
   
   // 保存当前结果为下次比较基准
   try {
     fs.writeFileSync(dataPath, JSON.stringify(currentResults), 'utf8');
   } catch (error) {
     console.error('Error writing results:', error);
   }
   
   // 如果有新内容，传递给下一个节点
   if (newItems.length > 0) {
     return newItems.map(item => ({ json: item }));
   }
   
   // 没有新内容，返回空数组
   return [];
   ```

5. **IF节点**：检查是否有新内容
   - 条件：{{$node["Function"].json !== undefined}}

6. **Send Email节点**：通知有新内容
   - 使用模板构建邮件内容，列出所有新文章

### 遇到的问题和解决方案

1. **问题**：最初HTTP Request节点无法正确获取某些网站的内容
   **解决**：添加了User-Agent头，模拟正常浏览器访问：
   ```
   User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36
   ```

2. **问题**：有时网站结构变化导致HTML Extract失败
   **解决**：使用更通用的选择器，并添加错误处理节点

心得：这个工作流帮我节省了大量时间，不用频繁刷新网站查看更新了。

## n8n与其他系统集成 - 2024年2月20日

今天研究了n8n如何与其他系统集成，发现它支持惊人数量的第三方服务！以下是我测试过的几个：

### 1. 集成Slack

创建了一个工作流，每当我的博客有新评论时，自动发送通知到Slack频道：

1. **WordPress节点**：监听新评论
2. **Slack节点**：发送消息到特定频道
   - 需要创建Slack App并获取API Token
   - 格式化消息，包含评论者、内容和链接

### 2. 集成CRM系统

将新客户信息自动同步到CRM系统：

1. **Google Forms节点**：收集潜在客户信息
2. **HubSpot节点**：创建新联系人
   - 映射表单字段到CRM字段
   - 设置默认值和标签

### 3. 集成支付系统

监控新订单并处理：

1. **Stripe节点**：监听新支付事件
2. **Function节点**：处理订单数据
3. **Send Email节点**：发送确认邮件
4. **Google Sheets节点**：记录订单信息

笔记：n8n的真正威力在于它的集成能力。目前它支持280+种集成，几乎覆盖了所有主流服务。

## 安全和性能优化 - 2024年2月25日

随着我的工作流越来越多，开始关注安全性和性能问题：

### 安全最佳实践

1. **使用环境变量存储敏感信息**
   - 创建`.env`文件存储API密钥
   - 在节点中使用`{{$env.API_KEY}}`引用

2. **设置访问控制**
   - 启用基本认证
   - 配置用户名和密码

3. **数据加密**
   - 使用HTTPS
   - 加密存储的凭证

### 性能优化

1. **减少节点数量**
   - 合并相似功能的节点
   - 使用Function节点代替多个转换节点

2. **批量处理**
   - 使用"Split In Batches"和"Merge"节点处理大量数据

3. **设置超时和重试**
   - 为不稳定的服务添加重试机制
   - 设置合理的超时时间

笔记：随着依赖n8n的程度越来越高，安全性变得尤为重要。

## 实际案例：内容聚合器 - 2024年3月5日

最近我创建了一个内容聚合器工作流，它从多个来源收集内容，过滤、整理后发送给我。这是我目前最复杂的工作流：

### 工作流设计

1. **Schedule节点**：每天早上7点执行

2. **HTTP Request节点组**：从多个来源获取数据
   - RSS源
   - API接口
   - 网页抓取

3. **Merge节点**：合并所有来源的内容

4. **Function节点**：内容去重和过滤
   ```javascript
   // 去除重复内容
   const uniqueItems = [];
   const seenTitles = new Set();
   
   for (const item of items) {
     // 标准化标题用于比较
     const normalizedTitle = item.json.title.toLowerCase().trim();
     
     if (!seenTitles.has(normalizedTitle)) {
       seenTitles.add(normalizedTitle);
       uniqueItems.push(item);
     }
   }
   
   // 过滤掉不包含关键词的内容
   const keywords = ['人工智能', '自动化', '工作流', 'n8n'];
   const filteredItems = uniqueItems.filter(item => {
     const content = (item.json.title + ' ' + item.json.description).toLowerCase();
     return keywords.some(keyword => content.includes(keyword.toLowerCase()));
   });
   
   // 按发布时间排序
   filteredItems.sort((a, b) => {
     const dateA = new Date(a.json.pubDate || a.json.date);
     const dateB = new Date(b.json.pubDate || b.json.date);
     return dateB - dateA; // 降序排列
   });
   
   // 只保留最新的10条
   return filteredItems.slice(0, 10).map(item => ({ json: item }));
   ```

5. **HTML节点**：生成漂亮的HTML摘要
   ```javascript
   // 生成HTML摘要
   let html = `<h1>今日精选内容</h1>
   <p>生成时间: ${new Date().toLocaleString('zh-CN')}</p>
   <hr>`;
   
   items.forEach(item => {
     html += `
     <div class="item">
       <h2><a href="${item.json.link}">${item.json.title}</a></h2>
       <p class="date">发布于: ${new Date(item.json.pubDate || item.json.date).toLocaleString('zh-CN')}</p>
       <p class="description">${item.json.description || item.json.summary || ''}</p>
       <p class="source">来源: ${item.json.source || '未知'}</p>
       <hr>
     </div>
     `;
   });
   
   return [{
     json: {
       html: html
     }
   }];
   ```

6. **Send Email节点**：发送HTML摘要邮件

### 效果评价

这个工作流运行了一周，效果超出预期：
- 节省了我每天大约30分钟的信息筛选时间
- 内容质量高，相关性强
- 排版美观，阅读体验好

优化点：考虑添加机器学习模型进行内容分类和推荐。

## 团队协作与分享 - 2024年3月15日

今天和团队成员分享了我创建的几个工作流，记录一下经验：

### 导出和分享工作流

1. **导出工作流**
   - 在工作流页面点击"导出"按钮
   - 选择"导出为JSON"
   - 保存文件

2. **导入工作流**
   - 在工作流页面点击"导入"按钮
   - 选择JSON文件
   - 调整节点连接和配置

3. **分享最佳实践**
   - 使用明确的命名规则
   - 添加注释说明每个节点的作用
   - 创建文档说明工作流的目的和配置

### 团队工作流示例：客户反馈处理

与市场团队合作创建了一个客户反馈处理工作流：

1. **Google Forms节点**：收集客户反馈
2. **Function节点**：分析反馈情绪
   ```javascript
   // 简单的情绪分析
   function analyzeSentiment(text) {
     const positiveWords = ['喜欢', '满意', '赞', '好', '棒', '推荐'];
     const negativeWords = ['失望', '不满', '差', '退款', '投诉', '糟糕'];
     
     let score = 0;
     
     // 计算情绪分数
     positiveWords.forEach(word => {
       if (text.includes(word)) score += 1;
     });
     
     negativeWords.forEach(word => {
       if (text.includes(word)) score -= 1;
     });
     
     // 返回情绪类别
     if (score > 0) return '积极';
     if (score < 0) return '消极';
     return '中性';
   }
   
   // 处理每条反馈
   return items.map(item => {
     const sentiment = analyzeSentiment(item.json.feedback);
     item.json.sentiment = sentiment;
     return item;
   });
   ```

3. **Switch节点**：根据情绪分类处理
   - 积极反馈：保存到"好评集"
   - 消极反馈：创建客服工单
   - 中性反馈：保存待分析

4. **Slack节点**：发送每日反馈摘要给团队

团队反馈：工作流大大提高了客户反馈处理效率，客服响应时间从平均24小时减少到4小时。

## n8n的限制和解决方案 - 2024年3月20日

使用n8n几个月后，也发现了一些限制，记录一下我的解决方案：

### 限制1：复杂数据转换

当需要进行复杂数据转换时，Function节点的简单JavaScript有时不够用。

**解决方案**：
- 创建单独的微服务处理复杂逻辑
- 在Function节点中调用该微服务API
- 利用多个Function节点分解复杂逻辑

### 限制2：状态管理

n8n是无状态的，保存工作流状态有时很困难。

**解决方案**：
- 使用外部数据库存储状态
- 利用文件系统存储简单状态
- 设计工作流为"幂等"操作，可重复执行

### 限制3：复杂调度

需要比Schedule节点更复杂的调度逻辑。

**解决方案**：
- 使用外部调度系统触发webhook
- 创建"元工作流"管理其他工作流的执行
- 利用多个条件触发器组合实现复杂逻辑

笔记：虽然有一些限制，但大多数都有解决方案。n8n的开放性使我们能够通过扩展来克服这些限制。

## 我的n8n最佳实践总结 - 2024年3月25日

经过近两个月的使用，总结了一些个人最佳实践：

### 1. 工作流设计

- **单一职责**：每个工作流应只做一件事，但做好
- **模块化**：将复杂工作流拆分为多个小工作流
- **可重用**：创建通用组件工作流，被其他工作流调用
- **可测试**：每个关键节点后添加测试逻辑

### 2. 错误处理

- 每个工作流都添加错误处理机制
- 关键节点后添加检查点
- 使用try-catch捕获Function节点中的错误
- 设置通知机制及时发现问题

### 3. 文档和注释

- 为每个工作流创建详细文档
- 使用注释节点说明工作流各部分的功能
- 记录API调用的预期响应格式
- 定期更新文档以反映变化

### 4. 性能和安全

- 定期备份工作流
- 使用环境变量存储敏感信息
- 设置适当的执行超时和重试策略
- 监控资源使用情况

我的n8n使用感受：从最初的好奇尝试到现在的日常依赖，n8n已经成为我提高工作效率的得力助手。它让我把注意力从繁琐的重复任务中解放出来，专注于更有创造性的工作。

## 未来计划 - 2024年3月30日

随着我对n8n的熟悉，我有一些未来的计划和想法：

### 短期计划

- 探索n8n的REST API，实现程序化控制工作流
- 尝试使用Docker Compose部署更稳定的环境
- 研究如何与AI工具集成，如OpenAI的API

### 中期计划

- 创建自定义节点扩展n8n功能
- 构建工作流监控仪表板
- 研究如何与现有企业系统深度集成

### 长期愿景

- 建立完整的自动化生态系统
- 在团队中推广自动化文化
- 为n8n社区贡献代码或文档

通过持续学习和实践，我相信n8n将帮助我实现更多自动化的可能性，让我的工作和生活更加高效。

---

## 附录：我的常用节点速查表

| 节点类型 | 用途 | 配置要点 |
|---------|------|---------|
| Schedule | 定时触发工作流 | 设置执行频率和时区 |
| Webhook | 响应外部请求 | 记录生成的URL |
| HTTP Request | 访问API或网页 | 设置请求方法、头部和认证 |
| Function | 处理数据 | 返回数组形式的结果 |
| IF | 条件分支 | 使用表达式构建条件 |
| Send Email | 发送邮件 | 配置SMTP服务器 |
| Google Sheets | 读写表格 | 先创建凭证 |
| Slack | 发送消息 | 需要Slack API Token |
| HTML | 生成HTML内容 | 可用于邮件模板 |
| Error Trigger | 错误处理 | 与主工作流分开设置 |
# 验证码设计的梳理

## 验证策略

- 智能免验，仅关键场景先判断用户是否可疑再请求
- 二次验证，相同请求若可疑，则返回待验证或拦截状态
- 业务风控，每次请求都记录，需接入风控体系和打击手段

## 验证下放策略

- 防护等级，调整验证策略和可疑判断标准
- 动态验证，每次下放验证方式都不同
- 自定义，自定义验证图和验证方式，强调应用个性

## 验证方式

### 适老化验证

<img src="https://s1.ax1x.com/2022/09/15/vxo8pT.png" alt="适老化验证" style="max-width: 300px">

### 滑动拼图验证码

<img src="https://s1.ax1x.com/2022/09/15/vxoK7n.png" alt="滑动拼图验证码" style="max-width: 700px">

### 文字点选验证码

<img src="https://s1.ax1x.com/2022/09/15/vxou0s.png" alt="文字点选验证码" style="max-width: 700px">

### 移动拖拽验证码

<img src="https://s1.ax1x.com/2022/09/15/vxoZ6g.png" alt="移动拖拽验证码" style="max-width: 400px">

### 区域点击验证码

<img src="https://s1.ax1x.com/2022/09/15/vxoV1S.png" alt="区域点击验证码" style="max-width: 400px">

### 语义点击验证

<img src="https://s1.ax1x.com/2022/09/15/vxoEp8.png" alt="语义点击验证" style="max-width: 300px">

## 应用场景

- 登陆注册，抵御机器人恶意注册，阻止撞库攻击
- 活动秒杀，抵御爬虫拦截刷单，保障真实用户权益，避免不必要的运营效果下降和经济损失
- 点赞发帖，适用于论坛、投票等场景，抵御屠版、灌水、刷票
- 数据保护，抵御爬虫盗取网页内容和数据

***当前方案--B**

# 方案 A：纯脚本壳（推荐）
一个 analyze_fund.py 脚本，调现有 API，不做项目改造。

输入基金代码（如 110011 或 SPY）
用 akshare（A股）或 yfinance（海外）拉持仓列表
循环调 TradingAgentsGraph.propagate(ticker, date) 逐个分析
汇总各股 final_trade_decision，写一份基金总览 markdown
信号不一致时，可选再跑一轮 LLM 汇总
优点：零侵入，不动项目代码，一个文件搞定。复用 main.py 的 headless 模式。
缺点：顺序执行，10 只股票可能要跑很久；没有并行。

# 方案 B：批量 CLI 命令（tradingagents analyze-batch）
在 CLI 里加一个 analyze-batch 子命令，接受基金代码或多 ticker 列表。

新增 CLI command，走 Typer 的 --ticker-list / --fund-code 参数
内部循环调 propagate()，带进度条
生成汇总报告
优点：和现有 CLI 体验一致，可 checkpoint、可恢复。
缺点：改动 cli/main.py（1200+ 行的大文件），侵入较大。

# 方案 C：并行调度器
独立模块 + 可配置并发数，用 asyncio / concurrent.futures 并行跑多只股票。

优点：快，10 只可以 3-4 只同时跑。
缺点：LLM API 限流风险；复杂度高不少。

我的推荐：方案 A。原因：

你的需求本质是"脚本化批量调用"，不是新 feature
一个 analyze_fund.py ≤ 200 行，立即可用
不碰现有代码，后续加并行或并进 CLI 都可以
先做最简单的，能用再说。你觉得哪个方案？或者有其他想法？
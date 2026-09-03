场景
用 Dify（v1.16，Docker 部署）搭了个「信息整理」工作流：开始节点(raw_text) → LLM 节点 → 结束节点，输入长文本输出摘要/重点/思维导图/错题。

现象
运行后模型总回复「请提供待整理原文」，看起来像"检测不到文本"。

排查
1. 看后端日志，不靠猜。 Dify 是 Docker 跑的，报错都在容器日志里：


docker logs docker-worker-1 --tail 300 | grep -iE "error|exception"
第一层问题：模型返回 402 余额不足——当时配的第三方中转站 ccsub.net 没钱了，LLM 节点直接 ABORT，整条流程无输出。

2. 换 DeepSeek 后报错消失，但输出仍错。 关键点：workflow_runs 里运行状态是 succeeded，但成功 ≠ 输出正确。查 workflow_node_executions 表看到每个节点的真实数据：

节点	实际数据
开始节点	raw_text = "InnoDB是…" ✅ 文本进来了
LLM 节点	输出 "请提供待整理原文" ❌ 模型没拿到
证据指向：文本进了工作流，但没进提示词——问题在变量绑定。

3. 读框架源码定位。 翻 graphon/nodes/llm/llm_utils.py，渲染逻辑是：


if message.edition_type == "jinja2":
    # 走 Jinja2 渲染，替换 {{var}}
else:
    # basic 模式，只认 {{#node_id.变量名#}}
根因
Dify 变量有两套语法：basic 模式用 {{#node_id.变量名#}}，jinja2 模式用 {{变量名}}（需消息标记 edition_type: "jinja2"）。

我手打了 {{raw_text}}（jinja2 语法），但消息没标 jinja2 类型，Dify 按 basic 处理，{{raw_text}} 被当成死字符串，正文从未填进去。

修复
给「用户」消息补上 jinja2 标记：


  "role": "user",
  "text": "待整理原文：{{raw_text}}",
+ "edition_type": "jinja2",
+ "jinja2_text": "待整理原文：{{raw_text}}",
改完重新运行即正常。

经验
症状 ≠ 根因：表面是"检测不到文本"，实际是"变量没绑定"（还叠了一层余额不足）。
看日志和数据库比看截图靠谱：docker logs + 查 Postgres 的 workflow_runs / workflow_node_executions 能拿到节点真实进出数据。
succeeded 不代表输出正确，要看实际内容。
Dify 里变量要用编辑器里的变量选择器插入，别手打双花括号。
搞不清框架行为时，读源码是终极大招。

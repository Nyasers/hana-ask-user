# hana-ask-user

> Hana Recipe ID：`hana-ask-user`

让 Agent 在必须由人裁决的地方停下来，投出一张提问卡：确认一件事、在几个选项里选、或者补一个它自己查不到的事实。用户在卡上作答，答案作为 `hana-ask-user.answer` 卡事件回到会话，Agent 在新回合里接着走。

这是 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 的 `ask_user_question` 的 Hana 卡片版复刻。

## 安装

recipe 走宿主的扩展安装入口，kind 为 `recipe`，source 指到本目录：

1. 安装入口收下本目录，暂存出 `recipe:hana-ask-user`；
2. 在安装审阅卡上确认；
3. 装好后 recipe ID 为 `hana-ask-user`，卡模板路径为 `hana-ask-user/assets/ask.card.html`。

## 用法

Agent 侧：把要问的事写进 `show_card` 的 `state`（题组一到四题），投卡后当回合即结束；等答案事件到达再继续，不猜、不代答、不重复投卡。

用户侧：卡一次显示一题，左侧 Skip、右侧 Previous/Next，最后一页是 Submit；整组一次性提交，跳过的题按空作答计入已完成。

## 许可

MPL-2.0，见 `LICENSE`。

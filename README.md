<div align="center">
	<img width="128" src="docs/nodice.png" alt="logo"></br>

# NoDice

*基于[ nonebot ](https://github.com/nonebot/nonebot)以及[ go-cqhttp ](https://github.com/Mrs4s/go-cqhttp)的轻量化QQ跑团掷骰机器人*

~~然而会有一些奇奇怪怪的扩展功能~~

[![License](https://img.shields.io/github/license/thereisnodice/nodice)](LICENSE)
![Python Version](https://img.shields.io/badge/python-3.7+-blue.svg)
![OneBot Version](https://img.shields.io/badge/OneBot-v11-black)

</div>

### Features

*理论上来说 `plugin` 文件夹里的所有插件都可以在 nonebot 基础框架的机器人上面即插即用，但是由于大多数 nonebot 机器人（如 HoshinoBot ）已经被二次封装，所以请不要对于这项特性抱有太大希望。*

*由于对松耦合的执着，我把原先作为一个整体的 Dice! 拆散成了多个插件并且复用了多个与 nonebot 框架**完全松耦合**的脚本，所以你很有可能在多个插件目录下都见到同样的文件，例如 `roll_dice`、`draw_deck`、`char_make` 里面都有 `calculator.py` ~~（以后可能还会有更多的插件包含这个东西）~~，请不要对此感到疑惑。*

- [x] Git热更新 `.bot update`
- [x] 重载插件 `.bot reload`
- [x] 标准掷骰 `.r(h) 4#3d6k2+5 reason`
- [x] 天气查询 `.weather` （目前NLP有bug）
- [x] 在线IDE `.exec python`
- [ ] Github Webhook 推送
- [x] 原Dice! 的牌堆功能 `.draw deckname`
- [ ] 各音乐平台点歌
- [ ] 机器人分群管理
- [x] 人物作成 `.sg/sk/coc/dnd`
- [ ] COC检定、成长与奖惩骰
- [ ] FATE掷骰
- [ ] WOD掷骰
- [ ] 人物卡管理
- [x] 今日人品
- [x] 使用说明 `.help`
- [x] CSU课表查询 `.csu`（此功能不开源）

### TO DO List

- 兼容 [SinaNya](https://sinanya.com/) 的 yaml 牌堆
- 链接 sqlite3 数据库
- ~~csu_class 可视化~~ 这是另一个项目了
- 接入数据库的群管功能
- 接入数据库的人物卡功能
- 所有的异常处理
- Dice! 556 mod 形式的help功能
- Dice! 2.5 的lua脚本

### Project for nonebot2

*大概会在1月20日之后开始迁移到 nonebot2 ，应该是新开个叫 nodice2 的仓库。*

*本仓库不会停止更新，只是随缘修复bug，不再增加新功能。*

迁移计划大概如下：

1. 首先是 bot_control ，没有这个插件的话远程部署和调试都会很麻烦，所以会优先开发本插件（不过貌似 nonebot cli 支持更新插件的功能，到时候如果能用它代替就不费劲了）。
2. 因为 nonebot2 支持子插件，所以 nodice2 将会作为一个完整的 nonebot2 插件，然后把 nodice 的那些独立插件都重写为子插件（这样做的好处就是我不用再去考虑各个命令模块之间的耦合度了）。
3. 由于核心功能 roll_dice 的核心 `calculator.py` 与 nonebot 是**完全松耦合**的，所以迁移这个功能本质上没有什么困难，当然如果能顺手解决效率问题自然是更好。
4. 基于数据库的群管、人物卡、今日人品功能，正在考虑是使用 nonebot2 插件商店里的 database 插件对接还是用自写的 `sqlite.py` （为了与其他插件松耦合我大概率会选择后者，欢迎来说服我） 。
5. 与 Dice! 无关的功能大概不会迁移到 nonebot2 ，或者说不会 push 到新仓库。

### Notes

1. 出现了个奇怪的问题，当我把bot重命名为bot_control之后，通过bot update命令让挂载在服务器的机器人进行更新后，bot插件明明加载了却无法对我的指令响应，此bug待研究，先记着。  
解决办法：原来是无序集惹的锅，本来我的重载函数是先重载机器人中已加载的插件，再去加载用os.path读取到的新插件，结果无序集这坑爹类型把顺序给整乱了，目前改成列表就没事了，只是会重载两遍函数，不过这并不是什么严重的问题（相对于之前来说）。  
2. 目前的dice_calculator无法识别并列的括号如 `(5+3)*(6+2)` ，只会识别嵌套括号如 `3*((6+2)*5)` ，目前显示计算过程的功能也因为重构不完全暂时挂掉了。  
解决办法：遍历算法的锅，我为了减少时间复杂同时从左边查找左括号和从右边查找左括号，结果导致了无法识别并列括号的问题。只要先从左边查找右括号，再从右括号的位置往左查找左括号，然后递归查找就行了。显示计算过程的功能也通过重构为Calculator对象的方法恢复了，不过目前没有报错功能，所以表达式输错了机器人只会不鸟你。
3. jrrp 功能目前暂时使用 json 实现了本地化储存，所以会与溯洄骰和塔骰有所不一致，以后完成了 `sqlite.py` 储存方案可能会换成 sqlite 数据库。
解决方案：无，我不会用别人的 API 的，也懒得自己搞 API 。
4. 使用简单粗暴的便利法搞定了 Dice! 的类 json 牌堆功能，当然问题还有很多，比如估计永远也不会支持牌堆牌数之类的。在这里简单说一下本项目的牌堆算法与 Dice! 的不同之处， Dice! 的牌堆实际上是将所有牌堆读取到内存中( Dice3 则是载入到数据库，不过 Dice3 并不支持自定义牌堆），然后提取字符串把它作为一个命令通过 `dice_utils` 模块处理得到想要的结果再输出。我的方法就比较蛋疼了，每次使用牌堆指令都会重新读取一次牌堆文件夹（懒得手动载入牌堆），所以理论上来说牌堆越多效率越低。接着通过递归遍历识别`{牌堆引用}`、`{%不放回牌堆}``[掷骰公式]`，再拼接得到想要的结果。  
解决方案：懒得想，等以后迁移到 nonebot2 再说。

### Lisence

go-cqhttp下的文件 ([go-cqhttp](https://github.com/Mrs4s/go-cqhttp) 的可执行程序) 保持使用原 [AGPL-3.0 License](https://github.com/Mrs4s/go-cqhttp/blob/master/LICENSE) 许可

项目中其余内容使用 [MIT License](LICENSE)

### Thanks

- @w4123 溯洄，Dice! 与 Dice3 的开发者，本项目命令格式大部分借鉴自其。
- @233a344a455 DeltaBot的开发者，本项目的 bot_control 插件修改自其。

### Nothing useful

本项目旨在用 python 移植 CQ原生插件 Dice! ~~（虽然本项目的架构更像是 Dice3）~~。

之所以取名 NoDice ，是因为一开始只是想要把 Dice的感叹号移到前面来，也就是 NotDice ，刚好可以把组织名取作 `this is not dice` 来玩双关。接着因为本项目基于 nonebot 框架，就想取个相近的名字，而 not 和none 就只有 no 这两个字相同了，组织名刚好也能玩双关，只是意思从“这不是骰子”变成了“这里没有骰子”~~（当然我更喜欢翻译为无骰骑士异闻录）~~。
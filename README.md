# BuildrootManualChinese
Buildroot 2025.05 手册 - 中文版（AI翻译）



译文在 [Github 仓库](https://github.com/Staok/BuildrootManualChinese) 和 [gitee 仓库](https://gitee.com/staok/BuildrootManualChinese) 保持最新，其它平台发的文档可能不会与之同步。

**希望能够共同维护这个 仓库的 Buildroot 手册 中文译文，帮助更多人真正深入学习理解，更好的工作、生活和创造。**



关于 AI 提示词 以及 更多工具 的收集，有一个仓库：

- Github [Awesome-AI-Tools/AI工具和用法汇总.md at main · Staok/Awesome-AI-Tools (github.com)](https://github.com/Staok/Awesome-AI-Tools/blob/main/28AI工具和用法汇总.md)。
- Gitee [AI工具和用法汇总.md · 瞰百/Awesome-AI-Tools - 码云 - 开源中国 (gitee.com)](https://gitee.com/staok/Awesome-AI-Tools/blob/main/28AI工具和用法汇总.md)。



# 说明



buildroot 官方手册网址：
 https://buildroot.org/downloads/manual/manual.html



网上的一些翻译文档，但是有点旧了

- [翻译: buildroot 用户手册 (更新中...) - Mojies - 博客园](https://www.cnblogs.com/mojies/p/14916089.html#_getting_started)
- [Buildroot用户手册中文版(正点原子翻译)_V1.0-OpenEdv-开源电子网](http://47.111.11.73/thread-328876-1-1.html)



- [Buildroot用户手册_海漠的博客-CSDN博客](https://blog.csdn.net/haimo_free/category_10247894.html)
- [8. Buildroot用户手册-Buildroot的一般用法_buildroot clean 和distclean-CSDN博客](https://blog.csdn.net/haimo_free/article/details/107723343)



过程：

手动拆分最新官方文档（2025.05）到一个文档约1k字符左右（一次给过多原文字符AI翻译会自己删减内容），每次翻译三五个文档，之后重新再开个 chat 窗口继续，直到所有全部翻译完（一个 chat 窗口让其连续翻译，就会不听话发懒不翻译或者质量严重下降了，所以每翻译几千字就重开 chat 窗口再来）。

下面译文若有与原文对不上的，敬请指出



使用 copilot 的 GPT4.1 的 agent 模式完成。

提示词输入：

你是一个中英文翻译专家，将用户输入的中文翻译成英文。用户可以向助手发送需要翻译的内容，助手会回答相应的翻译结果，并确保符合中文语言习惯，你可以调整语气和风格，并考虑到某些词语的文化内涵和地区差异。同时作为翻译家，需将原文翻译成具有信达雅标准的译文。"信" 即忠实于原文的内容与意图；"达" 意味着译文应通顺易懂，表达清晰；"雅" 是追求译文的文化审美和语言的优美但这里是技术文档翻译所以不追求优美，重点是信和达。而且注意，对于名词的翻译，还要加括号著名英文原文，确保是一篇易懂且准确的译文。只给出译文结果即可。 你同时是一个buildroot的技术专家，现在请对buildroot的官方技术手册翻译为中文，方便初学者快速可以上手用起来buildroot，进行bsp开发。 以下我将不断给出原文，请翻译为中文的参考手册。按照我给的原文如实翻译，不要精简和归纳。对于执行脚本和代码使用代码块格式。全文分好标题层级。
 原文文件为当前目录下的buildrootManualX.X.md格式的文件，以1.0、1.1...2.0.2.1...排序。逐个文件翻译后输出buildrootManualChineseX.X.md文件，全程自动无人值守静默执行。开始。



之后经过手动章节校对，目录与官方文档一致，但是译文可能与原文有稍许的差异，**如您确定译文有误或有不妥，大可直言提出优化，帮助译文改进。**



正文见 buildrootManualChinese.md

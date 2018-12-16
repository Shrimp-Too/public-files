# 一份收集 Zotero Translators 网站信息的报告

## 0. 摘要

本报告重点讲述了我是如何尝试收集 Zotero translators 的网站信息的。

## 1. 背景

信息分析课讲解了一个知识管理工具 Zotero， 还提到它的 translator 提供了一批优质的信息源，因为之前有写爬虫的经验，我就想动手尝试一下。

## 2. 信息收集整理的过程

### 2.0 整个流程框架

- 使用 Python 写爬虫抓取所有 translator 的源信息，包括 target 信息
- 尝试把 Target 转换为目标网站，最终放弃
- 直接搜索 label 来获得对应网站
- 尝试获取每个网站的站点信息
- 合并生成原始数据

### 2.1 使用 Python 写爬虫抓取所有 translator 的源信息，包括 target

虽然曾经用 Python 写过一个爬虫，但这一步还是耗费了不少时间，主要是熟练度还不够，而且没有掌握一套关于编程的全局知识。

我所用的方式：首先抓取 [translators 页面](https://github.com/zotero/translators)的所有 translator 名称和链接，做成一个表格，形如：

| title           | url                                             |
| --------------- | ----------------------------------------------- |
| ACLAnthology.js | /zotero/translators/blob/master/ACLAnthology.js |

发现 url 格式都是一样的，并且每个 translator 都可以通过[以下网址](https://raw.githubusercontent.com/zotero/translators/master/Douban.js)获得源码：

`https://raw.githubusercontent.com/zotero/translators/master/`  +  `TITLE.js`

然后按照上面地址去抓取每个 translator 的源信息，下面是最终生成表格的栏目和第一行内容：

| title                 | status | translatorID                         | label              | creator           | target                                                       | minVersion | maxVersion | priority | inRepository | translatorType | browserSupport | lastUpdated     |
| --------------------- | ------ | ------------------------------------ | ------------------ | ----------------- | ------------------------------------------------------------ | ---------- | ---------- | -------- | ------------ | -------------- | -------------- | --------------- |
| A Contra Corriente.js | Normal | bbf1617b-d836-4665-9aae-45f223264460 | A Contra Corriente | Sebastian Karcher | ^https?://acontracorriente\.chass\.ncsu\.edu/index\.php/acontracorriente/ | 2.1        |            | 100      | TRUE         | 4              | gcsibv         | 8/18/2016 20:51 |

其中最关键的就是 `target` 信息，表示对目标网站的匹配规则；`translatorType` 表明这个 translator 的种类是网站还是别的；`lastUpdated` 最后更新时间也值得参考。更详细情况请参考这个[官方说明](https://www.zotero.org/support/dev/translators#metadata)。

### 2.2 尝试把 Target 转换为目标网站，最终放弃

参考官方文档，抓取到的源信息中的 target 的格式是 JavaScript 的正则表达式，我的理解是 translator 插件就是根据这个正则表达式来判断你现在访问的网址是否是目标网站。例如豆瓣网的 target 是这样的：

`^https?://(www|book)\.douban\.com/(subject|doulist|people/[a-zA-Z._]*/(do|wish|collect)|.*?status=(do|wish|collect)|group/[0-9]*?/collection|tag)`

那么怎么把 target 转换成网址呢？先尝试用 Python 对字符串的一些方法处理，发现各种符号例如`/`和`\`太多了，识别起来非常复杂，几乎不可能。那么让我们动用强大的正则表达式吧，先学习了一下怎么使用正则表达式，然后做了一些尝试，发现哪怕用正则表达式也很难找到一个通用的匹配方法，因为这字符串本身就是各种各样的正则表达式匹配规则，有的简单，有的非常复杂，例如：

`^https?://(ukads|cdsads|ads|adsabs|esoads|adswww|www\.ads)\.(inasan|iucaa\.ernet|nottingham\.ac|harvard|eso|u-strasbg|nao\.ac|astro\.puc|bao\.ac|on|kasi\.re|grangenet|lipi\.go|mao\.kiev)\.(edu|org|net|fr|jp|cl|id|uk|cn|ua|in|ru|br|kr)/(cgi-bin|abs)/`

怎么办？既然它自己就是匹配规则，那么让它自己发挥作用好了，直接找一些网址让它自己去匹配。网址从何而来呢？可以去 Google 这些 Label，抓取一些搜索结果的网址让 target 自己去匹配，成功了说明他就是我们的目标网站。然而，这方法听起来非常美妙，实际操作遇到了困难。

首先是发现这个 Target 的表达式不能直接当作正则表达式用，似乎还要先学 JSON 语法，太麻烦了。至此，把 target 直接转换成目标网站的想法，被我搁置了。

### 2.3 搜索 label 来获得对应网站

想来想去，干脆简单一点，直接用 Google 搜索 label，然后用结果网址和 label 的相似程度来筛选搜索结果。在每个 label 之后加上 website 提高一点准确程度，例如：

`douban website`

这个时候，出现一个拦路虎：Google， Google 的结果虽然比较准确，但它对爬虫很严格，第一个网址都没抓到就已经判定我是机器人给屏蔽了，解决方法是模拟真实浏览器（ Selenium 或直接用 [GoogleScraper](https://github.com/NikolaiT/GoogleScraper) 库），但这种方法我不熟悉，太费劲。同时，在搜索过程中发现 Bing 似乎对爬虫很友好，遂改用 Bing。

测试过程中发现，第一个搜索结果准确度还不错，再看看已经花费的时间，干脆直接用第一个搜索结果的网址作为目标网站，不再对比了（保留前3个搜索结果网址作为参考）。

### 2.4 获取每个网站的站点信息

之前我就在使用 [SimilarWeb](https://www.similarweb.com/top-websites) 的网站，而且信息分析课也推荐了，它提供了丰富的信息，可以进行很多维度的分析，有明确的网站分类。可惜 SimilarWeb 的网站本身对爬虫毫不留情， API 则需要收费，而且价格是不透明的，要和销售人员协商。那么换成 Alexa 的排名，网站本身也屏蔽爬虫，但最终找到一个可用的 API 接口如下：

`https://data.alexa.com/data?cli=10&dat=s&url=`  +  `website.com`

这个 API 接口的结果用浏览器打开是[这样的](https://data.alexa.com/data?cli=10&dat=s&url=https://www.baidu.com)，这一部分抓取到的数据如下所示，注意到这个 country 被错误地被标为 China ，但是globalRank是比较准确的：

| title           | url1                     | globalRank | country | countryRank | url2                       | url3                                     |
| --------------- | ------------------------ | ---------- | ------- | ----------- | -------------------------- | ---------------------------------------- |
| ACLAnthology.js | www.aclweb.org/anthology | 39269      | China   | 9779        | aclweb.org/anthology/P/P17 | https://github.com/acl-org/acl-anthology |

最后合并源信息和站点信息，输出为原始数据。

## 3. 结论和初步分析

这份数据是非常粗糙的。由于使用了在 Bing 搜索引擎直接搜索 Label 的方法，并且对搜索结果没有比较、核对，导致 translator 对应的网站有不少错误。因为程序不太完善，有一些 Rank 很低的网站没有抓取到 Alexa 的数据。最重要的是，相比 SimilarWeb, Alexa 提供的站点排名信息太简略，对于 translator 的分类、筛选帮助不大，这也是最需要改进的地方。

在检查中，发现有一个错误是 Alexa 这个接口返回的数据中，将不少非中国的站点所属地标为中国，其他国家有没有错标还不清楚。

那么还有哪些工具提供关于站点的数据呢？做完之后发现有一个 [SimilarSites](https://www.similarsites.com/site/douban.com) 提供了分类和话题信息，不知道它对爬虫是否友好，需要再去尝试。

简单分析一下现有的不完美的数据，有166个 translator 的网站在 Alexa 全球排名前5000；有89个在全球前1000。还可以怎么分析，我目前想不到了，大家可以思考，源文件我放在了[这里](https://github.com/Shrimp-Too/public-files)。

## 4. 收获和讨论

对我来说，虽然结果不完美，但有这些收获：

- Bing 对爬虫非常友好，Google 完全相反。
- 正则表达式非常重要，非常好用，一定要再多练习一下。
- 在这个项目中经常需要检索信息，为了尽快找到答案，对如何检索不假思索，说明这一块还没有形成习惯、成为内隐知识。
- 对于这个项目，因为时间关系，没有把它作为实践信息分析所学的一个机会，导致交叉验证、检索技巧等框架都没怎么使用，那接下来需要思考这份报告应该如何用课程所学来改进。

项目进行一半的时候看到阳老师在微信上的教学示范，很多点值得思考：

- 元数据『借助简单的编程技巧批量抓取』，是什么编程技巧呢？
- 『不懂编程怎么办？参考第四讲的爬虫技巧』，用现成的爬虫技巧如何实现？
- 『思维路径的差异，导致的求解速度与准确度的差异』，如何提高这个项目的速度和准确度？我用了很多时间，得到的结果也不准确。

## Change Log

- 12/15/2018 Sat 5:43:48.11 初稿
- 12/15/2018 Sat 6:05:37.30  增加对阳老师教学示范的讨论

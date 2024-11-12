Welcome to policy citation knowledge graph!
Here is my project about KG of science and technology policies. This project adds a perspective of citation into the policies.

Given a title of a policy, you can find out: Who makes the policy? Which policies does it cite? Which policies cites it?

Further experiment can be conducted in: 1.what are the main policies that cited at most? 2.which is the longest citation chain? 3.Is longer chain means better policies? 4.are there citation community? what is the meaning?

How this citation has been made?

## 1 模式层构建

## 2 政策文献获取与处理

### 2.1 数据来源

数据来源于北大法宝网站

### 2.2 筛选标准

数据选取的是北大法宝“科技”专题下2019-2024年的所有政策（包含中央政策和地方政策），以及与这些政策具有引用关系、相关关系的其他政策。

### 2.3 数据获取与处理步骤

数据获取与预处理步骤分为四步，第一步为通过八爪鱼爬取北大法宝“科技板块”，获取“待爬取的政策链接.xlsx”；第二步为通过python的第三方包selenium模拟人的鼠标进行操作爬取网页链接，获取“政策表.xlsx”和“关系表.xlsx”;第三步为为获取“关系表.xlsx”中存在但“政策表.xlsx”中不存在的政策，并更新“政策表”；第四步为填补空值，对政策进行编号，在关系表中添加政策编号字段。（如图4.1所示）。

### 2.3.1 步骤一

步骤一是通过八爪鱼爬取北大法宝“科技政策”板块的数据。八爪鱼是一款可以模拟鼠标点击进行操作的软件，适合用于定位不复杂、切换页面后网页结构稳定的页面，本实验采用如图所示的流程编写八爪鱼任务流程，其中点击元素进行查询的步骤分为“选择中央政策/地方政策”、“选择专题分类为科技”、“选择单页展示100条”、“选择按引用量从高到低排序”。由于北大法宝一次展示数不超过500条，逐步选取各个省份的各个机关进行分类爬取。最终爬取到“科技”板块下的所有链接，包含中央政策1832条和地方政策29190条链接。

### 2.3.2 步骤二

步骤二是通过python的第三方包selenium模拟人的鼠标进行操作爬取网页。爬取的链接来自于步骤一获取的北大法宝链接，需要爬取政策的关键字段和网页右侧的引文关系、相关关系。

首先定义主函数get_passage_by_row(start, end)，在主函数中使用pandas读取xx文件。遍历每一行调用get_important_element(id, title, url, driver)、scrape_block(driver)、write_to_csv(id, title, result)。其中start表示开始的链接编号，end表示结束的链接编号，使用这两个参数可以在发生爬虫中断时在中断点再次调用get_passage_by_row(start, end)函数，实现流畅爬取，注意在发生中断时使用try语句捕捉异常TimeoutException和WebDriverException。 

get_important_element(id, title, url, driver)函数首先通过Xpath，“//*[@id="gridleft"]/div/div[1]/div[2]/ul”，定位所需字段的外层元素outer_element，再通过TAG_NAME，“strong”定位内层元素inner_elements，即所需字段，如标题、发布机构、政策类型等。对于inner_elements进行遍历，判断其父节点div的文本是否为所需字段，如果是则写入document_num的字段。最终通过Xieru_CSV()将字段保存入“中央政策_1832条”和“地方政策_29190条”的csv文件。

scrape_block()函数通过两个try语句分别对“法宝联想”和“智能发现”板块进行检测，如果出现NoSuchElementException则表示没有该板块。以“法宝联想”板块为例，首先通过xpath定位parent_element，“//*[@id="pkulaw-association"]”再通过CLASS_NAME，“lenovo”，定位inside_element。对inside_element进行遍历，判断TAG_NAME为h4的元素文本是否为“本篇引用”或“引用本篇”，之后调用BenPianYinYong()对其进行爬取。最终爬取的引用关系和被引关系、相关关系保存入字典中，返回为result。

BenPianYinYong(div_element)爬取div_element模块下的<a>元素，存储关联政策的标题和链接。

write_to_csv(id, title, result)函数获取id，title和result，读取result中的引用关系、被引关系、相关关系字典，将其存储为（政策A，关系，政策B）的三元组形式，命名为“地方政策_关系”和“中央政策_关系”。

最后将“地方政策_关系”和“中央政策_关系”汇总到“关系表.xlsx”中，将“中央政策_1832条”“地方政策_29190条”汇总到“政策表.xlsx”中。

### 2.3.3 步骤三

步骤三为获取“关系表.xlsx”中存在但“政策表.xlsx”中不存在的政策。对“关系表.xlsx”中的政策在excel表中使用VLOOKUP函数，精确查找“政策表.xlsx”中是否有该项政策，筛选出返回值为None的政策，存入“待爬取.xlsx”文件中，使用步骤二中的get_important_element（）对政策提取关键字段，并存储入“政策表.xlsx”文件中。

### 2.3.4 步骤四

步骤四为处理“政策表.xlsx”和“关系表.xlsx”文件。

首先对这两个文件进行去重操作，对政策表中的数据进行编号，查找空值插入“0”。其次，将两个文件通过VLOOKUP函数进行关联，在关系表中输入公式返回政策对应的编号，见公式（4.1）所示。

VLOOKUP(D11,'[政策表.xlsx]Sheet2'!$B:$C,2,FALSE)      （4.1）

## 3 数据获取与处理

本次爬取网站共获取中央政策3153条，地方政策33991条，共37144条政策。截止日期到2024年3月31日，地方政策中失效650条，尚未施行21条，已被修改150条，现行有效33135条，部分失效35条；中央政策失效107条，尚未施行1条，已被修改90条，现行有效2948条，部分失效7条。

## 4 知识图谱构建

### 4.1 实体识别

实体识别分为政策实体识别和发文机构实体识别两部分。政策实体识别需要从“政策表.xlsx”中抽取“编号”“标题”“是否有效”“发文字号”“法规类别”和“效力位阶”字段，形成“政策实体表.xlsx”。机构实体识别在关系抽取中完成。

### 4.2 关系抽取

关系抽取分为抽取机构和政策之间的发布关系及政策之间的引用关系。

首先抽取机构和政策之间的发布关系，抽取“政策表.xlsx”中的“编号”和“发文机构”两个字段，并将“发文机构”字段拆解为多个机构，分别存储入“机构_政策_关系表.xlsx”中。这里使用ast包中的liter_eval方法，institutions = literal_eval(institutions_string)，将[“机构1号”, “机构2号”]转变为列表，以此实现对机构的遍历，将发布关系以字典的格式存储入relations列表中。

对于机构实体的抽取问题，需要在抽取出institutions之后对其进行编码，用字典institution_dict存储机构编号和机构名，如果新的机构不在字典中，则将新机构添加入字典。

最后，将relations转化为dataframe格式relations_df，使用relations_df['发文机构'].apply(lambda x: institutions_dict[x])语句将发文机构和发文机构编号对应，并添加字段“发文机构编号”进入relations_df，最后存储入“机构实体表.xlsx”和“机构_政策_关系表”。

对于抽取政策之间的引用关系，提取“关系表.xlsx”中的“政策A编号”“关系”“政策B编号”三个字段，存储为“政策_政策_关系表.xlsx”。

### 4.3 图谱构建

### 4.3.1 实体导入

首先将使用py2neo将实体表“政策实体表.xlsx”和“机构实体表.xlsx”导入到neo4j中，以下是将政策实体表导入到neo4j的代码。需要注意到直接使用csv文件在neo4j软件操作时可能会出现中文无法识别的情况，因此我们使用py2neo库进行操作，读取excel文件中的字段后在创建节点，避免了neo4j处理中文编码和解码时可能出现的问题。

在导入实体时，第一步时连接到neo4j数据库，需要在neo4j软件中激活数据库后再连接，输入用户名、尼玛、数据库名。第二步时用pandas读取excel文件，并用df.iterrows()对每一行进行遍历，index指的是行数，row指的是列数。第三步是检查节点是否存在，需要使用到NodeMatcher包对节点进行查询，如果不存在则定义Node的参数后创建节点。

### 4.3.2 关系导入

其次将“机构_政策_关系表.xlsx”“政策_政策_关系表.xlsx”导入neo4j，注意检查是否已存在相同属性的节点，即使用merge方法将source_node和target_node消除歧义，再创建关系，否则会报错出现两个编号一样的节点。

解释文件调用方法：
首先运行爬虫文件，输入“中央汇总”和“地方汇总”两个文件，输出其关系、全字段实体。经过整理后得到“中央、地方、补充汇总文件”
运行neo4j文件，输入补充汇总文件，进行关系拆分后将元组输入neo4j，即完成数据库的搭建工作！

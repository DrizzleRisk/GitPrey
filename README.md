## GitHub敏感信息扫描工具
### 功能设计说明
GitPrey是根据企业关键词进行项目检索以及相应敏感文件和敏感文件内容扫描的工具，其设计思路如下：
* 根据关键词在GitHub中进行全局代码内容和路径的搜索（in:file,path），将项目结果做项目信息去重整理得到所有关键词相关的项目，即疑似项目结果；
* 基于PATTERN_DB中的敏感文件名或敏感代码对所有疑似项目做文件名搜索（filename:）和代码搜索（in:file）；
* 将匹配搜索到的结果按照项目整理输出；
由于无法做到精确匹配和精确识别，因此扫描结果或存在一定的漏报（比如项目中未出现关键词路径或内容）或误报（比如第三方项目引用关键词内容）情况，其中漏报的原因还包括Github的搜索限制：
* 默认只搜索主分支代码，多数情况下是master分支；
* Github最大只允许搜索1000条代码项，即100页代码；
* 代码搜索仅搜索不大于384Kb的文件；

此外，不同关键词搜索的疑似项目数量不同，少则数个，多则数十个甚至数百个，并会对搜索和扫描时间造成直接影响（另一影响因素是匹配的文件名关键词数量和内容关键词数量），项目和关键词越多，扫描时间越长。因此可以根据需要进行扫描深度的选择，这一维度由GitHub最近索引（Recently Indexed）排序的代码页决定，深度越深，检索的项目数量越多，反之亦然。深度选项和说明如下：
* Level 1：检索最近索引的前10页代码页；
* Level 2：检索最近索引的前20页代码页；
* Level 3：检索最近索引的前50页代码页；
* Level 4：检索最近索引的前70页代码页；
* Level 5：检索最近索引的前100页代码页；

深度选择与企业扫描周期性应该成正相关，深度选择小，则相应扫描的周期性也应当较小，如深度选择为Level 1，则相应的扫描周期基于企业情况可定为每天或每周，深度选择为Level 5，则相应的扫描周期可适当延长。例如，关键词“Google”最大（Level 5）可搜索两天前上传的项目代码，关键词“repoog”搜索结果则不足1页。

### 技术实现说明
项目配置文件Config.py中需要配置使用者的Github用户名、密码以及在个人设置中生成的Access Token值，其作用如下：
* 未登录Github进行代码搜索会因为请求速度过快（约10页代码结果页）而返回HTTP STATUE 429，即Too Many Requests的错误，因此需要登录后进行搜索；
* 在展现用户和项目信息实现中，采用GitHub API来实现，因此需要Access Token进行认证（未认证的请求频率限制为10次/分钟，认证的请求频率限制为30次/分钟）以增加请求频率限制（Rate Limit）；
* 在项目内关键词文件名和关键词内容扫描时未采用API，原因有两点：一是搜索代码的API频率限制很大（认证后30次/分钟）无法满足快速搜索；二是某些项目关键词的搜索结果项超过100条，而API在设置per_page参数后至多支持展现100条结果项；

***
## Sensitive info scan tool of Github
### Function introduction and design
GitPrey is a tool for searching sensitive information or data according to company name or key word something.The design mind is from searching sensitive data leakling in Github:
* Search code in file and path according to key word to get all related projects;
* Search code in every related project to find matching file or content in PATTERN_DB;
* Output all matching file information,project information and user information;

By the way, there is some missing file or mistake file with using Gitprey,the reason is:
* Only the default branch is considered by Github. In most cases, this will be the master branch.
* Only files smaller than 384 KB are searchable by Github.
* Github only make up to 1,000 results for each search.

Gitprey also provides the search level to adjust scanning deep, it's between Level 1 to Level 5:
* Level 1: Only search 10 pages in recently indexed code results.
* Level 2: Only search 20 pages in recently indexed code results.
* Level 3: Only search 50 pages in recently indexed code results.
* Level 4: Only search 70 pages in recently indexed code results.
* Level 5: Only search 100 pages in recently indexed code results.

You can modify the Level in Config.py.To search as quick as you can,you must configure your own Github account username and password to avoid 429 ERROR which is too many requests.

### Tech detail introduction
There are some hints to declare about technological details:
* Github API is not used in searching code,because its rate limit up to 30 times per minute,even if you authenticate by access token.
* The other reason is searching code only support up to 100 items in each searching.
* Only user information crawler used Github API,it's enough for scanning speed.

SmartOps Agent MVP 使用文档
1. 项目简介
SmartOps Agent 是一个面向 Spring Boot 后端项目 的智能故障诊断与代码链路分析工具。
它的目标是帮助 Java 后端开发者快速定位项目中的常见问题，例如：


接口返回 401 Unauthorized


JWT 校验失败


Spring Security 配置不完整


登录密码加密不匹配


SQL 查询性能较差


Redis 缓存未清理或缓存不一致


Controller、Service、Mapper 链路不清晰


项目代码结构复杂，难以快速理解业务流程


用户可以通过网页上传一个 Spring Boot 项目的 .zip 压缩包，或者直接粘贴运行日志，系统会自动分析项目代码和日志内容，并生成一份 Markdown 格式的诊断报告。

2. 项目解决的核心痛点
在真实 Java 后端开发中，很多问题不是只看一行报错就能解决的。
例如一个接口返回 401 Unauthorized，可能涉及：
前端请求头是否携带 token        ↓Spring Security 是否放行登录接口        ↓JWT Filter 是否正确解析 Authorization 请求头        ↓token 是否过期        ↓SecurityContext 是否成功写入用户信息        ↓Controller 接口是否需要权限
如果是初学者或者刚接手项目的开发者，往往会陷入下面的问题：
单个类能看懂，但是完整链路看不明白；报错信息能看到，但是不知道从哪里开始排查；Controller、Service、Mapper、Redis、Security 配置分散在不同文件里；项目越大，排查问题越慢；很多问题最后只能靠不断试错。
SmartOps Agent 的作用就是把这些分散的信息自动整理出来，帮助开发者快速得到：
这个问题可能是什么原因；它影响了哪些接口；应该看哪些代码文件；应该怎么修改；修改后怎么测试。

3. 项目核心能力
当前 MVP 版本主要包含以下功能：
3.1 项目 ZIP 上传分析
用户可以上传一个 Spring Boot 项目的压缩包，系统会自动扫描项目源码。
主要分析内容包括：


Controller 层接口


Service 层业务逻辑


Mapper / Repository 数据访问层


Entity 实体类


DTO / VO 请求响应对象


Spring Security 配置


JWT 相关过滤器


Redis 缓存注解


SQL 风险点


常见异常风险



3.2 日志粘贴分析
用户可以直接粘贴后端控制台日志，例如：
401 UnauthorizedBad credentialsNullPointerExceptionSQLSyntaxErrorExceptionRedisConnectionFailureExceptionAccessDeniedException
系统会根据日志关键词和上下文，判断可能的故障类型，并给出排查建议。

3.3 Controller-Service-Mapper 链路扫描
系统会尝试识别项目中的典型后端链路：
前端请求  ↓Controller 接收请求  ↓Service 处理业务逻辑  ↓Mapper / Repository 查询数据库  ↓返回结果给前端
例如登录链路可能被整理为：
POST /api/auth/login  ↓AuthController.login()  ↓AuthService.login()  ↓UserRepository.findByUsername()  ↓PasswordEncoder.matches()  ↓JwtService.generateToken()  ↓返回 token
这个功能可以帮助开发者快速理解一个接口从入口到数据库的完整流程。

3.4 Spring Security / JWT 检查
系统会检查项目中是否存在以下配置风险：


登录接口是否被放行


是否关闭 CSRF


是否配置无状态 Session


JWT Filter 是否加入 SecurityFilterChain


Authorization 请求头格式是否正确


是否存在 PasswordEncoder


密码校验逻辑是否合理


是否存在接口权限配置不完整问题


常见问题示例：
问题：访问接口返回 401可能原因：1. 前端没有携带 Authorization 请求头2. Authorization 没有加 Bearer 前缀3. JWT Filter 没有加入 Spring Security 过滤器链4. token 过期或签名密钥不一致5. 登录接口没有 permitAll()

3.5 SQL 风险检查
系统会扫描项目中的 SQL 或 Mapper 代码，识别常见性能风险。
例如：


查询没有分页


模糊查询使用 %keyword%


可能缺少索引


查询条件过少


存在 select *


存在循环查询风险


复杂查询没有拆分或优化


示例报告：
发现风险：SongMapper.xml 中存在模糊查询 like '%keyword%'，当数据量较大时可能导致索引失效。建议：1. 对搜索字段建立索引；2. 控制分页大小；3. 如果是歌曲搜索场景，可以考虑 Elasticsearch；4. 避免一次性查询过多数据。

3.6 Redis 缓存一致性检查
系统会检查项目中的缓存注解，例如：
@Cacheable@CacheEvict@CachePut
重点判断：


查询接口是否使用缓存


新增、修改、删除数据后是否清理缓存


缓存名称是否统一


是否可能出现缓存脏数据


示例：
@Cacheable(cacheNames = "public-playlists")public List<PlaylistResponse> listPublicPlaylists() {    ...}
如果新增歌单时没有清理缓存：
@CacheEvict(cacheNames = "public-playlists", allEntries = true)public PlaylistResponse createPlaylist(...) {    ...}
系统会提示：
public-playlists 被查询缓存使用，但在新增或修改公开歌单时没有发现明显的缓存清理逻辑，可能导致前端看不到最新数据。

3.7 Markdown 报告生成
分析完成后，系统会生成一份 Markdown 格式报告。
报告内容通常包括：
1. 项目基本信息2. 核心接口链路3. 日志诊断结果4. 安全配置检查5. SQL 风险检查6. Redis 缓存检查7. 可能的问题原因8. 修改建议9. 测试步骤10. 后续优化方向
这份报告可以直接复制到学习笔记、项目文档、面试材料或者 README 中。

4. 核心逻辑流
SmartOps Agent MVP 的整体逻辑如下：
用户上传项目 ZIP / 粘贴日志        ↓后端接收请求        ↓文件解压或日志解析        ↓代码扫描模块扫描 Java / XML / yml 文件        ↓识别 Controller、Service、Mapper、Entity、Security、Redis 等关键内容        ↓多个分析 Agent 分别处理不同问题        ↓汇总分析结果        ↓生成 Markdown 报告        ↓前端展示分析结果

5. 多 Agent 协作设计
当前 MVP 阶段采用的是 规则引擎 + 多 Agent 架构设计，不强依赖大模型 API。
每个 Agent 负责一个方向：
Agent 名称职责CodeScan Agent扫描项目目录，识别 Java、XML、YML 等文件RouteChain Agent分析 Controller、Service、Mapper 调用链路LogDiagnosis Agent根据日志内容判断异常类型SecurityCheck Agent检查 Spring Security、JWT、密码加密配置SqlRisk Agent检查 SQL 和 Mapper 中的性能风险RedisCache Agent检查 Redis 缓存注解和缓存清理逻辑Report Agent汇总所有分析结果，生成 Markdown 报告
完整协作流程：
CodeScan Agent    ↓RouteChain Agent    ↓SecurityCheck Agent    ↓SqlRisk Agent    ↓RedisCache Agent    ↓LogDiagnosis Agent    ↓Report Agent

6. 是否包含长链推理
项目包含长链推理思想。
它不是只根据一个关键词判断问题，而是尝试结合多个上下文信息进行综合判断。
例如分析 401 Unauthorized 时，不只是看到 401 就结束，而是会继续检查：
是否有 SecurityConfig        ↓是否配置 permitAll        ↓是否存在 JwtAuthenticationFilter        ↓是否读取 Authorization 请求头        ↓是否要求 Bearer 前缀        ↓是否设置 SecurityContext        ↓Controller 接口是否需要认证        ↓最终生成排查建议
再比如分析 Redis 缓存问题时，会检查：
是否存在 @Cacheable        ↓缓存名称是什么        ↓是否存在对应的 @CacheEvict        ↓新增 / 修改 / 删除接口是否清理缓存        ↓是否可能出现缓存脏数据
所以它的核心不是简单搜索代码，而是把一个问题还原成完整排查链路。

7. 环境要求
运行本项目需要准备以下环境：
环境版本建议JDK17 或以上Maven3.8 或以上操作系统Windows / macOS / Linux浏览器Chrome / EdgeIDEIntelliJ IDEA 推荐
检查 JDK：
java -version
检查 Maven：
mvn -v

8. 项目启动方式
解压项目后，进入项目根目录：
cd smartops-agent-mvp
启动项目：
mvn spring-boot:run
启动成功后，浏览器访问：
http://localhost:8088
如果是 Windows，也可以双击项目中的：
run.bat

9. 页面使用说明
9.1 首页
打开浏览器访问：
http://localhost:8088
首页通常包含两个核心区域：
1. 上传项目 ZIP 分析2. 粘贴日志分析

9.2 上传项目 ZIP
使用步骤：
1. 将你的 Spring Boot 项目压缩成 zip 文件；2. 打开 SmartOps Agent 首页；3. 点击“上传项目”；4. 选择项目 zip；5. 点击“开始分析”；6. 等待系统扫描；7. 查看生成的分析报告。
建议上传的项目结构：
your-project.zip  └── your-project      ├── pom.xml      ├── src      │   ├── main      │   │   ├── java      │   │   └── resources      │   └── test      └── README.md
不建议上传：
target/.idea/node_modules/.git/logs/
因为这些目录会让压缩包变大，也会影响扫描速度。

9.3 粘贴日志分析
使用步骤：
1. 复制 Spring Boot 控制台中的报错日志；2. 粘贴到日志分析输入框；3. 点击“分析日志”；4. 查看故障类型和排查建议。
示例日志：
2026-05-06 10:21:33 ERROR org.springframework.security.authentication.BadCredentialsException: Bad credentials
系统可能输出：
故障类型：登录认证失败可能原因：1. 用户名不存在；2. 密码加密方式不一致；3. 数据库存储的密码不是 BCrypt / Argon2 格式；4. PasswordEncoder.matches() 使用方式错误。建议：1. 检查数据库 password 字段；2. 检查登录逻辑是否使用 passwordEncoder.matches(rawPassword, encodedPassword)；3. 检查注册和登录是否使用同一种加密算法。

10. 典型使用场景
场景一：排查 401 Unauthorized
适用情况：
前端调用接口返回 401；登录成功后访问其他接口失败；请求头里有 token 但后端仍然拒绝。
使用方式：
1. 上传项目 zip；2. 粘贴浏览器 Network 中的请求信息或后端日志；3. 查看 Security / JWT 分析结果。
重点查看报告中的：
1. SecurityConfig 配置2. JWT Filter 配置3. permitAll 接口4. Authorization 请求头格式5. token 解析逻辑

场景二：排查登录密码错误
适用情况：
明明账号密码正确，但后端提示密码错误；数据库 password 字段是密文；项目刚从明文密码升级到 BCrypt / Argon2。
重点查看：
1. PasswordEncoder 是否配置；2. 登录时是否调用 matches；3. 注册时是否调用 encode；4. 数据库字段长度是否足够；5. 老密码是否需要平滑升级。
正确写法示例：
passwordEncoder.matches(rawPassword, encodedPassword)
错误写法示例：
passwordEncoder.encode(rawPassword).equals(encodedPassword)
因为 BCrypt 和 Argon2 每次加密结果都可能不同，不能直接重新加密后比较字符串。

场景三：排查 Redis 缓存不刷新
适用情况：
新增数据后，前端列表没有变化；数据库已经有新数据，但接口返回旧数据；重启项目后数据才正常。
重点查看：
1. 查询方法是否使用 @Cacheable；2. 新增 / 修改 / 删除方法是否使用 @CacheEvict；3. cacheNames 是否一致；4. 是否清理 allEntries；5. 是否存在多个缓存名混用。

场景四：排查慢 SQL
适用情况：
接口响应很慢；列表查询数据量一大就卡；搜索功能模糊查询很慢。
重点查看：
1. 是否分页；2. 是否 select *；3. 是否 like '%xxx%'；4. 是否缺少索引；5. 是否存在循环查库；6. 是否一次性查询大量数据。

场景五：快速理解陌生项目
适用情况：
刚拿到一个 Spring Boot 项目；不知道从哪里开始看；Controller、Service、Mapper 太多；想快速整理业务链路。
使用方式：
1. 上传项目 zip；2. 查看系统生成的接口链路；3. 按登录、查询、新增、修改、删除等业务分类阅读；4. 对照源码逐条理解。
建议阅读顺序：
Controller  ↓DTO / Request  ↓Service  ↓Mapper / Repository  ↓Entity  ↓数据库表  ↓异常处理  ↓权限和缓存

11. 报告字段说明
生成的 Markdown 报告一般包含以下部分：
11.1 项目概览
展示项目的基本信息，例如：
项目名称文件数量Controller 数量Service 数量Mapper / Repository 数量是否存在 Security 配置是否存在 Redis 缓存注解

11.2 接口链路
展示识别出的接口调用路径。
示例：
接口：POST /admin/login调用链路：AdminController.login()  -> AdminService.login()  -> AdminMapper.selectOne()  -> JwtUtil.generateToken()

11.3 安全风险
展示 Spring Security 和 JWT 相关问题。
示例：
风险等级：高问题：发现 JWT Filter，但未发现明显的 SecurityContext 设置逻辑。影响：token 即使有效，也可能无法让 Spring Security 识别当前用户。建议：在 JWT 校验成功后设置 UsernamePasswordAuthenticationToken 到 SecurityContextHolder。

11.4 SQL 风险
展示数据库访问层可能存在的性能问题。
示例：
风险等级：中问题：发现模糊查询 like '%keyword%'。影响：数据量较大时可能导致索引失效。建议：添加分页限制，必要时引入搜索引擎。

11.5 Redis 缓存风险
展示缓存读取和缓存清理是否匹配。
示例：
风险等级：中问题：发现 @Cacheable(cacheNames = "public-playlists")，但未发现对应的 @CacheEvict。影响：新增或修改公开歌单后，列表可能返回旧数据。建议：在新增、修改、删除公开歌单的方法上添加 @CacheEvict。

11.6 修复建议
系统会根据问题类型生成修复方向，例如：
1. 修改 SecurityConfig 放行登录接口；2. 前端请求头添加 Bearer token；3. 使用 PasswordEncoder.matches 校验密码；4. 为高频查询字段添加索引；5. 对缓存查询增加对应的缓存清理逻辑。

11.7 测试步骤
报告会给出修改后的测试建议。
示例：
1. 重新启动后端项目；2. 调用登录接口获取 token；3. 使用 Bearer token 访问受保护接口；4. 新增一条数据；5. 再次查询列表，确认缓存是否刷新；6. 查看控制台是否还有异常日志。

12. 项目目录说明
项目大致目录如下：
smartops-agent-mvp├── pom.xml├── README.md├── run.bat├── docs│   ├── architecture.md│   └── postman_collection.json├── src│   ├── main│   │   ├── java│   │   │   └── com.example.smartops│   │   │       ├── SmartOpsAgentApplication.java│   │   │       ├── controller│   │   │       ├── service│   │   │       ├── agent│   │   │       ├── scanner│   │   │       ├── analyzer│   │   │       ├── report│   │   │       └── model│   │   └── resources│   │       ├── application.yml│   │       ├── static│   │       └── templates│   └── test└── target
核心目录说明：
目录作用controller接收前端请求service组织业务流程agent多 Agent 分析模块scanner项目文件扫描analyzer安全、SQL、Redis、日志分析reportMarkdown 报告生成model分析结果模型static / templates前端页面资源

13. 常见问题
13.1 启动失败：端口被占用
如果提示：
Port 8088 was already in use
可以修改 application.yml：
server:  port: 8089
然后访问：
http://localhost:8089

13.2 Maven 下载依赖失败
可以检查 Maven 是否配置国内镜像。
在 Maven 的 settings.xml 中配置阿里云镜像：
<mirror>    <id>aliyunmaven</id>    <mirrorOf>*</mirrorOf>    <name>阿里云公共仓库</name>    <url>https://maven.aliyun.com/repository/public</url></mirror>

13.3 上传 ZIP 后没有识别到 Controller
可能原因：
1. ZIP 结构不正确；2. 项目源码不在 src/main/java 下；3. Controller 没有使用 @RestController 或 @Controller；4. 压缩包里套了太多层目录。
建议结构：
project.zip  └── project      ├── pom.xml      └── src/main/java

13.4 分析结果不够准确
当前 MVP 主要基于规则扫描，不是真正完整编译项目，所以存在局限。
例如：
1. 复杂动态调用无法完全识别；2. 反射调用无法准确分析；3. MyBatis 动态 SQL 只能做基础判断；4. 多模块项目可能需要后续增强；5. 暂时不能真正自动修改源码。
后续可以接入大模型 API，让它结合源码上下文做更智能的判断。

14. MVP 当前限制
当前版本主要是可运行演示版，重点是跑通核心逻辑。
限制包括：
1. 不会真正执行目标项目代码；2. 不会连接目标项目数据库；3. 不会真实调用 Redis；4. 不能保证识别所有复杂业务链路；5. 暂时不会自动提交 Git Patch；6. 对大型企业项目的分析能力有限；7. 多 Agent 目前是规则模块，不是完全的大模型 Agent。
但是它已经可以作为一个完整 MVP 展示：
上传项目  ↓扫描源码  ↓分析链路  ↓诊断问题  ↓生成报告

15. 后续升级方向
后续可以从 MVP 升级为真正高含金量的 Agent 项目。
15.1 接入大模型 API
让系统支持：
1. 长上下文源码理解；2. 复杂业务链路推理；3. 根据报错自动定位源码；4. 自动生成修复代码；5. 自动生成测试用例；6. 自动生成 Git Patch。

15.2 接入真实运行日志
可以支持上传：
application.logerror.logslow-sql.lognginx access.log
实现更真实的故障分析。

15.3 接入数据库表结构
可以支持上传：
schema.sql建表语句数据库 ER 图
进一步分析：
索引是否合理；字段类型是否合理；表关系是否清晰；SQL 是否命中索引。

15.4 接入 Git Diff
可以分析一次代码改动是否引入风险：
本次改动影响了哪些接口；是否改动了权限逻辑；是否影响缓存一致性；是否需要补充测试用例。

15.5 自动生成测试用例
可以根据接口链路生成：
Postman 测试用例JUnit 单元测试MockMvc 接口测试登录 token 测试流程

16. 推荐演示流程
如果你要向老师、面试官或者申请平台展示这个项目，可以按下面流程讲：
第一步：介绍痛点传统 Java 项目排查问题需要同时看日志、源码、数据库、缓存和权限配置，链路长、效率低。第二步：上传项目将一个 Spring Boot 项目打包成 zip 上传。第三步：展示链路分析系统自动识别 Controller、Service、Mapper，并生成接口调用链。第四步：展示故障诊断粘贴 401、密码错误、SQL 慢查询、Redis 缓存不一致等日志，系统输出问题原因。第五步：展示报告系统生成 Markdown 报告，包含风险点、修复建议和测试步骤。第六步：说明 Agent 架构项目包含日志分析 Agent、链路分析 Agent、安全检查 Agent、SQL 优化 Agent、Redis 缓存 Agent 和报告生成 Agent。第七步：说明后续规划后续接入大模型，实现自动生成修复代码、Git Patch 和测试用例。

17. 项目一句话总结
SmartOps Agent 是一个面向 Spring Boot 后端项目的智能故障诊断与链路分析平台，能够通过项目源码和运行日志，自动识别接口链路、安全配置、SQL 风险、Redis 缓存问题，并生成可执行的 Markdown 诊断报告。

18. 面试版项目介绍
可以这样说：
我做的是一个 SmartOps Agent，主要解决 Java 后端项目故障排查效率低的问题。传统排查一个 401、慢 SQL 或缓存不一致问题，需要手动看 Controller、Service、Mapper、SecurityConfig、JWT Filter、Redis 注解和日志，非常耗时。我的项目支持上传 Spring Boot 项目 zip 和粘贴日志，系统会自动扫描源码，识别 Controller-Service-Mapper 调用链，并通过多个 Agent 分别分析日志、安全配置、SQL 风险、Redis 缓存一致性，最后生成 Markdown 诊断报告。这个项目的核心亮点是长链路分析和多 Agent 协作。它不是简单根据关键词判断错误，而是会结合源码结构、请求链路、权限配置、数据库访问和缓存逻辑综合分析问题。当前 MVP 已经实现规则引擎版本，后续计划接入大模型，实现自动修复代码、生成 Git Patch 和自动补充测试用例。

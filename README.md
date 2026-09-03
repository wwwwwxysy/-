# 通用 Web 渗透测试流程工具包

> 版本：2026-09-03b（源自实际渗透测试项目经验沉淀，已做产品无关化处理——无目标/凭据/接口硬编码）
> 组成：`web-pentest/`（生成侧：六阶段流程 + 红线纪律 + 报告模板）、`pentest-report-check/`（校验侧：R1-R10 语义校验）
> 本版新增：启动信息完备性闸门（首次收到渗透请求先盘点登录信息，缺项先提醒再执行）

## 一、部署到新终端

1. **本包**：解压到 `~/.agents/skills/`（Windows 即 `C:\Users\<用户名>\.agents\skills\`），得到 `web-pentest/` 与 `pentest-report-check/` 两个 skill。skill 内路径均为用户相对路径，用户名不同不影响。
2. **ZCode CLI + 插件**：安装 ZCode CLI 后从官方 marketplace 装 `browser-use`、`computer-use`、`document-skills`（报告 docx/pdf 导出用）。
3. **工具栈**：docker 起 kali-pentest 容器（nmap/sqlmap/nuclei/whatweb/nikto/openssl 等）。
4. **网络**：新终端须能路由到目标产品的管理口；出口 IP 变化须先核对授权范围覆盖。

## 二、对新产品启动测试（流程入口：`web-pentest/SKILL.md`）

0. **启动信息完备性（第一动作，用户指令固化）**：首次收到渗透测试请求时先盘点四项——①产品地址（URL/IP+端口）②前端凭证（≥1 登录账号）③后台凭证（SSH/DB，决定灰盒）④前端多个不同权限的用户（越权/权限模型测试前提）。**缺项先提醒用户补齐并说明影响，不得默默降级开跑**；用户确认"先跑现有条件"才启动，缺口记入待补清单、报告如实标注。
1. **前置闸门（00-preflight，无授权不启动）**：书面授权文件 + IP/端口/应用边界 + 联系人 + 授权级别（黑盒/灰盒/白盒）逐项确认，摘要记入工作目录 `00-授权/授权范围.md`（**不含凭据**）。
2. **建工作目录**：按 `web-pentest/templates/workdir-layout.md` 的结构（00-进度状态 / 00-授权 / 01-测试方案 / 02-侦察报告 / 03-测试报告 / 04-补测清单 / 05-工具脚本 / 06-灰盒产物）。**每个产品一个独立工作目录**，不与历史产品混用。
3. **凭据文件**：工作目录之外单独存放（600 权限，如 `~/.<产品>_creds`），格式 `USER_<ROLE>=` / `PASS_<ROLE>=`。`scripts/auto-login.sh` 的登录接口/鉴权 header/凭据文件全部参数化传入——新产品登录协议不同（非 sha256 密码/非 token 字段）时扩展该脚本适配。
4. **六阶段流程**：preflight → 01-recon → 02-graybox → 03-blackbox → 04-confirm → 05-report；各阶段过 `05-工具脚本/phase-gate.sh`（A4 闸门），断点续传靠 `00-进度状态.md` + `resume-check.sh`。
5. **定稿**：报告按 `web-pentest/templates/report-template.md` 六要素 + 定级块三件套 + U 记录四要素写；**双轨终检**（fresh-context 结构审计 + pentest-report-check 语义校验）两项全过 + 终检后报告零修改（sha256 锚点核验）才能宣告终版——见 `web-pentest/flow/auto-decisions.md` §7.1.2。

## 三、纪律红线（skill 内已固化，此处提示重点）

- 破坏性操作（删数据/重启/关机/webshell 部署/RCE 实弹）按红线分级，未授权不执行
- 凭据不进工作目录、不进报告正文（占位符）；凭据文件 600 权限
- 审计发现即修；但定级争议/新漏洞验证/红线操作停下问用户
- 报告发布属红线，须用户确认

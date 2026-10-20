# WritingMaster · 写作大师（Skill 套装）

给 AI 用的写作外挂：**4 大赛道**（长篇网文·含言情模式 / 剧本 / 剧本杀 / 工具书）、
**13 条命令**、**82 张知识切片**、**29 个交付模板**，从开书、写章纲、写正文到防烂尾体检全流程包办。
零第三方依赖，纯 bash + Node，可离线跑。

本仓是 **Skill 套装**：给能读文件、能跑 bash 的 Agent 用（WorkBuddy / Claude Code / Cursor / Cline），
命令入口是 `bash scripts/wm-*.sh`，主路由是 `skills/writing-master/SKILL.md`。

> **MCP 套装不在这里**：它已经并进 `webnovel-mcp`（同一个服务里同时提供 `wm_*` 与 `webnovel_*` 两族工具）。
> 只想用 MCP、不装文件的话，去那边接一个服务就够了。

## 内含规模（v2.0.0 实测）

- 82 张知识切片（`knowledge/slices/`，带 `index.json` + `lock.json` 指纹锁）
- 29 个交付模板（`templates/`，覆盖 novel（含言情模式）/ screenplay / jubensha / toolbook 四赛道）
- 5 个技能（`skills/writing-master` 主路由 + `wm-novel` / `wm-screenplay` / `wm-jubensha` / `wm-toolbook`）
  - **v2.0 起言情不再单列赛道**：并入 `wm-novel` 作为「言情模式」（`--track novel --genre romance`）；
    旧的 `--track romance` 仍可用并自动映射，只是会打废弃提示；`romance-*` 知识切片一条没删
- 13 条命令（`scripts/wm-*.sh`）+ 1 个 Node 引擎（`scripts/wm.js`，零第三方依赖）
- 655 条自测 / 17 组用例（`tests/`，可删，删了不影响使用）
  - **跑法**：`bash tests/run.sh`。**注意**：v2.0 的 `tests/lib.sh` 仍把沙箱写死为 `/tmp/wm-sbx`，
    没有 `/tmp` 的环境（Android/Termux、只读根分区的容器）会连锁失败——那不代表套件坏了，
    换一台有 `/tmp` 的 Linux 跑就行（Windows 的 Git Bash 也可以，只是起进程慢，全量要 20 分钟以上）

## 装机

**skill 套装（满血版）**

```bash
cd suites/writing-master/writing-master-skill-suite
bash scripts/wm-init.sh --track novel --title 冒烟测试 --target-words 300000 --dir /tmp/wm-smoke
bash scripts/wm-doctor.sh --dir /tmp/wm-smoke
```

再把 `skills/writing-master/SKILL.md` 的路径写进 Agent 的系统提示，或让 Agent
在写作需求时先读该文件（详见包内 `README.md`）。

**mcp 套装**

```json
{
  "mcpServers": {
    "writing-master": {
      "command": "node",
      "args": ["<绝对路径>/suites/writing-master/writing-master-mcp-suite/scripts/mcp-server.mjs"]
    }
  }
}
```

握手自检：`echo '{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}' | node scripts/mcp-server.mjs`
应列出 13 个 `wm_` 工具（`wm_init`…`wm_verify`）。

## 与 webnovel-mcp 主体的边界（重要）

两套系统的**账本目录不同、状态不互通**，不要把同一本书同时挂在两套里：

| | 状态目录 | 命令/工具 |
|---|---|---|
| webnovel-mcp（主体） | `.webnovel/` | `webnovel_*` 12 个 MCP 工具 |
| WritingMaster（本套件） | `.wm/` | `wm_*` 13 个命令/工具 |

## 来源与处理说明

- 版本：**WritingMaster 2.0.0**（`mcp-server.mjs` 自报版本号）。双套装原包直入，**业务逻辑一行没改**；
  只做了下面几层「装进本仓」的适配
- 文件名修复：原始 zip 未置 UTF-8 标志，常规解压在 Windows 上会把 29 个中文模板名解成乱码
  （且乱码重名互相覆盖，只剩 12 个）。本目录是按 UTF-8 正确还原后的完整版，29 个模板齐全
- **ESM 作用域隔离**：两个套装根各自带 `package.json {"type":"commonjs"}`。原因：套件引擎
  `scripts/wm.js` 用 CommonJS 写就，放进任何 `"type": "module"` 的工程里都会被当 ESM 解释，
  直接 `ReferenceError: require is not defined` 全线熄火。这两个 `package.json` 把作用域切回来，
  **引擎与技能内容一行未改**；单独拷到别处使用也不受影响
- **判重门禁豁免**：套件的 `knowledge/source/novel-full.md` 与 26 张 `novel-*` 切片，
  内容就是本仓 `prompt/CREATIVE-PROMPT.md` 的整份/分段拷贝（同一份 19 节原文）。
  仓库的「规则正文全仓库只有一份」门禁会因此判红，已在 `scripts/check-rule-duplication.mjs`
  里把 `suites/` 列为豁免目录：套件是独立产品，它的知识库漂移由自己的
  `knowledge/lock.json` 指纹锁负责（`wm_verify` 每次都会校验），不归本仓单一原文原则管
- **运行产物不收录**：原包自带的 `tests/.tap.txt`、`tests/.run.log` 是跑测机器的日志
  （里面写着那台机器的绝对路径，进了公开仓就触发卫生门禁），只保留 `tests/REPORT.md`
- **两套用例都真跑过**：skill 套装与 mcp 套装各 17 组 / 655 条断言全绿（在一台有 `/tmp` 的机器上
  逐套装跑 `bash tests/run.sh`）。跑之前先看上面那条 `/tmp` 的说明，别把环境问题当成套件坏了
- 本套件不参与本仓 `node --test`（用例是 bash 写的，走 `tests/run.sh`）

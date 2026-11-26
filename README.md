# c-homework-vs2010（已归档）

> ⚠️ **本仓库已归档，内容已并入 [`vs-window-screenshot`](https://github.com/pixel-fable/vs-window-screenshot)。**

## 为什么归档

本技能描述的是「在 Windows 上用 VS2010 交 C 语言作业」的完整流程：
源码规范、编译、代码截图、运行截图、打包、验收清单。

问题在于：**这整条链路里有 80% 的内容是「怎么截窗口」**，与 `vs-window-screenshot` 高度重复，
且两边各自维护会不一致——事实上已经不一致了：

| 问题 | 本仓库（旧） | `vs-window-screenshot` v2（新） |
|---|---|---|
| `PrintWindow` 能否抓控制台 | 说「返回失败」 | **实测可以**（经典 `ConsoleWindowClass`） |
| 等窗口就绪 | 两段（句柄 + Splash） | **三段**（加了 `#32770` 错误对话框检查） |
| `cl` 编译日志编码 | 说按 GBK 解 | **实测是 UTF-8**（按 GBK 会出 `渚?.1.c`） |

其中「日志按 GBK 解码」这条会**把正常日志显示成乱码**，属于会误导人的错误。

所以两者合并成一份，避免同一件事有两个互相矛盾的版本。

## 请改用

👉 **[pixel-fable/vs-window-screenshot](https://github.com/pixel-fable/vs-window-screenshot)**

C 语言作业的完整流程在那边 `SKILL.md` 的**第六节**：
源码规范 → 编译验证 → 代码截图 → 运行截图 → 打包 → 验收清单。

## 保留了历史

本仓库内容不会被删除，`SKILL.md` 与 `README.md` 保持原样，供引用与追溯。
但**不再更新**，请不要在新项目里引用本仓库。

---

## 许可

MIT（沿用原内容）

---
name: c-homework-vs2010
description: 在 Windows 上用 VS2010/VS2022 完成 C 语言教材练习并交付「源码 + 代码截图 + 运行截图」压缩包。覆盖中文编码、VS2010 窗口截图、经典控制台抓取、回收站恢复、命名规范与验收自检。
---

# C 语言作业（VS2010/VS2022）+ 截图交付

## 何时用

用户给教材照片（例 X.Y / 同步练习 X-Y），要求在桌面产出可提交的作业包：
`.c` 源码 + IDE 代码截图 +（可选）运行结果截图，最后 `Compress-Archive` 打包。

## 铁律（血泪条款，按顺序执行）

1. **先看现状，再动手。** 列桌面、读用户已有的作业文件夹（如 `作业截图\`、`268104223徐志鹏\`），
   确认命名风格（`例1.9.c` 点号？`练习2-7.c` 横杠？）和配套图片类型。**照抄现有风格，不自创。**
2. **不动用户的文件夹结构与命名。** 需要放新内容时，**先问**：并入原文件夹，还是新建独立文件夹。
   默认：原文件夹保持原样，新作业放 `学号姓名_练习X.Y-...`。
3. **破坏性操作一律走回收站**，并在回复中明确说「已移入回收站，可恢复」。
   恢复用 Shell.Application 遍历 verbs，不可索引：
   ```powershell
   foreach ($x in $it.Verbs()) { if ("$($x.Name)" -replace '&','' -match '还原|Restore') { $x.DoIt(); break } }
   ```
   （`$verb[0].DoIt()` 会抛 `DISP_E_NOTACOLLECTION`。）
4. **每张截图必须当场读一遍**（read_image）。批量跑完再统一看 = 连废 3~4 轮。
5. **不要用 `Stop-Process conhost`**：会把托管自身会话的控制台一起杀掉，之后所有命令报 `0xE9`
   「管道的另一端上无任何进程」。只关自己启动的那个窗口（`PostMessage WM_CLOSE`）。
6. **不要删 `.exe`**：清临时文件时别 `Remove-Item *.exe`，否则后续「不是内部或外部命令」，
   截出来是空屏。
7. 用户说「在软件里编」就是让你少造轮子：能用 IDE 图形操作就别写一堆命令行胶水；
   但**编译本身用 `cl` 批量更快**，解释清楚「难的是截图不是编译」。

## 源码规范（与旧文件保持一致）

- **UTF-8 with BOM**（无 BOM 时 MSVC 按 GBK 解析，中文全乱）。
- 首行 `#define _CRT_SECURE_NO_WARNINGS`（压 C4996，VS2010 同样有效）。
- 4 空格缩进，`int main(void) {`，声明的变量放在块首（C89），`return 0;`。
- 教材原题要求「先定义 sex/age/height」之类，就按它的顺序声明。
- 写文件：
  ```powershell
  [IO.File]::WriteAllText($p, ($code -replace "`r`n","`n"), (New-Object System.Text.UTF8Encoding($true)))
  ```

## 编译与验证（VS2010 工具链）

```powershell
$vc10 = "D:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\vcvarsall.bat"
cmd /c "call ""$vc10"" >nul && cd /d ""$dir"" && cl /nologo /W3 ""$f"" /Fe:""$dir\r.exe"" /link /out:""$dir\r.exe"" >log.txt 2>&1"
```

- `/Fe:` 用**绝对路径**，并补 `/link /out:`，否则偶尔只出 `.obj` 不出 `.exe`。
- 捕获输出后按 GBK 解码：`[Text.Encoding]::GetEncoding(936).GetString([IO.File]::ReadAllBytes(...))`。
- 期望「0 警告 0 错误」，日志里只应出现源文件名。

## 代码截图（VS2010 窗口）

1. `Stop-Process VCExpress` → `Start-Process VCExpress.exe "<绝对路径.c>"`。
2. 轮询 `MainWindowHandle -ne 0`；再等 **`VSSplash` 类窗口消失**（枚举窗口按类名）。
3. 用 `PrintWindow(h, hdc, 0)` 抓整窗（VS2010 可用；VS2022 用 `2`）。
   `PrintWindow` 不依赖前台，不会抢焦点，也不怕桌宠遮挡。
4. 裁剪成 `3098x1743`（4K 满窗比例），保留标题栏/菜单，代码靠左、右侧留白 —— 与 `例1.9.png` 一致。
5. 代码行多的文件（>19 行）先 `Ctrl+Home` 回顶，或把裁剪高度放到 `1743` 以上。

## 运行截图（经典控制台）

Windows 11 默认把控制台委派给 Windows Terminal（`CASCADIA_HOSTING_WINDOW_CLASS`），
`PrintWindow` 抓不到、窗口标题不稳定。**用 `conhost.exe` 起经典 `ConsoleWindowClass`**：

```powershell
$before = [One4]::Consoles()          # 先快照已有控制台句柄
Start-Process "C:\Windows\System32\conhost.exe" -ArgumentList "cmd.exe /k chcp 936 >nul & $exe" -WorkingDirectory $dir
# 轮询：只认 $before 里没有的那个新句柄 → 确定性定位，避免抓到历次残留窗口
```

然后：
1. `SetWindowPos(h, -1, 80, 80, 1300, 700, SWP_SHOWWINDOW)` 置顶固定位置。
2. **点一下窗口内**（SetCursorPos + mouse_event 左键下/上）拿焦点，否则 SendKeys 打到别处。
3. 有输入的程序：`(New-Object -ComObject WScript.Shell).SendKeys("12345{ENTER}")`；
   无输入的程序用 `cmd /k` 保窗（`/c` 会瞬间关窗）。
4. `CopyFromScreen` 抓窗口矩形（`PrintWindow` 对控制台返回失败）。
5. 裁剪：`x=0, y=54`，两行输出 `w=1095,h=84`；单行 `h=38`。
   **切掉标题栏、命令行（含 `C:\Users\...` 路径）与后续提示符**，只留程序输出。

## 编码陷阱（PowerShell 5.1）

- 脚本文件里的**中文会被按 GBK 误读**，导致语法错或文件名乱码（`raw_缁冧範...png`）。
  → 脚本正文**只用 ASCII**；中文文件名由**父进程**（编码正确的 pwsh）在事后重命名。
- 脚本存成 UTF-8 **with BOM**。
- 参数传空字符串会被吞掉导致后续参数错位 → 用占位符 `NONE` 代替空值。

## 交付与打包

```
学号姓名_练习X.Y\
  例2.18.c        例2.18.png        例2.18-运行.png
  练习2.9-3.c     练习2.9-3.png     练习2.9-3-运行.png
```
```powershell
Compress-Archive -LiteralPath $folder -DestinationPath "$desk\学号姓名_练习X.Y.zip" -CompressionLevel Optimal
```
打包后核对：文件夹文件数 == ZIP 条目数。

## 验收清单（缺一不可）

- [ ] 每个 `.c` 在 VS2010 下 0 警告 0 错误
- [ ] 每个程序跑出预期结果（数值要对，如 5050 / 26 / 105 与 14）
- [ ] 代码截图：能看到文件名标签页 + 完整代码（末行 `}` 未被切）
- [ ] 运行截图：有输入回显 + 结果，**无路径、无标题栏、无残留窗口**
- [ ] 命名与用户既有风格一致（点号 / 横杠 / 前缀）
- [ ] 原有文件未被覆盖（必要时先在回收站确认可恢复）
- [ ] ZIP 解压后是单层文件夹

## 常见错误对照表

| 现象 | 根因 | 处置 |
|---|---|---|
| 截到 DSH/桌面而不是控制台 | 按类名找窗口，抓到历次残留 | 句柄差集定位 + 先清理残留 |
| 只有第一行提示、无输入 | 控制台丢焦点 | 每次截图前点击窗口 |
| `不是内部或外部命令` | 我清理时删了 `.exe` | 重编，别删产物 |
| `0xE9` 管道错误 | 杀了 conhost | 只 WM_CLOSE 自己那个窗口 |
| 文件名乱码 | 脚本中文被 GBK 读 | ASCII 脚本 + 父进程改名 |
| 只出 `.obj` 不出 `.exe` | `/Fe` 相对路径 | 绝对路径 + `/link /out:` |
| 中文乱码 | 源码非 UTF-8 BOM | `UTF8Encoding($true)` |

# 更新记录

## 2.0.2

动作从 26 个扩到 42 个，补的都是每台 iPhone 都有、无需另装 App 的通用动作。

- **天气**：`weather.forecast` 查预报＋`weather.detail` 取字段（两段式），
  「下雨提醒我带伞」现在能直接做了
- **系统控制**（新分组）：`flashlight` 手电筒、`airplanemode.set` 飞行模式、
  `wifi.set` 无线局域网、`bluetooth.set` 蓝牙、`lockscreen` 锁屏、
  `appearance` 浅色/深色
- **设备状态**：`getipaddress` 取当前 IP、`getdevicedetails` 取设备详情、
  `focus.get` 读当前专注模式
- **数据读写**：`downloadurl` 获取 URL 内容（可 GET/POST，常用于调家里服务）、
  `calculateexpression` 计算表达式、`setvariable` 设定变量
- **媒体**：`getcurrentsong` 取当前歌曲、`savetocameraroll` 保存到照片
- 新增的 16 个都标了「未真机验证」，只在编译层面验过结构；
  本机专用的动作（比如要另装 App 的）一律不往公开包里放

## 2.0.1

安全修复（ClawHub 安全审核指出，2.0.0 的问题）。

- **局域网传文件不再暴露私钥**：交付说明改成只服务 `out/` 输出目录
  （`python3 -m http.server 8000 -d out/`），并明确警告绝不要从技能目录
  起服务——`certs/` 里有 CA 私钥和签名私钥，从根目录起服务等于挂到
  局域网上任人下载
- **子进程不再继承完整环境**：`sc` 传给 gen-certs.sh / sign.sh 的环境变量
  改为最小集（PATH、HOME、语言、临时目录、证书目录及 SHORTCUT_* 覆盖项），
  父进程环境里的其他敏感信息不再透传给子进程和签名器
- **签名只签刚生成的文件**：`sc build` 在调签名器之前校验输出路径——
  必须是刚编译出的真实文件、不能在证书目录里、签名产物也不能写进证书目录
- **签名器可验证**：新增 `bin/shortcut-sign.sha256`，`sc doctor` 和每次
  `sc build` 前都会核对签名器哈希，对不上就拒绝签名并提示

## 2.0.0

从「帮你签名」升级成「说需求出文件」。

- **配方 DSL**：一份 JSON 描述动作序列，`if` / `repeat` / `repeat_each` / `menu` 四种控制流，
  7 个变量插值（`{clipboard}` `{now}` `{input}` `{prev}` `{battery}` `{location}` `{repeat_item}`）
- **26 个动作注册表**：按分组列出，参数键在构建期校验，写错当场报错并给出合法键
- **`sc adopt`**：用户发来 iCloud 链接或 `.shortcut` 文件，自动提取 CA 名建证书；
  与本机证书逐字节比对，能认出「同一张」还是「另一套」
- **`sc setup`**：一条命令装依赖、体检，装不动的直接给命令，不假装成功
- **平台自检**：签名器是 Linux x86-64 预编译二进制，非该平台会在体检和构建时明确说明，
  而不是抛 `Exec format error`
- **`references/actions.md`**：从同一份注册表生成的动作参数手册
- **修 `sc verify`**：原先喂未签名文件会触发签名器段错误（139），且被当成「通过」。
  现在先查文件头，未签名直接判定失败并说明原因

## 1.0.0

签名链路跑通：自建 CA（ECDSA P-256）+ 苹果 OID 扩展 + X9.63 裸私钥（97 字节）
+ AEA1 签名 + 反解校验动作数。手机装描述文件并完全信任后可导入。

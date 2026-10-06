<!-- BILINGUAL-EN-ZH -->

# Lutron Smart Bridges: Light and Shade Control / Lutron 智能桥接器：灯光与窗帘控制

**Last verified:** 2026-09-14 on a Lutron Smart Bridge 2 (firmware 8.28.0, non-Pro)

**最近验证：** 2026-09-14，验证环境为 Lutron Smart Bridge 2（固件 8.28.0，非 Pro 版）

Use this guide when fresh discovery identifies a Lutron Smart Bridge with  
HomeKit/HAP and the user asks to control lights or shades.

当全新发现识别出支持 HomeKit/HAP 的 Lutron Smart Bridge，且用户要求控制灯光或窗帘时，请使用本指南。

## Recommended Path / 推荐路径

1. Re-discover the bridge and use its advertised HAP endpoint through the Home  
   Link CONNECT proxy. Do not assume LIP/telnet or web access.

   重新发现桥接器，并通过 Home Link CONNECT 代理使用其公布的 HAP 端点。不要假设可以使用 LIP/telnet 或 Web 访问。
2. Use a maintained HAP client. If discovery advertises `ff=1`, select
   `PairSetupWithAuth`; plain `PairSetup` can fail at M2 with
   `kTLVError_Authentication`. Keep the entire pair-setup exchange on one
   persistent proxied TCP connection.

   使用持续维护的 HAP 客户端。如果发现结果公布了 `ff=1`，应选择 `PairSetupWithAuth`；普通的 `PairSetup` 可能在 M2 阶段失败并返回 `kTLVError_Authentication`。将整个配对设置交互保持在同一条持久的代理 TCP 连接上。
3. Pairing requires the eight-digit HomeKit setup code. Persist pairing material
   through approved credential storage; if file-backed, require mode `0600`.

   配对需要八位数的 HomeKit 设置码。将配对材料保存到经批准的凭据存储中；如果使用文件存储，权限必须为 `0600`。
4. For control, open an encrypted HAP session through the proxy, enumerate
   accessories, and read or write characteristics. Retry failed batch writes per
   device.

   控制时，通过代理打开加密的 HAP 会话，枚举配件，并读取或写入特征值。批量写入失败时按设备逐个重试。
5. HAP may expose generic device names instead of Lutron app room names. If the
   user wants room mapping, identify devices one at a time by toggling them.

   HAP 可能暴露通用的设备名称，而非 Lutron 应用中的房间名称。如果用户想要房间映射，请通过逐个开关设备的方式来识别它们。

【评论】"只使用发现公布的 HAP 端点、不假设 LIP/telnet 可用"属于规避旧式未加密控制通道的安全约束；配对材料要求 `0600` 文件权限则是本地凭据存储的常见加固做法。

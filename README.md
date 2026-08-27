# Workflow Update Documentation
### 1. Schedule Time Adjustment
* 將原有的 Cron 排程由 `20 23 * * *` 修改為 `0 18 * * *` (UTC)。
* 對應北京/香港時間 (UTC+8) 為每日凌晨 02:00 自動執行。

### 2. Log Output & Extraction
* 將 `node main.js` 的標準與錯誤輸出重定向至 `log.txt`。
* 透過 `awk` 指令，精準擷取 `log.txt` 中自 `[Genshin] Start` 至 `[Genshin] Sign-in completed` 區段的內容，並另存為 `genshin.log`，藉此過濾掉崩壞：星穹鐵道與雲遊戲等無關資訊。

### 3. Status Evaluation Logic
利用 `grep` 指令分析 `genshin.log` 內的關鍵字，判斷實際簽到狀態，並賦予對應的 Embed 邊框顏色與標記 (Tag) 條件：
* **Already Signed In (重複簽到)**: 偵測 `已签到`, `already`, `-5003`。顏色設為黃色 (16776960)，不觸發標記。
* **Login Status Lost (Cookie 失效)**: 偵測 `失效`, `not logged in`, `-100`。顏色設為紅色 (16711680)，觸發標記。
* **Sign-in Successful (簽到成功)**: 偵測 `OK`, `Sign-in successful`, `success`。顏色設為藍色 (5793266)，不觸發標記。
* **Unknown Status (未知狀態)**: 若皆不符合上述條件，顏色設為灰色 (8421504)，並觸發標記。

### 4. Discord Webhook Integration (Embed Format)
* 引入 `DISCORD_WEBHOOK` 與 `DISCORD_USER_ID` 變數。
* 使用 `jq` 工具動態生成 JSON 格式的 Payload。
* 將發送者名稱 (`username`) 自訂為 `原神小幫手`。
* 捨棄純文字通知，改採 Embed 格式美化排版，並將 `genshin.log` 的原始日誌字串，以 Markdown 程式碼區塊 (````text`) 的形式，附加於 Embed 的 `description` 欄位底部。
* 透過 `curl` 攜帶 JSON 檔案發送 POST 請求至 Discord。

## Required Environment Variables (Secrets)
除原專案必備之 `MYS_COOKIES` 等變數外，此更新要求引入以下配置：
* `DISCORD_WEBHOOK`: 存放於 GitHub Secrets 的 Discord 頻道 Webhook URL。
* `DISCORD_USER_ID`: 接收失敗/異常通知標記的 Discord 使用者 ID (目前以環境變數形式寫死於 YAML 中)。

* 

# 米游社、米家云游戏签到脚本 (Node.js 版) - 支持多账号 (崩铁 & 原神)

## 简介
本项目是一个用于自动签到米游社、云崩铁、云原神的 Node.js 脚本。支持多账号，并可本地部署或通过 GitHub Actions 自动运行。

## 功能特性
- **多账号支持**: 同时支持多个账号的签到。
- **本地部署**: 可以在本地或服务器上运行脚本。
- **GitHub Actions 部署**: 可以通过 GitHub Actions 定期自动运行签到任务。

## 免责声明
本脚本仅供交流测试使用。因使用本脚本而产生的任何问题，作者概不负责。官方可能更改接口导致脚本失效，脚本失效会尽快更新，但不保证第一时间。请低调使用，如有不同意，请关闭并停止使用。

## 获取 Cookie 方式 - 米游社签到必需

1. **打开浏览器**:
   - 打开你的浏览器，进入无痕/隐身模式。
   
2. **登录米游社**:
   - 访问 [原神论坛](http://bbs.mihoyo.com/ys) 或 [崩铁论坛](http://bbs.mihoyo.com/sr)，二选一进行登录操作，原神崩铁Cookie通用。
   
3. **获取 Cookie**:
   - 按下键盘上的 `F12` 或右键点击页面选择“检查”，打开开发者工具。
   - 切换到“Console”选项卡，复制粘贴以下代码：
     ```js
     const cookie = document.cookie
      const ask = confirm('Cookie:' + cookie + '\n\nDo you want to copy the cookie to the clipboard?')
      if (ask == true) {
        copy(cookie)
        msg = cookie
      } else {
        msg = 'Cancel'
      }
     ```
   - 按下回车键，此时 Cookie 已经复制到你的剪贴板。

## 获取 Token 方式 - 云游戏签到必需

1. **打开浏览器**:
   - 打开你的浏览器，进入无痕/隐身模式。
   
2. **登录云游戏**:
   - 访问 [云原神](https://ys.mihoyo.com/cloud/#/) 和 [云崩铁](https://sr.mihoyo.com/cloud/#/)。原神崩铁Token不通用！
   
3. **获取 Token**:
   - 按下键盘上的 `F12` 或右键点击页面选择“检查”，打开开发者工具。
   - 切换到“Network”选项卡
   - 在XHR请求中找到一条 `Request Headers` 中含有 `x-rpc-combo_token` 字段的请求。
   - 选中并复制`x-rpc-combo_token`的值，此时 Token 已经复制到你的剪贴板。



## GitHub Actions 部署

你可以通过 GitHub Actions 定期自动运行签到任务。以下是配置步骤：

1. **Fork本仓库**

2. **配置 GitHub Secrets**:
   - 打开你的 Fork 仓库页面。
   - 点击 `Settings` -> `Secrets and variables` -> `Actions`。
   - 添加新的 Secret, 多个cookie/token用英文逗号(,)分隔，若cookie/token下有原神/崩铁角色会执行签到：
     - 名称: `MYS_COOKIES`
       值: `cookie1,cookie2,cookie3`
     - 名称: `GENSHIN_TOKENS`
       值: `ys-token1,ys-token2`
     - 名称: `STARRAIL_TOKENS`
       值: `sr-token1,sr-token2`

3. **创建 Workflow 文件**:
   - 在 `.github/workflows` 目录下创建一个新的 YAML 文件，例如 `main.yml`。

4. **YAML 配置示例**:

   ```yaml
    name: MiHoYo Sign-In Script

    on:
      schedule:
        - cron: '20 23 * * *' # 每天UTC 23:20, 对应北京时间7:20，实际运行时间有偏差。
      workflow_dispatch: # 允许手动触发

    jobs:
      build-and-run:
        runs-on: ubuntu-latest

        steps:
        - name: Checkout repository
          uses: actions/checkout@v3

        - name: Set up Node.js
          uses: actions/setup-node@v3
          with:
            node-version: '18'

        - name: Install dependencies
          run: npm install

        - name: Run main.js
          env:
            MYS_COOKIES: ${{ secrets.MYS_COOKIES }}
            GENSHIN_TOKENS: ${{ secrets.GENSHIN_TOKENS }}
            STARRAIL_TOKENS: ${{ secrets.STARRAIL_TOKENS }}
          run: node main.js
   ```

## 本地部署

1. **安装 Node.js**:
   - 确保你已安装 Node.js 版本大于 14.x。
   
2. **Clone项目**:
   ```sh
   git clone https://github.com/GildedFlames/MYS_Game_Singin.git
   ```
   
3. **安装依赖**:
   ```sh
   npm install
   ```

4. **在 /src/MYS/index.js getCookieConfig 方法中设置Cookie信息**:
    ```js
    const MYSCookies = 'cookie1,cookie2'
    ```

5. **在 /src/MihoyoCloud/index.js getTokenConfig 方法中设置Token信息**:
    ```js
    const genshinTokens = 'ys-token1,ys-token2'
    const StarRailTokens = 'sr-token1,sr-token2'
    ```

6. **运行脚本**:
   ```sh
   node main.js
   ```

## 更新日志

- **2025-02-08**: v1.0.0 初始发布，支持原神和崩铁多账号签到功能。
- **2025-09-17**: v1.1.0 增加云原神云崩铁签到功能。

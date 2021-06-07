# Zabbix 簡介

[TOC]

## 2.1 Zabbix 用戶

&emsp;&emsp;[Zabbix用戶](https://www.zabbix.com/users)在世界各地都有使用，根據官方資料顯示，Zabbix **單個服務器節點可以支持 10 萬台設備的監控**，**其每秒可以處理 5 萬次請求** (NVPS 在有歷史數據和觸發器的情況下)，在無歷史數據和觸發器情況下可達到單機 30 萬次。也因此 Zabbix 自身定位是中型企業和大型企業。

## 2.2 使用 Zabbix 具備基礎

&emsp;&emsp;Zabbix 學習可分為 3 個階段：

* **入門階段**：從未接觸監控系統，也不熟悉 Linux 操作系統，**此階段學習 Zabbix 安裝及基本配置**。
* **中級階段**：具備 Linux 基礎，熟悉 LAMP 和 LNMP 環境搭建、MySQL 資料庫、Shell Script，以及具有簡單閱讀英文能力，主要難點在於觸發器、數據庫調教和 API 使用。**此階段可以將 Zabbix 與其他系統進行集成連接**。
* **高級階段**：熟悉 PHP or C 語言，具備二次開發能力，可以修改源代碼，可以對 Zabbix 從代碼級別進行優化和擴展。**此階段一般能熟練掌握 Zabbix 各種功能，已經從使用階段到源碼級別的研究階段，因此主要是對編成能力的要求**。

## 2.3 Zabbix 介紹

&emsp;&emsp;Zabbix 是企業級的高度集成的開元監控系統，提供分散式監控解決方案，可以用來監控設備、服務等可用性和性能。
&emsp;&emsp;Zabbix SIA 是 Zabbix 技術團隊的核心，其運作是商業軟體開源，即軟體完全開源，供客戶免費使用，但若需要官方提供技術服務，則需透過收費的商業模式。

## 2.4 選擇 Zabbix 原因

&emsp;&emsp;對比同類產品，以下幾點選擇使用 Zabbix：

&emsp;&emsp;(1) 用戶可以對源碼進行任意修改和二次開發。Zabbix 採用 GNU General Public License (GPL) version 2 開源協議。
&emsp;&emsp;(2) 安裝和配置簡單。
&emsp;&emsp;(3) 搭建環境簡單，基於開元軟體建構平台，只需要 Linux、Apache/Nginx、MySQL/PostgreSQL/Oracle、PHP 即可。
&emsp;&emsp;(4) Zabbix-Agent 支持 Linux、UNIX、Windows、AIX、BSD 和 Solaris 監控，Zabbix-Server 和 Zabbix-Agent 採用 C 語言編寫，對系統資源占用非常少，數據收集性能快。
&emsp;&emsp;(5) 可將收集到的數據持久存儲到資料庫中，便於對監控數據的二次分析。
&emsp;&emsp;(6) 具有良好的擴展能力，可以自定義監控項和實現數據收集，幾乎能監控所有的數據。例如可以監控網站的訪問次數、UPS、和天氣溫度等等。
&emsp;&emsp;(7) 採用開園社區運作模式，有論壇、郵件列表、IM 即時溝通。
&emsp;&emsp;(8) 由 Zabbix 授權的公司提供商業服務支持。

## 2.5 Zabbix 版本

&emsp;&emsp;非 LTS 版本，均為實驗版本，每 6 個月發布一個新版本，對於生產環境建議使用 LTS 版本，可擁有長期更新支持服務。每 18 個月會提供 LTS 版本發布。

## 2.6 Zabbix 架構

&emsp;&emsp;Zabbix 採用客戶端/伺服器端模式，分散式架構採用客戶端/代理端/伺服器端模式。Zabbix-Server 將收集到的數據存儲到資料庫中，用前端 UI 展示給用戶。
&emsp;&emsp;Zabbix 收集數據不僅可以使用 Agent 方式，也可以使用其他方式如 SNMP、SSH、Telent、IPMI 等。

## 2.7 Zabbix 功能特性

&emsp;&emsp;Zabbix 常見商業監控軟體所具備功能有：主機性能監控、網路設備性能監控、資料庫性能監控、FTP 等通用協定監控、多種告警方式、詳細報表圖表繪製、分散式、可擴展能力、API 等。
&emsp;&emsp;**1. 數據收集**
&emsp;&emsp;&emsp;&emsp;1.1 可用性、性能檢測。
&emsp;&emsp;&emsp;&emsp;1.2 支持 Agent、SNMP (Trapping and Polling)、IPMI、JMX、SSH、Telnet。
&emsp;&emsp;&emsp;&emsp;1.3 自定義檢測。
&emsp;&emsp;&emsp;&emsp;1.4 自定義收集數據的頻率。
&emsp;&emsp;&emsp;&emsp;1.5 客戶端/代理端/伺服器端模式。
&emsp;&emsp;**2. 靈活的觸發器**
&emsp;&emsp;&emsp;&emsp;2.1 可以定義非常靈活的告警閾值和與多種告警相關聯的條件。
&emsp;&emsp;**3. 高度可訂製的告警**
&emsp;&emsp;&emsp;&emsp;3.1 發送通知，可訂製包括告警級別、動作升級、收件人和媒體類型。
&emsp;&emsp;&emsp;&emsp;3.2 通知可以使用全局宏變量 (Macro) 和自定義變量。
&emsp;&emsp;&emsp;&emsp;3.3 自動處理功能包括遠程命令的自動調用和執行。
&emsp;&emsp;**4. 即時繪圖功能**
&emsp;&emsp;&emsp;&emsp;4.1 監控項將數據即時繪製在圖形上。
&emsp;&emsp;**5. Web 監控能力**
&emsp;&emsp;&emsp;&emsp;5.1 Zabbix 可以模擬瀏覽器請求訪問一個網站，並檢查返回值和響應時間。
&emsp;&emsp;**6. 多種視覺化展示**
&emsp;&emsp;&emsp;&emsp;6.1 可以自定義監控的展示圖、將多種監控數據集中展示到一張圖上。
&emsp;&emsp;&emsp;&emsp;6.2 網路拓樸圖 (Map)。
&emsp;&emsp;&emsp;&emsp;6.3 自定義 Screens 和 Slide shows 可以將多種圖形集中展示。
&emsp;&emsp;&emsp;&emsp;6.4 報表功能。
&emsp;&emsp;&emsp;&emsp;6.5 資源使用情況的監控展示。
&emsp;&emsp;**7. 歷史數據的存儲**
&emsp;&emsp;&emsp;&emsp;7.1 將數據存儲在資料庫中。
&emsp;&emsp;&emsp;&emsp;7.2 歷史數據的存放週期可以配置。
&emsp;&emsp;&emsp;&emsp;7.3 定期刪除過期的歷史數據。
&emsp;&emsp;**8. 配置簡單**
&emsp;&emsp;&emsp;&emsp;8.1 配置只需兩步驟：(1) 新增設備，(2) 應用模板即可完成監控。
&emsp;&emsp;**9. 使用模板**
&emsp;&emsp;&emsp;&emsp;9.1 模板可以分組。
&emsp;&emsp;&emsp;&emsp;9.2 模板具有可繼承性。
&emsp;&emsp;**10. 網路發現**
&emsp;&emsp;&emsp;&emsp;10.1 支持自動發現網路設備和服務器 (可以透過配置自動發現服務規則實現)。
&emsp;&emsp;&emsp;&emsp;10.2 Agent 自動註冊。
&emsp;&emsp;&emsp;&emsp;10.3 支持用自動發現 (Low Level Discovery) 實現動態監控項的批量監控 (支持自定義)，內痔的自動發現包括文件系統、網路接口、SNMP OID，可訂製自動發現。
&emsp;&emsp;**11. 快速的訪問接口**
&emsp;&emsp;&emsp;&emsp;11.1 Web 頁面基於 PHP。
&emsp;&emsp;&emsp;&emsp;11.2 遠程訪問。
&emsp;&emsp;&emsp;&emsp;11.3 日誌審計。
&emsp;&emsp;**12. API 功能**
&emsp;&emsp;&emsp;&emsp;12.1 應用 API 功能可以方便的與其他系統結合，包括手機客戶端的使用。
&emsp;&emsp;**13. 系統權限**
&emsp;&emsp;&emsp;&emsp;13.1 不同的用戶展式監控的資源不同。
&emsp;&emsp;&emsp;&emsp;13.2 用戶身分驗證。
&emsp;&emsp;**14. 程式特性**
&emsp;&emsp;&emsp;&emsp;14.1 Zabbix-Server 和 Zabbix-Agent 使用 C 語言撰寫，性能高，記憶體用量少。
&emsp;&emsp;**15. 大型環境**
&emsp;&emsp;&emsp;&emsp;15.1 利用 Zabbix-Proxy 方式可以輕鬆建構遠程監控。
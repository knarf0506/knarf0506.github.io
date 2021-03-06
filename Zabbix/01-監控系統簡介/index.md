---
export_on_save:
  html: true
---

@import "..\..\Packages\bootstrap-5.0.1-dist\css\bootstrap.min.css"
@import "..\..\Packages\bootstrap-5.0.1-dist\js\bootstrap.min.js"
@import "..\..\Packages\fontawesome-free-5.15.3-web\css\all.css"
@import "..\..\Packages\fontawesome-free-5.15.3-web\js\all.js"

<nav class="navbar navbar-expand-lg navbar-light bg-light">
  <div class="container-fluid">
    <a class="navbar-brand" href="https://knarf0506.github.io/"><i class="fas fa-home"></i></a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navbarSupportedContent">
      <ul class="navbar-nav me-auto mb-2 mb-lg-0">
        <li class="nav-item">
          <a class="nav-link active" aria-current="page" href="#">Blog</a>
        </li>
      </ul>
    </div>
  </div>
</nav>

# 監控系統簡介

[TOC]

## 1.1 監控系統功能概述

&emsp;&emsp;監控 (Monitoring) 可以分為**監測**和**控制**。在計算機領域可以分為五種監控類型：

* 應用性能監控 (Application Performance Monitoring)
* 商務交易監控 (Business Transaction Monitoring)
* 網路性能監控 (Network Monitoring)
* 操作系統監控 (System Monitoring)
* 網路站點監控 (Website Monitoring)

## 1.2 監控系統實現原理

### 1.2.1 監控組成

&emsp;&emsp;監控系統分成：數據儲存分析告警展示 (伺服器端, **Server**)、數據收集 (客戶端, **Agent**)。

### 1.2.2 監控協議

&emsp;&emsp;數據收集協議可分為兩種：專用客戶端收集和公用協議收集 (**SNMP**、**IPMI**、**SSH** 和 **Telnet**)。

### 1.2.3 收集模式

&emsp;&emsp;監控系統數據收集可以分為**被動模式** (Server 端到 Client 端收集數據, **pull**) 和**主動模式** (Client 端主動上傳數據到 Server 端, **push**)。

### 1.2.4 監控項目

&emsp;&emsp;監控系統支持的常見的監控項目如下：

| 監控項目 | 監控內容 |
| :---- | :---- |
| 主機 | CPU、記憶體、硬碟剩餘空間/使用率、I/O、SWAP 使用率、系統啟動時間、進程數、負載 |
| 網卡 | Ping 響應時間及數據包發送的成功率、網卡流入/流出量和錯誤的數據包數 |
| 文件 | 文件大小、文件指紋 hash 值、匹配查詢、字串存在與否 |
| URL | 指定 URL 訪問過程中的返回碼、下載時間及文件大小、返回數據的匹配 |
| 應用程式 | 端口和內存使用率、CPU 使用率、服務狀態、請求數、併發連接數、消息隊列字串數、客戶端事務處理數 |
| 資料庫 | 資料表空間、由、會話數、事務數、死鎖數、緩衝池命中率、庫緩存命中率、當前連接數、進程的內存使用率 |
| 日誌 | 錯誤日誌匹配、特定字串匹配 |
| 硬體 | 溫度、風扇轉速、電壓、電源、主板控制器、硬碟陣列 |

### 1.2.5 代理架構

&emsp;&emsp;對於**大規模的監控環境**，被監控的節點越多且監控類型越多，監控產生的數據和網路連接開銷非常大，這時**數據收集**方式除了使用**主動模式**，還需要使用代理架構，通過**代理架構分散伺服器端性能開銷**。另外，代理架構還可以支持跨地域、跨網路的分散式監控。常見的代理架構，及 C/P/S (Client/Proxy/Server)，此處的 Client 等同於 Agent。

### 1.2.6 數據存儲

&emsp;&emsp;監控系統儲存數據會用以下方式：
&emsp;&emsp;(1) 本地：使用本地磁碟，基於文件方式存儲。
&emsp;&emsp;(2) 時序資料庫：如環狀資料庫 (Round Robin Database, RDB)，OpenTSDB、Graphite、InfluxDB、Promethus，比起用文件存儲，時序存儲更加高效。
&emsp;&emsp;(3) 資料庫管理系統 (Database Management System, DBMS)：常見 MySQL、Oracle、SQL Server，缺點是到達一定量會**降低讀/寫速度**。優化方法有三種，一是減少數據量；二是優化資料庫本身，調整配置參數，優化環境；三是使用分散式資料庫和資料庫集群技術方案。
&emsp;&emsp;(4) NoSQL：相較於 DBMS，單機的 QPS 通常較高，但本身不是為了監控系統設計，在數據結構存儲方面存在缺陷，故使用 NoSQL作為存儲較少。
&emsp;&emsp;(5) 列存資料庫：設計初衷為大數據所用，但由於部署、維運較為複雜，故一般監控系統也不會採用，這方便資料庫代表為 HBase。
&emsp;&emsp;(6) 全文搜索引擎資料庫：代表為 Elasticsearch，作為監控數據存儲有天然優勢，支持集群、分散式佈署、容災，並且集群能夠提供較高的性能。目前採用 Elasticsearch 代表作為 Elastic Stack，在 Zabbix 這方面 4.0 版本後也可以做存儲。

### 1.2.7 告警功能

&emsp;&emsp;監控系統的重要功能是根據設定的閾值進行告警，同時也要在發生故障時有一定的故障自動化處理功能，對於特殊的告警還需要具備告警的升級功能，將不同級別的告警分成不同梯度發送給不同的接收人。
&emsp;&emsp;過多的告警對於人是沒有幫助的，因此必須優化監控系統。在**觸發**和**發送告警**時，告警模組需要支持故障的有效匯報和集中匯報，盡量避免出現告警風暴，防止同一時間大量發送重複、類似的告警，及告警功能支持**對告警內容進行分析和自動處理，防止誤報、漏報及抖動**。以一個例子來說，當機房網路發生故障時，**用戶會收到每台設備的故障**，但是如果將告警聚合，我們會希望收到的訊息只有一則："某機房存在網路故障，受影響設備 IP 為 X.X.X.X，受影響業務為 X.X.X。
&emsp;&emsp;事後還須對告警訊息進行統計分析，以方便對系統的運行狀況進行分析統計，從而衡量系統的穩定性、可用性。通常使用 SLA 服務質量指標來衡量。

### 1.2.8 可擴展性

&emsp;&emsp;可擴展性是指監控系統本身具備良好的擴展能力，包括監控方式的擴展、監控能力的擴展、監控數據存儲的擴展、分散式的支持。對於告警，要求支持**多種方式，如簡訊、郵件、即時通訊和其他接口且具備可制定化能力**，可以對第三方告警提供可編程接口，例如將告警發送到專用的告警分析系統。

### 1.2.9 總結

&emsp;&emsp;上述最為重要的是對於告警，我們希望程序能夠自動處理，減少人工干預，讓程序自動修復，只在出現嚴重故障時、程序無法判斷時，才發送告警通知給管理人員處理。

## 1.3 監控系統開源產品

&emsp;&emsp;前述介紹通用的監控系統實現原理，接下來介紹現有的開源監控系統。
&emsp;&emsp;在監控系統中，開源的解決方案有**流量監控** (MRTG、Cacti、SmokePing)，**性能告警** (Nagios、Zabbix、Zenoss Core、Ganglia、Netdata)，**基於時序資料庫存儲數據的監控** (Graphite、OpenTSDB、InfluxDB、Prometheus、OpenFalcon)，**基於全文搜索引擎資料庫存儲數據監控** (Elastic Stack)。上述都各有特點，但具有共同特徵：收集數據、視覺化、告警以及故障自動處理。

### 1.3.1 Cacti

&emsp;&emsp;基於 PHP、MySQL、SNMP 和 RRDtool 開發的網路流量監測圖形分析工具。數據收集僅支持 SNMP 格式，透過 snmpget 指令來獲取監控數據，使用RRDtool 儲存歷史數據和繪圖，並提供數據和用戶管理功能。支持與 LDAP 結合進行用戶認證，同時也能自定義模板。對於網路設備監控。

### 1.3.2 Nagios

&emsp;&emsp;Nagios 是一款插件式的監控系統，可以監控服務的運行狀態和網路信息，並能監視所指定本地貨遠程主機參數以及服務，同時提供異常告警通知功能。Nagios 支援客戶端的數據收集，通過編寫客戶端插件，可以獲取各種監控數據，根據設定閾值進行告警，但大部分告警邏輯都是通過監控插件實現。

### 1.3.3 InfluxDB 套件

&emsp;&emsp;InfluxDB 並不是監控產品，而是一個開源分散式時序、時間和指標資料庫，使用 Go 語言開發，其功能式數據指標的存儲，不提供告警分析和數據收集功能，僅僅是一個資料庫，支持 SQL 語言查詢，其語法和 MySQL 相似。隨 InfluxDB 時序資料庫功能不斷完善，官方也提供做為監控系統的相關套件，分別是 Telegraf (收集數據的客戶端)、InfluxDB (存儲監控數據)、Kapacitor (告警分析)、Chronograf (監控數據視覺化)。


### 1.3.4 Prometheus

&emsp;&emsp;Prometheus 是一套開源系統監控報警框架，能夠支持 Docker、Kubernetes。提供靈活的查詢語法 (PromQL)、告警策略、易於管理、方便擴展，其數據收集使用私有客戶端，傳輸協議使用 HTTP，資料庫使用私有資料庫。

### 1.3.5 OpenFalcon

&emsp;&emsp;OpenFalcon 是一個企業級、高可用、可擴展的開元監控解決方案。由多個組件構成：數據收集、數據傳輸、數據存儲、告警分析合圖形展示等等，數據收集使用私有客戶端，傳輸協議為私有的，資料存儲支持 OpenTSDB 和 RRD 兩種方式。

### 1.3.6 Netdata

&emsp;&emsp;Netdata 是 Linux 系統實時性能監測工具，以 Web 視覺化方式展示系統及應用程序的實時運行狀態 (CPU、記憶體、硬碟輸入/輸出、網路等性能數據)。
&emsp;&emsp;Netdata 數據收集為私有客戶端，數據存儲為司也格式，也支持多種時間序列後端服務，比如 Graphite、OpenTSDB、Prometheus、JSON document DBs，並支持告警功能，在單機監控中具有其他監控系統所不具備的優勢。

### 1.3.7 Elastic Stack

&emsp;&emsp;Elastic Stack 主要由 Elasticsearch、Logstash、Kibana。Elasticsearch 是實時全文搜索和分析引擎，提供收集、分析、存儲數據三大功能，是一套開放 REST 和 Java API 等結構，提供高校搜索效能，並可擴展分散式系統，建構在 Apache Lucene 搜索引擎上。
&emsp;&emsp;Logstash 是一個用來收集、分析、過濾日誌的工具。幾乎支持所有類型日誌，包括系統日誌、錯誤日誌和自定義應用程序日誌。他可以從許多來源接收日誌，包括 syslog、消息傳遞 (如 RabbitMQ)、JMX。他能夠以多種方式輸出數據，包括 Email、WebSocket、和Elasticsearch。
&emsp;&emsp;Kibana 是基於 Web 的圖形介面，用於搜索、分析和視覺化存儲 Elasticsearch 指標中的日誌數據。他利用 Elasticsearch 的 REST 接口來檢索數據，不僅允許用創建自己數據的訂製儀表板視圖，還允許他們以特殊的方式查詢和過濾數據。
&emsp;&emsp;其他針對日誌收集開發了專用的客戶端 Beats 系列產品，包括 Filebeat (日誌文件)、Metricbeat (指標)、Packetbeat (網路數據)、Winlogbeat (Windows 事件日誌)、Auditbeat (審計數據)、Heartbeat (運行時間監控)。

### 1.3.8 Zabbix

&emsp;&emsp;Zabbix 是企業級分散式監控系統，支援多種收集數據方式，有專用的 Agent (代理)，也支持 SNMP、IPMI、JMX、Telnet、SSH 等多種協議，他將收集到的數據存放到資料庫中，可以支持 MySQL、Oracle、PostgreSQL、SQLite、Elasticsearch。達到條件觸發告警，並支持對告警數據的分析統計。



<div>
    <a style="float:left" href=../index.html>上一頁</a>
    <a style="float:right" href=../02-Zabbix%20簡介/index.html 簡介>下一頁</a>
<div>
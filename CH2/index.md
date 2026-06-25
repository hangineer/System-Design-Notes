# [粗略估算](https://learning-guide.gitbook.io/system-design-interview/xi-tong-she-ji-mian-shi-nei-mu-zhi-nan-di-yi-juan/chapter-02-back-of-the-envelope-estimation)
### 學習重點
- 了解估算的指標和數據

Jeff Dean 說：「粗略估算是使用一系列常見的性能數據進行估算，以便對不同的設計能有一個良好的了解」
> 小科普：Jeff Dean 是傳奇軟體工程師，也是 Google 唯二到達 L11 的工程師。有興趣了解其故事的人，非常推薦《The Friendship That Made Google Huge》一文 (連結)

## 2 的次方
- 1 個 Byte 等於 8 個 Bit
好的，已為您將上述內容整理成 Markdown 表格格式：


| Power ($2$ 的次方) | 精確值 (Bytes) | Approximate value | Full name | Short name |
| --- | --- | --- | --- | --- |
| 10 ($2^{10}$) | 1,024 | 1 Thousand(一千) | 1 Kilobyte | 1 KB |
| 20 ($2^{20}$) | 1,048,576 | 1 Million(一百萬) | 1 Megabyte | 1 MB |
| 30 ($2^{30}$) | 1,073,741,824 | 1 Billion(十億) | 1 Gigabyte | 1 GB |
| 40 ($2^{40}$) | 1,099,511,627,776 | 1 Trillion(一兆)  | 1 Terabyte | 1 TB |
| 50 ($2^{50}$) | 1,125,899,906,842,624 | 1 Quadrillion(一千兆) | 1 Petabyte | 1 PB |

## 軟體工程師都應該了解的延遲(Latency)數據
以下是 Jeff Dean 2010 提到每個做系統設計的工程師都應該要知道的數字。現在一些數字已經過時。然而，這些數字仍然應該能夠讓我們了解不同電腦操作的速度和慢速。

- L1 快取參考 0.5 奈秒
- 分支錯誤預測 5 奈秒
- L2 快取參考 7 奈秒
- 互斥鎖定/解鎖 100 奈秒
- 主記憶體參考 100 奈秒
- 使用 Zippy 壓縮 1K 位元組 10,000 奈秒
- 通過 1 Gbps 網路傳送 2K 位元組 20,000 奈秒
- 從記憶體順序讀取 1 MB 250,000 奈秒
- 在同一資料中心內的來回行程 500,000 奈秒
- 磁碟尋道 10,000,000 奈秒
- 從網路順序讀取 1 MB 10,000,000 奈秒
- 從磁碟順序讀取 1 MB 30,000,000 奈秒
- 發送封包從加州到荷蘭再回加州 150,000,000 奈秒

以上數字得出以下結論：
- 記憶體速度快，但磁碟速度慢，盡可能避免磁碟尋道（Disk seek）
- 善用壓縮，簡單的壓縮演算法速度快，在將資料透過網路傳送出去之前，盡可能先對資料進行壓縮以節省頻寬
- 網路傳輸有代價，跨區域傳輸一定會有所延遲

### 前端工程師要了解的數字
- 在 60fps 裝置下，每幀需要 16 毫秒 (超過就代表會掉幀)
- 在 60fps 裝置下，會需要額外考量 5 - 10 毫秒，是裝置的額外處理時間
- 在同個區域中，伺服器到資料庫再回來，約會是 10 毫秒
- 同國家中，從一個城市到另一個再回來，平均傳輸要 33 毫秒
- 從地球一端到另一端再回來，在非光纖網路平均傳輸要 300 毫秒
- 解析 1MB 的 HTML 約需要 120 毫秒
- 解析 1MB 的 CSS 約需要 100 毫秒
- 解析 1MB 的 JavaScript 約需要 150 毫秒

## 可用性數據
大多數服務的可用性介於 99%~100% 之間
### 服務水準協議（Service-Level Agreement, SLA）
其概述正常執行時間、交付時間、回應時間和解決時間等指標，SLA 一般由客戶和服務供應商達成協議，同公司內的不同業務單位也可以相互制訂 SLA。
[AWS - 什麼是 SLA?](https://aws.amazon.com/tw/what-is/service-level-agreement/)

Amazon、Google 和 Microsoft 將它們的 SLA 設定在 99.9% 或更高
<img width="809" height="416" alt="image" src="https://github.com/user-attachments/assets/859179c1-323a-4d20-aec1-25c8301e0dc9" />

(註：Downtime 停機時間)


## 範例：估算 X 的查詢量和儲存需求
開始前，先確認「查詢量」是哪一種
| 類型    | 例子                               | 估算方式                   |
| ----- | -------------------------------- | ---------------------- |
| 寫入量   | 發 tweet / reply / retweet / like | posts per day ÷ 86400  |
| 讀取量   | 首頁 timeline、個人頁、通知、留言串           | DAU × 每人每天讀取次數 ÷ 86400 |
| 搜尋查詢量 | 搜尋關鍵字、hashtag                 | DAU × 每人每天搜尋次數 ÷ 86400 |

（註：86400 是 一天的秒數）

👉 第一步：設定假設 (Assumptions)：
以寫入量來說，我們假設：
- DAU(Daily Active User) = 2 億
- 每天 5 億 tweets/posts
[X (Twitter) Statistics: How Many People Use X?](https://backlinko.com/twitter-users)

👉 第二步：估算每秒查詢量 (QPS - Queries Per Second)
> QPS = 總查詢數 / 完成總時間（秒）
QPS = 500,000,000 / 86,400 ≈ 5,787 writes/sec
但 Twitter 尖峰很誇張，不能只設計 6K QPS，尖峰 = 平均 × 10 ~ 25 倍
寫入容量約 60K ~ 150K writes/sec （這個數值跟歷史尖峰 143K TPS 很接近）

👉 第三步：估算儲存需求 (Storage Requirement)
先只算純文字 post，不算圖片、影片

假設每筆 tweet 儲存這些資訊：
```
- tweet_id: 8 bytes
- user_id: 8 bytes
- timestamp: 8 bytes
- text: 280 chars，UTF-8 估 300~600 bytes
- metadata: reply_to、retweet_id、lang、visibility、counters...
```
面試時可以大概抓
- 每筆 tweet object ≈ 1 KB
- 每天原始資料量：500M × 1 KB = 500 GB/day


如果要算多媒體，像是圖片、影片
```
- 500M posts/day
- 15% 有圖片
- 平均圖片壓縮後 1 MB
- 2% posts 有影片
- 平均影片 10 MB
```
圖片原始量：500M × 15% × 1 MB = 75 TB/day
影片原始量：500M × 2% × 10 MB = 100 TB/day

假設：
- DAU: 200M
- Posts per day: 500M
- 每人每天讀取 50~100 次
- 每筆 tweet object: 1 KB
- Replication factor: 3

總結：
| 項目                  |             估算 |
| ------------------- | -----------------: |
| 平均發文寫入 QPS          |         約 5.8K/sec |
| 寫入尖峰                |   約 60K ~ 150K/sec |
| Tweet object 原始儲存   |       約 500 GB/day |
| Media               |     可能數十到數百 TB/day |


> Ｑ：什麼時候該用 DAU？什麼時候該用 MAU？
估 QPS / 流量 / 系統負載 → 用 DAU
估總使用者規模 / 帳號資料 / 長期儲存 → 用 MAU


### 面試 Tip
粗略估計更注重過程而非結果，解決問題比得到準確結果更為重要。以下是一些估算時的原則：
1. 使用整數和近似值來簡化問題。沒有必要花費寶貴的時間來解決複雜的數學問題，精確度並不是必需的。例如，「100000 / 10」
2. 寫下假設。在開始計算前，必須先定義系統的規模。在面試中，你可以主動提出一組合理的假設，並與面試官確認
3. 標記單位，以便消除歧義
4. 常見的粗略估計問題包括：[QPS（Queries Per Second）](https://ithelp.ithome.com.tw/m/articles/10349002)、峰值 QPS、儲存、快取、伺服器數量等
5. 數量級與基礎常識的掌握，記下一些常見數字

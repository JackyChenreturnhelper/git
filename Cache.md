# Cache

快取可以提高頁面讀取速度，並且減少伺服器和資料庫的負載。(減少主資料庫的負擔)

## Client

快取可以在客戶端(作業系統或瀏覽器)
讓Cache是在 Client 與 Server 之間透過 Header 的「交互作用」機制來完成。所以Cache會用Expire Date和Cache-control的交互應用來處理Cache的效期。

## Server
目的就是為了避免使用者對資料庫的大量寫入並獲得結果(Input/output DB資料庫)造成效能上的耗竭，而Server端使用Cache比較常見是Redis。

## CDN 快取

內容傳遞網路(CDN) 也被視為一種快取。


## 建議快取的資料：
不常更動的資料
大流量且不重要的頁面資訊
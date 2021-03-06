# availability 可用性 VS consistency 一致性


Web應用程序在過去十年快速發展、越來越普遍。作為應用開發人員，當然希望你的系統會被很多人使用。隨之而來的是事務的增長。如果你的系統依賴於持久化，數據存儲很可能會是你的系統瓶頸。

  任何應用程序都有兩種擴展的策略。第一種是垂直擴展，也是最簡單的一種：將應用遷移動更強大的SERVICE（更多的RAM、更多的空間）。對數據來說垂直擴展有效，但存在一些限制。最明顯的限制就是大型系統的數據容量的快速增長。垂直擴展費用高昂，通常需要購買下一代更新的系統來滿足額外的需求。垂直擴展通常會依賴於特定供應商，進一步增加成本，但硬體的升級是有極限的，終究還是遇到了瓶頸。

水平擴展提供更大的靈活性，也會更加複雜。數據的水平擴展可從維度進行。Functional Scaling 包含按功能對數據進行分組，並按功能將數據分佈在不同數據庫中。第二個維度是對同一功能組內的數據進行數據分片。

## CAP 理論

加州伯克利大學教授、Inktonmi共同創始人及首席科學家Eric Brewer認為web服務不能同時滿足以三個屬性：

1.          一致性。客戶端可立刻感知一組操作帶來的變化。

2.          可用性。每次的操作必須以既定的響應結束。

3.          分區容錯性：即使部分組件不可用，操作仍然可完成。

Web只能支持最多兩個屬性，不論採用哪種數據庫設計方案。水平擴展顯然是基於數據分區。因此設計人員只能從一致性和可用性中做出選擇。

## Partition tolerance
先看Partition tolerance，中文叫做"分區容錯"。

大多數分佈式系統都分佈在多個子網絡。每個子網絡就叫做一個區（partition）。分區容錯的意思是，區間通信可能失敗。比如，一台服務器放在台灣，另一台服務器放在美國，這就是兩個區，它們之間可能無法通信。

![Partitiontolerance](/images/bg2018071601.png)

上圖中，G1 和G2 是兩台跨區的服務器。G1 向G2 發送一條消息，G2 可能無法收到。系統設計的時候，必須考慮到這種情況。

一般來說，分區容錯無法避免，因此可以認為CAP 的P 總是成立。CAP 定理告訴我們，剩下的C 和A 無法同時做到。

## Consistency
Consistency 中文叫做"一致性"。意思是，寫操作之後的讀操作，必須返回該值。舉例來說，某條記錄是v0，用戶向G1 發起一個寫操作，將其改為v1。

![Partitiontolerance](/images/bg2018071602.png)

接下來，用戶的讀操作就會得到v1。這就叫一致性。

![Partitiontolerance](/images/bg2018071603.png)

問題是，用戶有可能向G2 發起讀操作，由於G2 的值沒有發生變化，因此返回的是v0。G1 和G2 讀操作的結果不一致，這就不滿足一致性了。

![Partitiontolerance](/images/bg2018071604.png)

為了讓G2 也能變為v1，就要在G1 寫操作的時候，讓G1 向G2 發送一條消息，要求G2 也改成v1。

![Partitiontolerance](/images/bg2018071605.png)

這樣的話，用戶向G2 發起讀操作，也能得到v1。

![Partitiontolerance](/images/bg2018071606.png)

## Availability
Availability 中文叫做"可用性"，意思是只要收到用戶的請求，服務器就必須給出回應。

用戶可以選擇向G1 或G2 發起讀操作。不管是哪台服務器，只要收到請求，就必須告訴用戶，到底是v0 還是v1，否則就不滿足可用性。

## Consistency 和Availability 的矛盾
一致性和可用性，為什麼不可能同時成立？答案很簡單，因為可能通信失敗（即出現分區容錯）。

如果保證G2 的一致性，那麼G1 必須在寫操作時，鎖定G2 的讀操作和寫操作。只有數據同步後，才能重新開放讀寫。鎖定期間，G2 不能讀寫，沒有可用性不。

如果保證G2 的可用性，那麼勢必不能鎖定G2，所以一致性不成立。

綜上所述，G2 無法同時做到一致性和可用性。系統設計時只能選擇一個目標。如果追求一致性，那麼無法保證所有節點的可用性；如果追求所有節點的可用性，那就沒法做到一致性。


## 取舍

CAP理論提出就是針對分佈式數據庫環境的，所以，P這個屬性是必須具備的。
P就是在分佈式環境中，由於網絡的問題可能導致某個節點和其它節點失去聯繫，這時候就形成了P(partition)，也就是由於網絡問題，將系統的成員隔離成了2個區域，互相無法知道對方的狀態，這在分佈式環境下是非常常見的。
因為P是必須的，那麼我們需要選擇的就是A和C。
大家知道，在分佈式環境下，為了保證系統可用性，通常都採取了複製的方式，避免一個節點損壞，導致系統不可用。那麼就出現了每個節點上的數據出現了很多個副本的情況，而數據從一個節點複製到另外的節點時需要時間和要求網絡暢通的，所以，當P發生時，也就是無法向某個節點複製數據時，這時候你有兩個選擇：
選擇可用性A(Availability)，此時，那個失去聯繫的節點依然可以向系統提供服務，不過它的數據就不能保證是同步的了（失去了C屬性）。
選擇一致性C(Consistency)，為了保證數據庫的一致性，我們必須等待失去聯繫的節點恢復過來，在這個過程中，那個節點是不允許對外提供服務的，這時候系統處於不可用狀態(失去了A屬性)。

最常見的例子是讀寫分離，某個節點負責寫入數據，然後將數據同步到其它節點，其它節點提供讀取的服務，當兩個節點出現通信問題時，你就面臨著選擇A（繼續提供服務，但是數據不保證準確），C（用戶處於等待狀態，一直等到數據同步完成）。

## 總結

現如今，對於多數大型網路應用的場景，主機眾多、部署分散，而且現在的集群規模越來越大，節點只會越來越多，所以節點故障、網絡故障是常態，因此分區容錯性也就成為了一個分佈式系統必然要面對的問題。那麼就只能在C和A之間進行取捨。但對於傳統的項目就可能有所不同，拿銀行的轉賬系統來說，涉及到金錢的對於數據一致性不能做出一絲的讓步，C必須保證，出現網絡故障的話，寧可停止服務，可以在A和P之間做取捨。

總而言之，沒有最好的策略，好的系統應該是根據業務場景來進行架構設計的，只有適合的才是最好的。

## 在什麼場合，可用性高於一致性？(AP)

我們在商城上瀏覽一個商品有他的瀏覽記錄

在做活動的時候 每個商品也有瀏覽記錄 但是必須保證服務器能正常運行

不會優先統計瀏覽記錄 也就是AP（保證服務器的可用性）

## 在什麼場合，一致性高於可用性？(CP)

金錢交易等等注重安全的行為
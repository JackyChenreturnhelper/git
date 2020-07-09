
# 1.1. kubernetes介紹

kubernetes是一種開源的容器編排工具，通過調度系統維持用戶預期數量和狀態的容器正常運行。kubernetes提供的功能：

• 服務發現和負載均衡：Kubernetes 可以使用DNS 名稱或自己的IP 地址公開容器，如果到容器的流量很大，Kubernetes 可以負載均衡並分配網絡流量，從而使部署穩定。

• 存儲編排
Kubernetes 允許您自動掛載您選擇的存儲系統，例如本地存儲、公共雲提供商等

• 自動部署和回滾
您可以使用Kubernetes 描述已部署容器的所需狀態，它可以以受控的速率將實際狀態更改為所需狀態。例如，您可以自動化Kubernetes 來為您的部署創建新容器，刪除現有容器並將它們的所有資源用於新容器。

• 自我修復
Kubernetes 重新啟動失敗的容器、替換容器、殺死不響應用戶定義的運行狀況檢查的容器，並且在準備好服務之前不將其通告給客戶端。

• 密鑰與配置管理
Kubernetes 允許您存儲和管理敏感信息，例如密碼、OAuth 令牌和ssh 密鑰。您可以在不重建容器鏡像的情況下部署和更新密鑰和應用程序配置，也無需在堆棧配置中暴露密鑰。

# 1.2. kubernetes架構圖
![kubernetes架構圖](/images/1579312205203-1c0e9f19-36df-483a-8218-52cc01708172.png)

![kubernetes架構圖](/images/1579312164461-0950f02a-1227-4c34-9388-f07250eae017.jpeg)

![kubernetes架構圖](/images/1579315734436-7816bddf-dd44-45e6-a358-8ce77d128d6b)

# 2. kubernetes組件

## 2.1. etcd

etcd 是兼具一致性和高可用性的鍵值數據庫，可以作為保存Kubernetes 所有集群數據的後台數據庫。在二進制部署etcd集群的時候，必須要考慮到高可用方案，一般部署三個或者三個以上的奇數個節點，因為當master宕機時，是通過選舉制度來選擇master的。

## 2.2. master組件

master和node是兩個邏輯上節點，當服務器資源充足時，可以將其分開在不同的機器上部署，當服務器資源不足時，也可以放到同一台機器上部署。master節點在部署的時候必須要考慮高可用方案，至少部署兩個master。

### 2.2.1. apiserver

主節點上負責提供Kubernetes API 服務的組件；它是Kubernetes 控制面的前端。是整個集群中資源操作的唯一入口，並提供認證、授權、訪問控制、API註冊和發現等機制。
apiserver提供了集群管理的restful api接口(鑑權、數據校驗、集群變更等)，負責和其它模塊之間進行數據交互，承擔了通信樞紐功能。
通過kubectl操作集群資源時需要登陸到master節點之上，而跨主機調用apiserver時，直接通過master前端的負載均衡來實現，詳細可以參考：https://www.yuque.com/duduniao/ww8pmw/tr3hch#ziHTp

### 2.2.2. controller manager

controller manager 譯為“控制器管理器”，k8s內部有很多資源控制器，比如：Node Controller、Replication Controller、Deployment Controller、Job Controller、Endpoints Controller等等，為了降低複雜度，將這些控制切都編譯成了一個可執行文件，並且在同一個進程中運行。
controller manager 負責維護集群的狀態，比如故障檢測、自動擴展、滾動更新等。

### 2.2.3. scheduler

調度器組件監視那些新創建的未指定運行節點的Pod，並選擇節點讓Pod 在上面運行。調度決策考慮的因素包括單個Pod 和Pod 集合的資源需求、硬件/軟件/策略約束、親和性和反親和性規範、數據位置、工作負載間的干擾和最後時限。
當前各個node節點資源會通過kubelet匯總到etcd中，當用戶提交創建pod請求時，apiserver將請求交給controller manager處理，controller manager通知scheduler進行篩選合適的node。此時scheduler通過etcd中存儲的node信息進行預選(predict)，將符合要求的node選出來。再根據優選(priorities)策略，選擇最合適的node用於創建pod。

## 2.3. node組件

### 2.3.1. kubelet

一個在集群中每個節點上運行的代理，kubelet 接收一組通過各類機制提供給它的PodSpecs，確保這些PodSpecs 中描述的容器處於運行狀態且健康。kubelet 不會管理不是由Kubernetes 創建的容器。
簡單來說主要是三個功能：

• 接收pod的期望狀態(副本數、鏡像、網絡等)，並調用容器運行環境(container runtime)來實現預期狀態，目前container runtime基本都是docker ce。需要注意的是，pod網絡是由kubelet管理的，而不是kube-proxy。

• 定時匯報節點的狀態給apiserver，作為scheduler調度的基礎

• 對鏡像和容器的清理工作，避免不必要的文件資源佔用磁盤空間

### 2.3.2. kube-porxy

kube-proxy是集群中每個節點上運行的網絡代理，是實現service資源功能組件之一。kube-proxy建立了pod網絡和集群網絡之間的關係，即cluster ip和pod ip中間的關係。不同node上的service流量轉發規則會通過kube-proxy進行更新，其實是調用apiserver訪問etcd進行規則更新。
service流量調度方式有三種方式: userspace(廢棄，性能很差)、iptables(性能差，複雜)、ipvs(性能好，轉發方式清晰)。

![kubernetes架構圖](/images/1578149293211-8106014b-6239-4038-8f51-5ad7a6f72d28.png)

### 2.3.3. container runtime

即容器運行環境，Kubernetes 支持多個容器運行環境: Docker、 containerd、cri-o、 rktlet 以及任何實現Kubernetes CRI (容器運行環境接口)。常用的是Docker CE。

## 2.4. 核心附件

核心附件雖然不都是必須要存在的，但是為了更好使用和管理kubernetes集群，這些核心附件強烈建議安裝！

### 2.4.1. CNI網絡插件

Kubernetes設計了網絡模型，但是卻將具體實現方式交給了網絡插件來實現，CNI網絡插件實現的是pod之間跨宿主機進行通信。默認情況下安裝好node節點後，無法跨node進行宿主機通信。可以參考: https://www.yuque.com/duduniao/ww8pmw/tr3hch#GtBRc
CNI網絡插件有很多，比如：Flannel、Calico、Canal、Contiv、OpenContrail等，其中最常用的是Flannel和Calico兩種。Flannel在node節點比較少，不誇網段時性能比較好，也是市場佔有率最高的網絡插件。

### 2.4.1.1. Flannel 通信原理
在已安裝完畢flannel的node節點上，查看路由表會發現，其實flannel就是添加了一個路由信息，將跨宿主機訪問pod的路由方式寫到了系統路由表中。

### 2.4.1.2. Flannel三種工作模式
• Host-Gateway 模式：
只是在主機上添加前往其它主機的pod網段的路由表而已，這種模式在同一個網段內部效率非常高。

![kubernetes架構圖](/images/1579327528851-66d99344-c34b-4a23-aba4-821ab7316b8a.png)

• VXLan模式：

VXLan(Virtual Extensible LAN虛擬可擴容局域網)，是Linux本身就支持的模式，vxlan模型中的VTEP(VXLAN Tunnel End Point虛擬隧道端點)就是各個node上的flannel.1設備。

當使用vxlan模式工作時，會將數據包經過docker0轉給flannel.1設備，該設備會查詢到目標pod宿主機的flannel.1設備的mac地址，在數據包基礎上加封一個vxlan頭部，這裡面有一個VNI標誌，flannel默認為1，flannel.1將封裝好的數據包交給宿主機網卡ens32處理。ens32再添加目標pod的宿主機ip和mac。

目標pod的flannel.1的mac地址和宿主機的IP地址都是由flannel進程通過etcd維護的。

![kubernetes架構圖](/images/1579330560501-5c147a03-41ff-4e69-b185-59caa1eba48e.png)

### 2.4.1.3. flannel的模式選擇和查看
在生產中，一般會將所有的node規劃到同一個局域網下，200多台的高配置的node基本夠用了。對於環境隔離要求很高的業務，也不會放到同一個k8s集群，因此跨網段進行node管理的場景不多。host-gw模式下性能損失很小，vxlan模式下性能損失較大，如果存在跨網段可以考慮flannel的混合模式，或者採用其它的CNI網絡插件。

### 2.4.1.4. flannel 部署和nat優化
https://www.yuque.com/duduniao/ww8pmw/tr3hch#boFnZ
## 2.4.2. DNS
在k8s集群中pod的IP地址會發生變化，k8s通過標籤選擇器篩選出一組pod，這組pod共同指關聯到抽象資源service，這個service擁有固定的IP地址，即cluster ip。此時無論後端的pod如何變化，都可以請求都可以通過service到達後端的pod上。
每個service有著自己的名稱和cluster IP地址，可以通過IP地址直接訪問到service後端對應的pod資源，也可以使用DNS將cluster ip 和service 名稱關聯，實現通過service的名稱訪問後端pod。kubernetes的dns插件有kube-dns和coredns兩種，在kubernetes 1.11 開始使用coredns較多，在此之前使用kube-dns比較多。
使用dns解析時，如果在pod外，需要使用全域名解析如：nginx-web.default.svc.cluster.local，pod內部則可以使用短域名。

## 2.4.3. Dashboard
Dashboard是管理Kubernetes集群一個WEB GUI插件，可以再瀏覽器中實現資源配置和查看。可參考：https://www.yuque.com/duduniao/ww8pmw/myrwhq#aZGMp
### 2.4.4. Ingress Controller
service是將一組pod管理起來，提供了一個cluster ip和service name的統一訪問入口，屏蔽了pod的ip變化。 
ingress是一種基於七層的流量轉發策略，即將符合條件的域名或者location流量轉發到特定的service上，而ingress僅僅是一種規則，k8s內部並沒有自帶代理程序完成這種規則轉發。ingress-controller是一個代理服務器，將ingress的規則能真正實現的方式，常用的有nginx,traefik,haproxy。但是在k8s集群中，建議使用traefik，性能比haroxy強大，更新配置不需要重載服務，是首選的ingress-controller。
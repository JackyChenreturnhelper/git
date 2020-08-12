# Terraform

## Terraform 是什麼？

Terraform 是一種安全有效地構建、更改和版本控制基礎設施的工具(基礎架構自動化的編排工具)。它的目標是"Write, Plan, and create Infrastructure as Code", 基礎架構即代碼。Terraform 幾乎可以支持所有市面上能見到的雲服務。具體的說就是可以用代碼來管理維護IT 資源，把之前需要手動操作的一部分任務通過程序來自動化的完成，這樣的做的結果非常明顯：高效、不易出錯。

Terraform 提供了對資源和提供者的靈活抽象。該模型允許表示從物理硬件、虛擬機和容器到電子郵件和DNS 提供者的所有內容。由於這種靈活性，Terraform 可以用來解決許多不同的問題。這意味著有許多現有的工具與Terraform 的功能重疊。但是需要注意的是，Terraform 與其他系統並不相互排斥。它可以用於管理小到單個應用程序或達到整個數據中心的不同對象。

Terraform 使用配置文件描述管理的組件(小到單個應用程序，達到整個數據中心)。Terraform 生成一個執行計劃，描述它將做什麼來達到所需的狀態，然後執行它來構建所描述的基礎結構。隨著配置的變化，Terraform 能夠確定發生了什麼變化，並創建可應用的增量執行計劃。

# Terraform 核心功能

    基礎架構即代碼(Infrastructure as Code)
    執行計劃(Execution Plans)
    資源圖(Resource Graph)
    自動化變更(Change Automation)

## 基礎架構即代碼(Infrastructure as Code)
使用高級配置語法來描述基礎架構，這樣就可以對數據中心的藍圖進行版本控制，就像對待其他代碼一樣對待它。

## 執行計劃(Execution Plans)
Terraform有一個plan步驟，它生成一個執行計劃。執行計劃顯示了當執行apply命令時Terraform將做什麼。通過plan進行提前檢查，可以使Terraform操作真正的基礎結構時避免意外。

## 資源圖(Resource Graph)
Terraform構建的所有資源的圖表，它能夠並行地創建和修改任何沒有相互依賴的資源。因此，Terraform可以高效地構建基礎設施，操作人員也可以通過圖表深入地解其基礎設施中的依賴關係。

## 自動化變更(Change Automation)
把複雜的變更集應用到基礎設施中，而無需人工交互。通過前面提到的執行計劃和資源圖，我們可以確切地知道Terraform將會改變什麼，以什麼順序改變，從而避免許多可能的人為錯誤。
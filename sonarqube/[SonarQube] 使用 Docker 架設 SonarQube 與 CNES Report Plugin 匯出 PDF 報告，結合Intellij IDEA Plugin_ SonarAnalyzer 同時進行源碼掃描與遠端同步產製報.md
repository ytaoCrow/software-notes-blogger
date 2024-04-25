# [SonarQube] 使用 Docker 架設 SonarQube 與 CNES Report Plugin 匯出 PDF 報告，結合Intellij IDEA Plugin: SonarAnalyzer 同時進行源碼掃描與遠端同步產製報告

[TOC]

## 前言：

本篇筆記著重在 SonarQube，紀錄初學 SonarQube 的應用，分享如何使用 SonarQube CNES Report Plugin 匯出源碼掃描之報告，並結合 Intellij IDEA SonarAnalyzer plugins 在 IDEA 內部本地使用 SonarQube  同時進行源碼掃描與遠端同步產製報告

## 使用 Docker 架設 SonarQube

### Step 1. 下載 Docker Image

#### 優先建立要使用的開源程式碼檢測平台：SonarQube

* 執行以下指令，讓 Docker 建立並運行 SonarQube 的 container

```dockerfile=
docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube:lts
```

* -d：detach，建立 container 後，後台運行容器
* –name：命名 container
* -p：Docker 外部與 SonarQube內部對應的 port，其中左邊為外部 * Docker 的 port，右邊為 SonarQube 內部的 port
* sonarqube:lts：SonarQube 的 LTS 版本

#### 執行結果：

![截圖 2024-04-24 上午10.49.42](https://hackmd.io/_uploads/ry4UmlI-0.png)

### Step 2. 打開 SonarQube

#### 預設網址為：http://localhost:9000/

* 第一次登入時，會要求你輸入帳號密碼
* 預設帳號：admin
* 預設密碼：admin

![截圖 2024-04-24 上午10.54.43](https://hackmd.io/_uploads/HJhaEgIWC.png)

#### 然後會要求更改密碼，這時間輸入自己密碼喜好就好
#### 切記！一定要記住密碼

![截圖 2024-04-24 上午10.56.48](https://hackmd.io/_uploads/rJpgHxLZC.png)

#### 密碼更改完後，就會進入 SonarQube 的主頁面

![截圖 2024-04-24 上午10.58.18](https://hackmd.io/_uploads/ByDIBeLWC.png)

### Step 3. 使用 SonarQube 手動建立項目

* 本次只先教導大家，如何用在本地專案
* 未來會新增如何串接 GitHub 和 GitLab

#### 點選 Manually

![截圖 2024-04-24 上午11.04.55](https://hackmd.io/_uploads/BJ1SPlU-R.png)

#### 進入到手動 Create Project

* 創建自己的專案名稱
* 這裡我先用 testProject 命名
* branch 先選 main 即可
* 都命名完後，可點擊 **SetUp** 創建專案

![截圖 2024-04-24 上午11.07.18](https://hackmd.io/_uploads/ryruveUZR.png)

#### 點選 Locally
* 指定為本地專案

![截圖 2024-04-24 上午11.11.13](https://hackmd.io/_uploads/S1WOOlLbR.png)

* 這時候會進到 Overview 裡面
* Generate a project token 出來

![截圖 2024-04-24 上午11.12.30](https://hackmd.io/_uploads/Hk6sOxU-R.png)

#### 點選 Continue
* 這個 token 要特別備份出來，待會串接 SonarAnalyzer 會用到

![截圖 2024-04-24 上午11.14.17](https://hackmd.io/_uploads/rkuGtx8-A.png)

#### 選擇你在用自動化建構工具

* 由於筆者習慣用 Maven，所以就選擇 Maven
* 挑選工作需要用到的就好

![截圖 2024-04-24 上午11.16.20](https://hackmd.io/_uploads/Sk-5YeLW0.png)

#### 點選完後，就會出現 mvn clean 的指令，把整個指令都複製下來

![截圖 2024-04-24 上午11.19.12](https://hackmd.io/_uploads/B1kH9eLZ0.png)

### Step 4 . 用 Terminal 指定到自己的專案目錄，貼上指令

* 筆者這裡直接使用 Intellij IDEA 內建的 Terminal
1. ![截圖 2024-04-24 上午11.25.23](https://hackmd.io/_uploads/By72slUZA.png)
2. ![截圖 2024-04-24 上午11.26.04](https://hackmd.io/_uploads/ryK0ilIZR.png)

### Step 5. 回到原本的 SonarQube 網頁

* BUILD SUCCESS 回到原本的 SonarQube 網頁，就能看到掃描結果了

![截圖 2024-04-24 上午11.29.48](https://hackmd.io/_uploads/BJx1ae8ZR.png)

* 這時候原本設置的 testProject，會被變成源碼掃描的專案名稱
* 他會根據 pom.xml 裡面的 tag，改變自己的名稱，以符合掃描的專案名稱
```xml=
<name>accounts</name>
```
* ![截圖 2024-04-24 上午11.34.20](https://hackmd.io/_uploads/H1t6axLbA.png)


## 掛載 CNES Report Plugin 到 docker container 匯出 PDF 報告

目前使用的 SonarQube 是 Community Edition 9.9 LTS，但預設並不支援 PDF 報告的產製。若想要取得紙本報告，就需要透過外掛來實現。最初我找到了一篇由 [Ivan Chang 在 iThome 上分享的文章](https://ithelp.ithome.com.tw/articles/10313246)，介紹了如何在 SonarQube 8.7 版本下產生 PDF 分析報告。但可惜的是，文章中提到的外掛已不再支援。因此，需要尋找其他可行的外掛或解決方案。


### [SonarQube CNES Report](https://github.com/cnescatlab/sonar-cnes-report) 

是由 [CNES CAT LAB](https://cnescatlab.github.io/) 所維護的報告文件產製套件，算是目前有在定期更新的第三方套件。

![upload_b0f59fedfc0bddc59155873ab7bf43a4](https://hackmd.io/_uploads/HJ5febUZR.png)

* 雖然官網是寫不再支援 SonarQube CNES Report 這個插件，CAT Lab Github 上的 [README](https://github.com/cnescatlab/sonar-cnes-report#compatibility-matrix) 已經有清楚寫明各 SonarQube 版本支援的情況，讀者可以直接根據自己使用的 SonarQube 版本，選擇對應的 Release jar。
* ![截圖 2024-04-24 上午11.46.41](https://hackmd.io/_uploads/H1-px-IbC.png)

### Step 1. 進入 [Release 頁面](https://github.com/cnescatlab/sonar-cnes-report/releases) 選擇對應的 Release jar

* 筆者這裡是使用 4.3.0.jar，我測試下來是可以使用的
* 或是你可以選擇適用的 jar 檔


![截圖 2024-04-24 上午11.50.32](https://hackmd.io/_uploads/HJch-Z8WA.png)

### Step 2. 解壓縮 Jar 檔，放在桌面


![截圖 2024-04-24 上午11.58.44](https://hackmd.io/_uploads/SJlFQWI-C.png)

### Step 3. 進到Docker containter 內部

* 將桌面的 sonar-cnes-report-4.3.0.jar 掛載或覆寫到 SonarQube container 的 /opt/sonarqube/extensions/plugins/ 路徑底下

* 使用這個指令進到 sonarqube 這個 container 內部
```dockerfile=
docker exec -it sonarqube /bin/bash
```
![截圖 2024-04-24 下午2.23.26](https://hackmd.io/_uploads/BJ0vrmU-A.png)

* 在進到「cd extensions/plugins/」這個目錄裡面
* 原本在 plugins 底下只有 README.txt
![截圖 2024-04-24 下午2.17.44](https://hackmd.io/_uploads/rJNzN7L-0.png)

#### 用下面的指令，從桌面的 jar 檔，放入 sonarqube container 內部

* 若前面已經跑起一個 SonarQube Container，可以選擇把它刪掉或更改新的 container name
* --name sonarqube -> --name sonarqubeplus
* 筆者這裡選擇刪除，再重跑一個新的 containter

```dockerfile=
docker run -d -v /<你的檔案目錄路徑>/sonar-cnes-report-4.3.0.jar:/opt/sonarqube/extensions/plugins/sonar-cnes-report-4.3.0.jar --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube:lts
//範例：
docker run -d -v /dont/tell/you/sonar-cnes-report-4.3.0.jar:/opt/sonarqube/extensions/plugins/sonar-cnes-report-4.3.0.jar --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube:lts
```

* 最後再進到 containter 內部，檔案已經被放進去囉！

![截圖 2024-04-24 下午2.39.03](https://hackmd.io/_uploads/HkUMtmIWC.png)

### Step 4 . 回到 SonarQube 主頁

* 點選 More 就會出現 CNES Report 的插件囉！


![截圖 2024-04-24 下午2.55.45](https://hackmd.io/_uploads/SJkW678WA.png)

* 點進去，就可以開始產生報告了

![截圖 2024-04-24 下午2.57.15](https://hackmd.io/_uploads/BJD8pX8-C.png)

## Intellij IDEA Plugin: SonarAnalyzer 同時進行源碼掃描與遠端同步產製報告

SonarAnalyzer 是一種靜態程式碼分析工具，它是 SonarSource 公司的產品之一。這個工具用於檢測程式碼中的缺陷、漏洞、代碼風格問題等。SonarAnalyzer 可以集成到不同的集成開發環境（IDE）中，如 Eclipse、IntelliJ IDEA 等，也可以與持續集成工具（如 Jenkins）和版本控制系統（如 Git）集成，以實現自動化的程式碼分析和檢查。它能夠提供即時反饋，幫助開發人員在開發過程中及時發現和修復程式碼中的問題，從而提高代碼的質量和可維護性。


* 可以使用 SonarAnalyzer 連上 SonarQube 用 IDEA 進行源碼掃描


![截圖 2024-04-24 下午3.00.59](https://hackmd.io/_uploads/HJFEAQLbR.png)

* 先安裝好套件，再進到 Tools 裡面，找到 SonarAnalyzer 進行連接
* **點選 Url 旁邊的 + 號**

![截圖 2024-04-24 下午3.03.35](https://hackmd.io/_uploads/S1NRAQIZA.png)

* 就會跳出視窗，要求你填寫連線資訊

![截圖 2024-04-24 下午3.06.58](https://hackmd.io/_uploads/Hkei1VIWA.png)
* 會要求你的 url 和 token，這時候再輸入
    1. 未這個連線命名
    2. http://localhost:9000/
    3. 當時建立的 token 序號


![截圖 2024-04-24 下午3.11.44](https://hackmd.io/_uploads/r1Ahe4U-C.png)

* 再按 ok 和 Apply 就可以用這個套件掃描整個專案囉

* 按綠色箭頭啟動掃描
![截圖 2024-04-24 下午3.15.35](https://hackmd.io/_uploads/HJSjZ4IWC.png)
* 掃描結果就會出來囉！
![截圖 2024-04-24 下午3.17.14](https://hackmd.io/_uploads/Hyv-zN8WR.png)

之後就可以再Intellij IDEA內部，編寫程式，邊進行源碼掃描。
持續提升系統品質！！！


## reference: 

* [原來程式碼品質也可以被檢測：初探 SonarQube](https://medium.com/starbugs/%E5%8E%9F%E4%BE%86%E7%A8%8B%E5%BC%8F%E7%A2%BC%E5%93%81%E8%B3%AA%E4%B9%9F%E5%8F%AF%E4%BB%A5%E8%A2%AB%E6%AA%A2%E6%B8%AC-%E5%88%9D%E6%8E%A2-sonarqube-14e99687806e)
* [如何使用 Docker 安裝 SonarQube？](https://old-oomusou.goodjack.tw/sonarqube/docker/)
* [SonarQube Community Edition 使用 SonarQube CNES Report Plugin 匯出 PDF 報告](https://hackmd.io/@dh46tw/sonarqube-community-cnes-report-plugin)
* [SonarQube 如何產生 PDF 分析報告](https://ithelp.ithome.com.tw/articles/10313246)
* [sonar-cnes-report](https://github.com/cnescatlab/sonar-cnes-report)
* [Day17：使用 Docker Volume 的功能 (一)](https://ithelp.ithome.com.tw/articles/10192397)



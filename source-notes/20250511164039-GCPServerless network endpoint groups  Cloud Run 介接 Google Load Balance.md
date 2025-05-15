---
type: source-note
title: "[GCP]Serverless network endpoint groups | Cloud Run 介接 Google Load Balance"
id: 20250511160539
created: 2025-05-11T16:40:39
source:
  - web
url: https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/
tags:
  - source-note
  - cloud-native/serverless
processed: false
archived: false
---
Status: Completed

## Agenda

## 簡介 Serverless Network Endpoint Groups

目前 [NEG support serverless (in in Beta) \[1\]](https://cloud.google.com/load-balancing/docs/negs/serverless-neg-concepts) ，透過Load balance 轉 `HTTPs` 轉換為 `HTTP` 可以滿足BIOS 連接Cloud Run需求。另外我理解acer，也需要WAF功能抵擋外部攻擊，但現在\[Cloud Armor\[2\]\](Cloud Armor.)還未能實現，最後文章指出未來幾個月Cloud Armor也會加入戰力

Network Endpoint Groups (NEG) 新功能歸功於Google Cloud網絡和負載平衡的基本功能， NEG的運作原理，請見圖1，它是存在Load balance的後端，它可以定義應外部流量，如何到達一組端點(Endpoint)，因此NEG 可以服務外來request，並將request直接經Load balance發送給GCP backend (eg. 可以是Compute Engine VM 或在VM上運行的服務，或是Endpoint URL, FQDN: port, IP)

[![Untitled.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled.png)

## 準備環境

1. Cloud Run server：已準備 `ngnix web` [https://ngnix-v1-ve3udnlh4q-uc.a.run.app](https://ngnix-v1-ve3udnlh4q-uc.a.run.app/)
2. Network Endpoint Groups (NEG) ：設定一組 `Network endpoints`
3. Load balance：設定backend選擇 `Network endpoints`

## A Service runs on Cloud Run

### Step1. 建立Cloud Run Service

- `endpoint`: [https://ngnix-v1-ve3udnlh4q-uc.a.run.app](https://ngnix-v1-ve3udnlh4q-uc.a.run.app/)

[![Untitled1.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled1.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled1.png)

### Step2. 測試endpoint

- 注意現在使用https連線
	[![Untitled2.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled2.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled2.png)

## Enable Network Endpoint Groups(NEG)

### Step1. 建立 Network Endpoint Group

- 此步驟是將外部來源轉為 `GCP 內的 backend service` ，方可與 GCP HTTP(S) LB 串接。
- 路徑 GCP console -> Compute Engine -> Network Endpoint Group

[![Untitled3.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled3.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled3.png)

### Step2. Create Network Endpoint Group

[![Untitled4.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled4.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled4.png)

### Step3. 設定NEG

- NEG 命名
- Network endpoint group type：請選 `Internet`
- Default port：因為來源是Cloud Run 走 `HTTPs` `443`
- Add through endpoint
	- Fully qualified domain name
	- Fully qualified domain name：請輸入你的 `Cloud Run endpoint`  
		(eg. ngnix-v1-ve3udnlh4q-uc.a.run.app)

[![Untitled5.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled5.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled5.png)

## Load balance

### Step1. 建立 Load balance

- 建立 GCP HTTP(S) LB, 在 GCP console -> Network services -> Load balancing
- 選擇 `backend services`

[![Untitled6.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled6.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled6.png)

### Step2. 設定 backend

- backend service 命名
- backend type：請選 `Internet network endpoint group`
- Default port：因為來源是Cloud Run 走 `HTTPs` `443`

[![Untitled7.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled7.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled7.png)

詳細設定如下

- Protocol 請選 `HTTPS`
- Backend 選擇上面步驟建立 `Network Endpoint Group`
- 其餘設定依需求自行調整

[![Untitled8.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled8.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled8.png)

header(非常重要)

- Custom request headers
	- Header name： `host`
	- Header value：Cloud Run 產生的 `endpoint`  
		(eg. ngnix-v1-ve3udnlh4q-uc.a.run.app)

[![Untitled9.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled9.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled9.png)

### Step2. 設定frontend

這邊可以有不同的option `HTTP`, `HTTPs` ，也可以同時開啟 80 與 443 ，作法需各別開兩個 `frontend` 設定

- **Option1. HTTP**
	- frontend 命名
	- Protocol ：選擇 `HTTP`
	- IP address ：可選擇 `Ephemeral`, 或先註冊一組 `static IP`
	- Port： `80`
		[![Untitled10.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled10.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled10.png)
- **Option2. HTTPs with certificate**
	- frontend 命名
	- Protocol ：選擇 `HTTPS`
	- IP address ：可選擇 `Ephemeral`, 或先註冊一組 `static IP`
	- Port： `443`
	- Certificate 可以有二種選擇
		1. https 可以 `自行管理` 或是使用 `Google管理的證書`
		2. 若要使用自己的SSL證書，可上傳憑證檔；format 如下  
			填入 `Public key certificate`, `Certificate chain`, `Private key`
			[![Untitled11.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled11.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled11.png)
	- Certificate：填入 `Public key certificate`, `Certificate chain`, `Private key`
	- 設定完後建立，需要時間生效
		[![Untitled12.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled12.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled12.png)

使用Google管理的SSL證書

- 憑證也可以使用Google管理的SSL證書，輸入網域名稱即可，若該網域的DNS設定有正確指向HTTP(S) Load Balancer的IP，少許時間之後可以看到該證書為 `ACTIVE` 狀態。
	[![Untitled13.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled13.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled13.png)

## Review Setting of Load balancer

- 系統會確認您的設定值

[![Untitled14.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled14.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled14.png)

- 若無誤status 會呈現綠色勾勾

[![Untitled15.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled15.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled15.png)

## Test Original Cloud Run with “HTTP header”

### Test1. curl

- Cloud Run base是採kubernetes，故ingress是L7 load balance，所以Cloud Run 需要透過 `header` host來作為辨識
- 因此curl 時可以指定Load balance HTTP IP後，再補上 `-H host header`
```bash
$ curl -k -H "host: ngnix-v1-ve3udnlh4q-uc.a.run.app" http://35.201.76.173
```

[![Untitled16.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled16.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled16.png)

### Test2. Browser

- 我們已經了解Clour Run識別需要使用header，其實上面設定load balance froentend時，就已經把header寫入，故我們在Browser連線時就不用在加上header
- 透過HTTP([http://35.201.76.173/](http://35.201.76.173/))，可以正常連線至Cloud Run ngnix 網頁

[![Untitled17.png](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled17.png)](https://joehuang-pop.github.io/2020/08/01/GCP-Serverless-network-endpoint-groups-Cloud-Run-%E4%BB%8B%E6%8E%A5-Google-Load-Balance/Untitled17.png)

## References

1. [Serverless network endpoint groups overview](https://cloud.google.com/load-balancing/docs/negs/serverless-neg-concepts)
2. [Global HTTP(S) Load Balancing and CDN now support serverless compute](https://cloud.google.com/blog/products/networking/better-load-balancing-for-app-engine-cloud-run-and-functions)
# Docker 中的 Image、Container、Registry 的用途

在 Docker 中，Image、Container、Registry 是很常出現的三個名詞：

## 1. Image（映像檔）

這裡可以把 image 想像成是任何一段程式（或是一個 .jar 檔），所以當 Developer 寫完程式之後，就可以將這段程式封存成一個 image。

## 2. Container（容器）

Container 的概念類似於「買一台新電腦，然後在裡面運行 image 裡的程式」，在 Docker 中可以運行許多個 Container，每一個 Container 都是一個獨立的環境。

## 3. Registry（倉庫）

可以把 Registry 想像成一個雲端倉庫，裡面會存放所有 image 們，所以不管是要 push image 上去、還是要 pull image 下來，都是在跟 Registry 倉庫溝通。

而 Docker 官方的 Registry 叫做 Docker Hub，預設就是會從這個地方下載 image 下來。


# Docker 常用指令

## pull: 從倉庫中拉取 image 下來
- 在下載和使用 image 時，最好都要特別指定要使用的是哪個 tag，無腦用 latest 很容易會遇到版本升級導致的奇怪問題。

```bash
docker pull hello-world
docker pull hello-world:1.28
```

## run: 創建並運行 container（run 還有許多參數可擴展）
- 如果 run 一個還沒下載下來的 image，Docker 就會先去倉庫中下載該 image，並且載完之後直接運行該 container，等於是同時做 pull + run 兩件事。
- run 所生成的 container 在運行完畢之後不會自己刪掉，只是狀態會變成 exit 而已。
- 如果不想要一直創建新的 container 出來的話，就可以改成使用 `docker start [container id]` 指令，去重複運行已經存在的 container。

```bash
docker run hello-world:1.28
```

## start: 運行 container
- 一定要先使用 run 把 container 創建出來之後，start/stop 這兩個指令才有用。

```bash
docker start [container id]
```

## stop: 結束 container
```bash
docker stop [container id]
```

## ps -a: 查看所有 container 的運行狀態
- 注意指令需要加 -a 才能顯示全部的 container，不加的話只能看到當前運行的 container 們而已，沒辦法看到停止運行的 container。

```bash
docker ps -a
```

## exec: 對正在運行的 container 下達指令
- exec 通常是在初期開發時比較常用，因為它可以針對正在運行的 container 下達指令，或是登入該 container，去查看運行時的 log、或是程式位置是否有擺對地方。
 
```bash
docker exec -it [container id] /bin/bash  # 登入進該 container
```

## logs: 查看該 container 輸出的 log
- 直接列出 container 中的 log 出來，不用登入進去 container。
- 要注意是 `logs` 不是 `log`。

```bash
docker logs [container id]
docker logs -f [container id]  # 持續列出最新的 log
docker logs -f --tail 10 [container id]  # 持續列出最新的 log，並且只顯示最後 10 筆
```

## inspect: 查看該 container 的詳細資訊
- 如果想要查看這個 container 的詳細資訊（ex: 環境變數、網路設定），那就可以靠 inspect 指令來取得。

```bash
docker inspect f10ff7cdfb54
```

## rm: 刪除某個 container
```bash
docker rm [container id]
```

## rmi: 刪除某個 image
```bash
docker rmi [image id]  # 根據 image id 刪除
docker rmi [image name]:[image tag]  # 根據 image name 和 tag 刪除
```

## images: 查看目前已下載了哪些 image
```bash
docker images
```

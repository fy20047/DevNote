解決 ArgoCD 與 HPA 的同步衝突問題
====

問題：ArgoCD 顯示 `OutOfSync`
-----------------------------

在實作專案的自動化部署過程中，我發現當 HPA 啟動後，ArgoCD 的介面會頻繁地將 Deployment 標記為 `OutOfSync`。

- **原因**：Git 裡的 `replicas` 設定為 3，但又因為 HPA 自動調降變成了 2。ArgoCD 偵測到實際狀態與 Git 預期狀態不符，不斷嘗試將副本數改回 2，導致 Pod 反覆被刪除與建立。

---

K8s 控制器與 ArgoCD 的角色分工
-----------------------------

| **Component** | **角色跟行為** | **與 ArgoCD 的衝突** |
| --- | --- | --- |
| **ArgoCD** | **GitOps 控制器**：只負責將 Git 裡的資源「套用/校正」到叢集。 | 若它與其他控制器（如 HPA）同時修改同一個欄位（如 replicas），就會引發衝突。 |
| **HPA** | **K8s 內建 Controller**：根據 CPU/Memory 指標，直接修改 `Deployment.spec.replicas`。 | 因為會主動改變叢集狀態，導致與 Git 來源不一致。 |
| **PDB** | **中斷保護規則**：定義維護時（Eviction）「最少需保留」或「最多可驅逐」的 Pod 數量。 | 它只負責保護規則，不改動 replicas 數值，通常不與 ArgoCD 衝突。 |

**備註**：HPA 和 PDB **都不是** ArgoCD 的功能，它們是 K8s 本身的 Controller 機制。ArgoCD 的任務在於將這些 YAML 套用進 cluster，實際執行行為是由 K8s 的 control plane 負責。

---

解決方法
------------------

### 修改 ArgoCD  (`ignoreDifferences`)

在 ArgoCD 的 Application 設定中 (app.yaml)，加入忽略特定欄位的參數，告訴 ArgoCD 讓 HPA 管理 replicas：

```YAML
spec:
  ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas # 忽略 replicas 差異，避免 ArgoCD 再度改回 Git 設定
```
-   統一由 HPA 定義 `minReplicas` 與 `maxReplicas`，達成自動化彈性擴縮。
---

📝 結論
--------

在設計 CI/CD 流程時，必須清晰定義單一事實來源 (Single Source of Truth) 的邊界。

-   ArgoCD 負責資源的「部署與校正」。
-   K8s Controllers (HPA/PDB) 負責叢集內的「動態維運行為」。
-   透過 `ignoreDifferences` 可以有效解決多個 Controller 操作同一資源欄位時產生的 Race Condition。

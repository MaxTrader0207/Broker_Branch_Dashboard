# 券商分點量化研究模型（每日自動更新）

追蹤 5 個券商分點的逐日進出，產生 15 個子策略 Dashboard、2 個驗證頁（S16 訊號台帳、S17 消融測試）、一頁式摘要與總覽頁。
每個交易日晚上由 GitHub Actions 自動執行 `broker_dashboard.ipynb`，結果發佈到 GitHub Pages。

> 分點是「交易通路」而非投資人身分；所有分數皆為研究用途，不構成投資建議。

## 排程

| 項目 | 設定 |
|---|---|
| 執行時間 | 週一～五 台北 22:37（`.github/workflows/broker-daily.yml` 的 `cron`，時間為 UTC） |
| 第一次執行 | 無快取，約 2～2.5 小時（抓 3 年分點、法人、融資、當沖） |
| 之後每天 | 只補最新一天，約 5～15 分鐘 |
| 資料快取 | GitHub Actions cache（失敗或逾時也會存，下次只補缺的） |
| 失敗通知 | GitHub 預設寄 Email 給 repo 擁有者（不使用 LINE） |

工作會在以下情況**失敗且不發佈**，避免把錯的網頁覆蓋上去：自我測試未通過、最新交易日完全沒有分點資料。

## 第一次設定（只需做一次）

1. **Settings → Pages → Build and deployment → Source** 選 **GitHub Actions**。
2. **Actions** 分頁 → 選「券商分點每日更新」→ **Run workflow**（可勾「使用模擬資料」先測流程，約 1 分鐘）。
3. 再不勾模擬資料跑一次真資料（約 2 小時），完成後網址為 `https://<帳號>.github.io/<repo>/`。

## 訊號台帳（`ledger/`，V1.7）

每次排程把最新交易日每檔股票的 Score、ML Score、十大因子與各策略判定存成 `ledger/年/日期.csv`，並由工作流程自動提交到 `main`。

- **只增不改**：同一天重跑不覆寫（除非先前有分點缺資料、這次補齊）。滿 21 個交易日後回填 5／10／20 日報酬並定案，每個檔案最多被改寫一次。
- **不要手動編輯或刪除 `ledger/`**：它是唯一無法重新產生的資料（事後補算就不是樣本外了）。`index.json` 記錄每日的記錄時間與參數簽章，`columns.json` 是欄位對照。
- 提交失敗（例如同時有人 push）不會讓發佈失敗；台帳在 Actions cache 另有備份，下次執行會補回。
- 排程每天會在 `main` 新增一個提交，本機修改前請先 `git pull`。
- 勾「使用模擬資料」手動執行時不會寫入台帳。
- 結果看 `16_signal_ledger.html`；約 4 個月後才可能出現統計顯著的結論。

S17 消融測試每 7 天重算一次（約多 2～3 分鐘），其餘日子讀快取。

## 手動執行

- **立即更新**：Actions → 券商分點每日更新 → Run workflow。
- **在 Colab 跑**：直接開 `broker_dashboard.ipynb` 全部執行即可（不設 `BROKER_RUN_MODE` 時行為與以前相同，快取存雲端硬碟）。

## 改參數

所有參數在 notebook 第 1 個程式格「1. 參數設定」。改完 commit 到 `main`，下一次排程就會套用。
若改了影響歷史資料的參數（如 `LOOKBACK_YEARS`、`BRANCHES`），第一次會多抓缺少的部分。

## 每次執行的產出

- 網站：`index.html`（總覽）、`00_summary.html`（一頁式摘要）、`01`～`15` 子策略頁、`16` 訊號台帳、`17` 消融測試、`build_info.json`（資料日期、覆蓋率、截尾率、存活者偏誤、自我測試結果）。
- 執行過程：Actions → 該次執行 → build → 「執行 notebook」步驟，每格輸出即時顯示（看得到抓到第幾頁、哪個資料源慢）。
- Actions 的 artifact `executed-notebook`：含每格輸出的執行紀錄，保留 14 天，除錯用。

## 注意

- 公開 repo 連續 60 天沒有提交，GitHub 會停用排程；工作流程每 30 天會自動提交一次 `status/build_info.json` 避免此狀況。
- GitHub 的排程可能延遲數分鐘到一小時，尖峰時段偶爾會跳過一次。
- TPEx（上櫃法人／融資）若擋掉 GitHub 的連線，會自動退回中性分，不影響其他頁面。

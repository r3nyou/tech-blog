---
title: "Work_diary"
date: 2022-05-16T21:33:26+08:00
draft: false
---

> 這篇文是紀念來二天就離職的同事

同事來二天就離職，說每天寫工作日誌、晨會報告工作進度，壓力太大。

雖然我也很討厭寫日誌，從小就不愛簽聯絡簿，但工作日誌實屬必要，我還是堅持每天寫。

這是我某二天的工作日誌

2021-04-14
2021-04-15
- 頭痛，不想工作

2021-04-16
- 完成了依據 sku 統計 GA event，並且合併 term 的功能
- makeTagReport 改成產生二個 mongoDB collection（document term map skus & sku performance），過程需要用 go 執行另一個 command，發現使用 command 帶參數的問題，` out, err := exec.Command("./cobraCommand type --file=test.csv").Output()` 出現錯誤訊息 fork exec no such file or directory，後來看[文件](https://pkg.go.dev/os/exec#Command)裡有關於參數的說明: If name contains no path separators, Command uses LookPath to resolve name to a complete path if possible. Otherwise it uses name directly as Path. 原來 Command 的 name 參數無法寫整條命令，因為會被當成路徑，所以參數只能在後面帶入，例如 `out, err := exec.Command(“./ga-report”, “makeTagReport”, “--file=test.csv”).Output()`

我對工作日誌的要求是
- 誠實
- 必須寫清楚細節
- 每天都要寫

也許同事認為每天暴露工作內容，等同受到監督，因此得每天全力工作，想當然壓力大；但我認為沒有人能每天精神飽滿的工作，比起做了什麼，做不到、做不好的事情更值得記錄，畢竟不如意之事十常八九，如何把事情做好，取決於如何對待這八九。

工作日誌是寫給自己看的，如果不真實記錄自己的工作，就無法穩定的產出，也許在狀態好時產出驚人，而狀態差時一整天都沒有產出；沒產出並不可怕，可怕的是隔天就忘了，還以為自己昨天很努力，長久下來就懈怠了。
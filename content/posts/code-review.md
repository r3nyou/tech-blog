---
title: "不好好地 Code Review 怎麼行"
date: 2022-06-03T14:29:01+08:00
draft: false
tags: ["開發流程"]
categories: ["開發流程"]
---

Code Review 的好處是沒有爭議的，關於如何 Code Review、與諸多好處，可以參考：
- [Code Review Best Practices](https://blog.palantir.com/code-review-best-practices-19e02780015f)
- [How Google Does Code Review](https://dzone.com/articles/how-google-does-code-review)
- [LinkedIn’s Tips for Highly Effective Code Review](https://thenewstack.io/linkedin-code-review/)

但我發現有許多公司沒有 Code Reivew，所以蒐集了一些反對的原因：

## 不做 Code Review 的原因

**1. 有 QA，不要有 Bug 就好，Code Review 沒太大意義**

我想這是誤會，Code Review 並不負責找出程式碼的錯誤，這也是很多 Reviewer 的誤區，Code Review 應該把力氣放在程式碼的品質，例如可讀性、維護性、擴展性等，錯誤應該是由測試來處理的。

當然我不是說，不能在 Code Review 去檢查出 Bug，只是那並非 Code Review 主要目的。

**2. 需求變化太快，專案的生命週期短，寫好程式沒有意義，反正過沒多久就拋棄了**

如果是一次性的東西、短期的解決方案，的確不需要太高品質的程式碼，反正用沒多久就要丟了，但至少要 Review 一下這個短期的程式碼有沒有影響到長期程式碼吧? 如果整個專案全都是臨時的程式，那要思考一下自己的部門是否也是一個臨時部門?

**3. 時間太緊急，連 coding 都來不及，哪有時間 Code Reivew**

時間緊急、需求變化太快，其實與 Code Reivew 無關，而是管理的問題，這些問題也不該成為反對 Code Review 的理由；有些人能看到背後價值，堅持做對的事情，有些人因噎廢食，而人與人的不同就在此。

## Code Review 做不起來

大家都知道 Code Review 的好，但以下情況會造成 Code Review 沒效果：

**1. 能力不足**

菜雞互啄，分不清楚程式碼好與壞，Review 大多數力氣放在檢查 Bug、程式碼排版風格，上面講過 Code Review 不是在檢查 Bug，而程式碼排版風格是要用自動化去處理的，這些都是自己處理的事情，不要在 Code Review 浪費大家時間。

對於能力不足的人，可以看看 Code Complete 這類的書，另外公司也該再找人了。

**2. 心態問題**

一種心態是懶，交差了事，甚至為自己爛程式碼辯護，因此更應該推動團隊的 Code Review，讓爛程式碼曝光在更多人面前；另一種懶是自掃門前雪，覺得某功能是其他人負責的，不想花時間去理解，自掃門前雪的風氣一旦蔓延，整個團隊會很被動，不可能做好事情。

另一種心態是自負，有些人喜歡在 Code Review 批評、貶低別人，以顯得自己厲害，有這種心態的人就是所謂「半瓶水」；而被批評的人會產生反抗心態，於是團隊的 Code Review 淪為意氣之爭。

請記住，程式最大的問題就是自負，特別是 Code Review 時，對象應該是程式碼，不是人。提交者要虛心接受建議，因為別人的建議是讓你做的更好；審核者則要用正面的態度提出意見，不要在 Code Review 去質疑別人的能力，因為那是你的隊友。我們不是一段程式碼，而是人。

> We shall do a much better programming job, provided that we respect the intrinsic limitations of the human mind and approach the task as Very Humble Programmers. 
>
> -- Edsger W. Dijkstra

Dijkstra 在獲得圖靈獎的感言中說道，「唯有意識到人類內在思維的極限，並以謙虛的態度寫程式，才能達到更高境界。」

**3. 時間不夠、需求變化太快**

上面提到時間不夠、需求變化太快是管理問題，不是 Code Review 的問題，不要因噎廢食，那工程師具體的解決方式是：

- 你是否思考過，PM 給出這麼多需求，那些是合理的? 那些是不合理的? 拿到需求請先問「為什麼要做?」、「影響範圍(效果)多大?」、「有多少客戶受益?」回答不清楚這些問題，沒有資料佐證就不做，PM 需要做很多調查研究的工作，不是光用想的，或是少數客戶在抱怨就要開需求
- 如果發生這些情況，工程師應該要教育、管理 PM，而非一昧的抱怨，然後淪為「沒關係，反正 PM 要就做給他，有問題也不關我的事，因為我是照規格開發」的擺爛心態，工程師要告訴 PM，「要把握好需求」、「要對放棄的功能與要做的功能感到同樣自豪」，「與其做好一個半成品，不如做半個好產品」，因為做好的半成品只會讓人失望，而半個好產品則會讓人期望
- 別抱怨需求變化太快，因為需求本來就會變，你該思考的是產品的方向是否一直改變? 如果方向不變，那你要培養能判斷未來可能需求的能力，如果方向一直改變，今天要往東，明天要往西，那這個產品也沒什麼出息

當你累得跟狗一樣時，請停下來想想，是否自己的思考、做事方式沒有比狗高明太多?

## 總結

關於 Code Review 無用論的說法，其實都在抱怨其他問題，請不要混為一談。而 Code Review 做不起來的原因，我也提供了解決方法。最後我想說一句比較武斷的話，「沒有 Code Review 的公司都沒必要待，因為那一定是不尊重技術的公司」。

一直用低標準做事情，會成習慣的。

![habbit](/img/code-review/habbit.png)
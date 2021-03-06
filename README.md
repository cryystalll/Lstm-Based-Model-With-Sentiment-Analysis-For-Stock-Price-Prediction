# Lstm-Based Model With Sentiment Analysis For Stock Price Prediction

# 基於LSTM與情感分析之股價預測

<div align=center><img src="/img/p1.jpg" alt="Cover" width="40%"/></div>

## 1.	摘要
```
本次專題實驗聚焦在股票分析議題，使用以LSTM的強化式學習模型為基底的架構進行股票預測，並摻入短時間高頻率之情感分析，實作深度學習預測即時股票價格。
實驗比起驗證時間序列模型的股票預測成果，更關注在如何最大化利用情感分析結果優化預測準確度、定義情感分析與股票價格之間的關聯以及推論影響股票價格可能的原因。
金融市場的股票價格與人為的活動密不可分，是本實驗研究情感分析的出發點，為了多方面的考量實驗因子、從不同角度審視股票分析的問題，
實驗採用股票新聞重點標題與社群媒體的大量文本資料，即時性的監控大眾的情感（SENTIMENT），或稱之為「意向」，來達到完善的預測結果。
第一階段事先準備資料，用API取得歷史股票價格資訊與即時性情感分析文本，對資料集做好相對應的預先處理後，匯入用LSTM+CNN建構之時間序列模型，得到第一階段預測結果。
第二階段實驗交叉比對即時性情感分析與股票價格的關聯性，用深度學習與自然語言處理來分析文本之情感特徵值。
一方面我們保留時間序列變化，另一方面加入情感因素，利用時間對股價的折線圖來比較各種單一方法、複合方法的優缺點 ; 
計算RSME、MAPE來驗證此增強式集成學習演算法的實用性。最後的預測結果能達到27%的優化率，
除了證實時間序列加入情感分析是正確的研究方向，也說明即時性情感分析對股票價格的影響力是不容忽視的。
```
## 2. 模型設計
```
本次研究參考了現有的相關研究，除了選擇平均表現(performance)最好的兩個數據預測演算法lstm(Long Short-Term Memory)、cnn結合，
更著重在如何用情感分析的波動優化現有的預測模式，得到混合式模型股票預測，並期望結果能同時處理時續性預測與保有因應時勢變化的波動性。
股票價格時序分析中，採用LSTM — CNN + LSTM的多層模型。以時間序列分析上來說，由rnn發展而成的lstm(Long Short-Term Memory)擁有相對其他方法的優勢，
它可以有效處理非線性的股票金融資料，學習不同步時間資料之間的相關性，將跨時間步的高維分析降維成低維表徵式；
cnn卷積層可以在序列資料上萃取(extract)特徵(feature)，找出跨時間、不同步資料的特徵。
情感分析（Sentiment Analysis）設計原理則是使用自然語言處理（NLP），python3中的Flair套件有本身的NLP函式庫，也可以輸入字典訓練自己的函式庫，
在使用上也比NLTK精簡，將情感文本輸入模型後即可拆解句型、進行詞彙分析、判斷正負極值，
本專題實驗除了使用本身內建已經訓練好的NLP模型，也有使用自己訓練而成的自然預言處理模型做集成分析，
目的是要找到同時間股票價格（Close price history）與情感極值（Sentiment Polarity)之間的關聯。
最後，將LSTM 與 Sentiment Analysis 的結果做二維迴歸模型（Regression），用梯度下降法（Gradient Decendent）找到最佳解，完成股票價格的預測，用實際資料驗證平均表現。
```
<div align=center><img src="/img/p3.png" alt="Cover" width="40%"/></div>

## 3. 研究方法與步驟
## 1.資料收集與資料預處理
* A. 從Yahoo Finance API 取得Apple公司股票價格
```
利用滑動視窗演算法（Sliding Window），將資料從61日開始，將每日的前60日的60筆資料包裝為此日的訓練資料，直到最後一筆訓練資料為止。目的就是要找尋跨時間資料集之間的關聯。
將每筆資料標準化到（0,1）之間，避免極值（Feature Scaling），最後我們得到每份60筆的 x_train 資料，與當天的價格 y_train 資料。
```
* B. 情感分析語句資料
```
1. 由 Twitter API 取得全球用戶情感分析文字資料（sentence）
2. 由 Finnhub API 取得公司新聞標題文字資料（text)
```
<div align=center><img src="/img/p4.png" alt="Cover" width="40%"/></div>

## 4.訓練模型
* A.使用Flair訓練集與自然語言處理（NLP:Natural Language Processing）
```
解析與篩選文本，得到語句正負極值
```
<div align=center><img src="/img/p5.png" alt="Cover" width="40%"/></div>

```
一段文本中的各個單詞的特性會賦予這個集合一個實數數值，能被情感分析所辨識與計算，生成前述的文字向量。
而Flair使用NLP從文字庫經由深度學習訓練好的模型，能使用在文字的解析上。
```
<div align=center><img src="/img/p8.png" alt="Cover" width="40%"/></div>

* B.使用CNN+LSTM模型
```
輸入資料預處理得到的 x_train（前60天）、y_train（當天）資料訓練cnn+lstm混合模型。將用cnn得到攤平後的資料集傳入LSTM模型，單位元設為128個，input_shape為輸入資料（60,1），經過lstm三個閘門計算後輸出，最後搭配Drop out層，避免過擬合，隨機刪除節點與神經元，只選擇更新部分網路的權重(weight)，將神經網絡模型平均化，設定batch_size = 1、epoch = 1，最後得到預測結果。
```
```
在論文 A hybrid model integrating deep learning with investor sentiment analysis for stock price prediction(NanJingaZhaoWubHefeiWangc,2021) 
提出的方法是先用情感分析資料訓練模型，再輸入歷史股價數據進行時間序列分析，此方法放大情感分析與時間序列分析的優點，平均話兩者的缺點。跟現有的多數方法一樣是長時間的分析。
但參考前一篇情感分析的資料，本專題想實驗短時間內高頻率的情感分析資料與價格曲線的波動性之間的關聯性，證實他們可以修正時間序列分析的誤差。
步驟是先用時間序列資料建立模型，再用短時間的大量情感分析資料做回歸計算與集成學習模型。現有文獻大部分著重於單一模型的股票趨勢預測，如記憶式時間序列分析學習模型，或著重於社群媒體的大眾情感分析。這些單一的模型預測有一定水平的成果，但效果良莠不齊，前者的預測有一定精準度但波動過小，往往無法反映股票價格波動的特性 ;
後者波動幅度過大，單一的情緒事件就會放大誤差，偏離原本的數據，主要問題在於他們的研究多數搜集資料的頻率過低，也因為缺少時間序列預測，只引用情感分析預測大致的價格波動。但情感資料卻是短時間內細微的變化，較好的做法是以高頻率的資料分析明確的時間點，才能最大化情感分析的效益。
```
<div align=center><img src="/img/p2.png" alt="Cover" width="40%"/></div>

## 5.Results and Accuracy
```
     CnnLstm:        {Rsme:1.71, Mape:0.98}
CnnLstm+Sentiment:   {Rsme:0.77, Mape:0.39}
```
```
用前五天股價分別訓練兩種模型，預測後兩天的結果展示，可以證實加入情感分析（黃線）能有效降低預測誤差
藍線：真實股票價格
紅線：cnn+lstm
黃線：((cnn+lstm)+sentiment)) +regression
```
<div align=center><img src="/img/p7.png" alt="Cover" width="40%"/></div>

```
Cnn+Lstm混合模型近四個月股價實際值與預測值比較折線
```
<div align=center><img src="/img/p11.png" alt="Cover" width="40%"/></div>


## 6.七天內隨著時間取得高頻率股價資料與情感分析資料之間的關聯性展示

```
七天內Apple股票收盤價高頻資料折線圖
```
<div align=center><img src="/img/p9.png" alt="Cover" width="40%"/></div>

```
七天內Apple股票情感分析高頻資料折線圖
```
<div align=center><img src="/img/p10.png" alt="Cover" width="40%"/></div>

```
依序為真實股價、cnn+lstm混合模型預測值、情感分析極值、混合模型加上情感分析優化之預測值
```
<div align=center><img src="/img/pp.png" alt="Cover" width="40%"/></div>

















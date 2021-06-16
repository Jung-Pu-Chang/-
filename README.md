#  電子類股 - 進出場時機預測

## 成員: 劉乃嘉、陳侑杰、張容溥

## 目標: 建立進出場預測模型 & 挑出重要指標


## 第一階段: 挑k個最具有樣本代表性的股票(by GMM分群)

1. 資料集說明(來源: TEJ): 

   以季為單位，抓取2016/1/11~2018/9/7期間(股市穩定區間)，共11季

   共蒐集402支電子類股股票資料，共4,539筆
   
   使用股票間標準一致的指標:
   
   每股淨值(F)－TSE公告數、營業毛利率、營業利益率、稅後淨利率、ROA－綜合損益、ROE－綜合損益、常續性EPS

   ![image](https://user-images.githubusercontent.com/79561258/122146517-ccfe6e80-ce89-11eb-89d9-9a1aab510abc.png)


2. 資料清洗:
   
   因NA值極少，且無法確定其來源是否為TEJ錯誤，故一律刪除

   因本次的進行進出場分析是以投資為目的，表現較差的股票不在投資名單中，故只保留ROA 11季總和>0的股票，共315支電子類股
   


3. GMM分群:

   因傳統硬分群(如:K-means)會無法分辨出落在邊界上的點，故使用軟分群來觀察每個點在每群的機率

   又因本次想藉由分群去抓出美群最具樣本代表性的股票，藉此回推整個電子類股(母題)，故選擇以統計為主的GMM演算法

   透過BIC指標(如下圖)決定本次分兩群

   由分群結果發現群一各指標均值，皆小於群二，推測群二是表現較好的電子類股
   
   藉由GMM的最大分群機率挑出最具樣本代表性的股票，群一代表為敦陽科，群二代表為旭隼
   
   ![image](https://user-images.githubusercontent.com/79561258/122143353-e69cb780-ce83-11eb-9cd8-807061c5bc6b.png)



4. 第一階段結果:
   
   挑出敦陽科、旭隼兩支最具樣本代表性的股票，有足夠力度回推整個電子類股，故後續針對兩支股票進行分類預測&指標分析
   
   本次特別採用統計抽樣的概念，取代將所有股票都進行分類預測，以避免預測效果不佳的情形

## 第二階段: 建立進出場分類預測模型(by 隨機森林)

1. 資料集說明(來源: TEJ): 

   抓取2016/1/11~2018/9/7期間(股市穩定區間)，旭隼(672筆)、敦陽科(651筆)每日各指標資料
   
   應變量: 買進、賣出、持有，共3個類別，分別為1~3
   
   自變量: 使用股票間標準一致的指標，如下
   
   投信持股率、自營持股率、合計持股率、投信週轉率、自營商週轉率、外資週轉率、當沖比重資券互抵、現股當沖比重、現沖加當沖比重、本益比TSE、股價淨值比TSE、股利殖利率TSE、nas收盤、加權收盤、費半收盤

   ![image](https://user-images.githubusercontent.com/79561258/122146696-15b62780-ce8a-11eb-9faa-5b5be0542f61.png)
   
   ![image](https://user-images.githubusercontent.com/79561258/122146838-531ab500-ce8a-11eb-9f41-7cc627909c0f.png)



2. 資料清洗:
   
   因NA值無法確定其來源是否為TEJ錯誤，故一律刪除

   因 NAS & 費半指標預設為美國日期，所以轉成與其對應的台灣日期



3. 建立隨機森林模型
   
   因本次為次序資料做分類預測，且後續須針對指標進行說明，故傳統的自迴歸(ARIMA系列)、神經網路不太適合
   
   故捨棄使用次序相關模型(後續建模不考慮次序)，考慮常見分類模型，包含 XGB SVM 隨機森林
   
   將兩支股票資料各自建立模型，最終皆以隨機森林的8成準確度最高，F1 score也以隨機森林最高。
  
  
  
 4. 分類預測結論
    
    產出預測模型，未來僅需將上述自變量指標投入，即可知道該買進、賣出，還是持有
    
    有助於非從業人員投資股票

## 第三階段: 重要指標分析(by 特徵工程 + Apriori)

1. 資料集說明(來源: 模型產出): 

   延續第二階段的指標，如下: 
   
   投信持股率、自營持股率、合計持股率、投信週轉率、自營商週轉率、外資週轉率、當沖比重資券互抵、現股當沖比重、現沖加當沖比重、本益比TSE、股價淨值比TSE、股利殖利率TSE、nas收盤、加權收盤、費半收盤

   依照所有指標的特徵工程結果，分成重要(1)與不重要(0)
   
2. 特徵工程(整理成關聯格式資料)

   觀察各指標在各種特徵工程方法中的表現
   
   類似的特徵工程會擇一使用，如: ridge & lasso 擇一，避免關聯分析的權重不同
   
       隨機森林以大於整體GINI平均值，來判定屬於重要指標
   
       XGB以大於整體Gain平均值，來判定屬於重要指標
   
       SVM以大於整體AAD平均值，來判定屬於重要指標
   
       ridge迴歸以beta大於整體平均值，來判定屬於重要指標
   
       fisher score以落在前8名，來判定屬於重要指標

   ![image](https://user-images.githubusercontent.com/79561258/122148413-0dabb700-ce8d-11eb-8f93-92ba19333926.png)

3. 關聯分析(by Apriori): 擷取重要指標

   觀察出現頻率(coverage)可以發現，本益比TSE、股利殖利率TSE最高，故為最重要的兩個指標

   觀察交互影響(JC)可以發現，以下五種組合，只要當中一個指標被列為重要，剩餘指標也一定會重要:

       nas收盤 & 費半收盤 

       投信週轉率 & 自營持股率 & 自營商週轉率 

       投信持股率 & 加權收盤股利殖利率 & TSE合計持股率 & 自營持股率 & 股價淨值比TSE & 費半收盤

       投信持股率 & 現沖加當沖比重 & 現股當沖比重 & 合計持股率  

       自營持股率 & 自營商週轉率 & 合計持股率

影片-前導版: https://www.youtube.com/watch?v=-ikwYKZIQtk 

影片-完整版: 

會議記錄: https://www.notion.so/32e24a35c85242aaba895f4611d95ea4

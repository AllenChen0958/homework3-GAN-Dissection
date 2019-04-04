 ISA525700 Computer Vision for Visual Effects<br/>Assignment 3: GAN-Dissection<br/>Team 19
===

## Abstract
- GAN Dissectiong使用視覺化探討GAN內部不同layer中的每個神經元在幹什麼，透過找出關聯性可以在特定位置透過提高特定神經元的輸出來產生特定特徵於圖片中，甚至能找到壞掉的神經元，移除後改善輸出品質。
- 本次報告會介紹基本GAN-Dissection這篇論文的內容，並且實際用來分析GAN model，討論神經元的一些有趣現象，最後與其他用於移除場景中特定物件的方法進行比較，並在最後做一些跟影像辨識有關的延伸討論。
- 本文結論中發現在Dissect、Contextual Attention、opencv(NS)、opencv(TA)效果評比，以Dissect輸出影像最佳。

Keyword: GAN Dissection, GANpaint
## Table of Contents
1. [Introduction](#Introduction)
2. [GAN Dissection論文要點](#GAN-Dissection論文要點)
3. [Generate images with GANPaint](#Generate-Images-with-GANPaint)
4. [Dissect GAN Model and Analyze](#Dissect-GAN-Model-and-Analyze)
5. [Compare Method](#Compare-Method)
6. [總結](#總結)
7. [延伸補充](#延伸補充)
8. [Reference](#Reference)
## Introduction
- 生成對抗網路(Generative Adversarial Networks, GAN)，近年來於實務應用中有許多令人矚目的成果，並且有很多新的GAN出現，在樣本的品質(Sample quality)與訓練穩定性(Training stability)上均有得到大幅改善，但是卻沒有可視化或理解。
- GAN有許多問題，例如如何在內部代表我們視覺世界?如何影響GAN學習?解決這些問題有助於發現更多新的解析與建立更好的模型。因此GAN-Dissection的作者提出一個分析框架，對GAN在神經元、對象與場景等進行可視化與解析。
- 首先使用基於分割的網路剖析方法識別與對象概念關聯度高的一組神經元(Unit)。然後透過測量干擾輸出特徵像(Feature map)的能力，量化可解釋的因果關係。
- 此外透過觸發神經元於背景的指定位置，還可以將特定特徵插入圖片中特定位置。
- 甚至透過對每個神經元進行分析，可以找出哪些神經元是會產生artifact的壞的神經元，移除後可以改善Generator的輸出品質。

## GAN Dissection論文要點
- 可以依照我們期望，生成出期望的場景。
- 提出可視化方式，體驗GAN神經元在學習甚麼?
- 辨識哪些神經元輸出結果不好，或辨識到會產生扭曲的神經元， 均可-將它設為0。
- 具有可互動框架，自適性調整字元，產生新增或移除的類別(Class)功能。
- 某項類別對應神經元(Unit、Channel、Feature map)，代表意義?
- 每個物件(Object)對應哪些神經元。
- 神經元在每個場景資料集(Scene dataset)學習到甚麼特徵?可觀察出GAN。
- 是否存在Bias。

可觀看影片近一步了解[GANDissection](https://openreview.net/pdf?id=Hyg_X2C5FX)概念。

### GANpaint

[GANPaint](http://gandissect.res.ibm.com/ganpaint.html?project=churchoutdoor&layer=layer4) 是一套APP，左邊的各個按鈕如Tree、grass、door、sky、cloud..對應到20組神經元。GANpaint是直接啟動或停用指定位置深度學習網路的神經元集合，產生增加或移除特定物件，如移除樹、增加草地。


### 解開(Dissection)
![](https://i.imgur.com/218LtQe.png)


在圖片生成過程，首先輸入雜訊z，再生成圖片x。因此z是經過一層層的r(layer)生成x，逐漸轉成圖片。
![](https://i.imgur.com/JP19Gnf.png)

令其中一層的feature maps為 r ，u為其中一個unit(其負責feature maps 其中一個channel)，u∈U,p∈P(畫面的pixels subset)，提出其中一個unit在特定pixels subset產生的feature map，透過upsample resize成照片大小，圈出activate大於特定閾值與產生圖片疊合，就能產生下面的截圖。

![](https://i.imgur.com/5YEcx0w.png)
<br/>圖為layer=3 unit#54的其中一張疊合後圖片

與原圖片對特定class進行segmentation後的圖片可以進行IoU計算出該unit與特定class的關聯性，公式如下：

![](https://i.imgur.com/DMlvIGF.png)

這裏只要暫時理解為越接近1越好，如下圖：

![](https://i.imgur.com/nwqfcbY.png)

### 介入(Intervention)
通過Dissection，可以求出任一unit與class的IoU，並排出對該class有最高IoU的unit集合。
如果想要在特定區域增加或移除特定class，例如移除樹或是增加天空，先透過把feature maps 切成對該 class 有高關聯性的unit集合以及較無關的unit集合。

![](https://i.imgur.com/LLfS6Oh.png)

再透過下面公式：
![](https://i.imgur.com/L92vvPv.png)

想增加就設為常數c，移除則設為0，就可以在圖片特定Pixels處增加或減少特定class了。
![](https://i.imgur.com/E88pRXV.png)

## Generate Images with GANPaint
1. Example Image add/remove tree
以下是使用[GANPaint](http://gandissect.res.ibm.com/ganpaint.html?project=churchoutdoor&layer=layer4)對別墅做增加或移除樹(Tree)的結果截圖。

|original|add|remove|
|--|--|--|
|![](https://i.imgur.com/CLyWxbs.png)|![](https://i.imgur.com/BBXwP4t.png)|![](https://i.imgur.com/FqjfMq9.png)|

## Dissect GAN Model and Analyze
本次使用的GANDissect可以提供兩個功能，一個功能是解析出不同層裡的unit分別代表對圖片中哪些物件造成影響，另一個則是是可以透過在產生圖片時消除特定unit中使其產生移除特定物件的效果。
### 執行過程截圖
![](https://i.imgur.com/4u1S3xz.png)

### server 截圖
下圖為跑完GANDissect程式後可開啟server部分截圖，living room共dissect四層，前三層3,6,9在一個資料夾，在 ~/dissect.html 頁面可以看到，第4層在另一個資料夾透過另一個 server 的 ~/edit.html 呈現，我們也跑了church的GAN model的第四層，呈現在 ~client/index.html，用其進行 Intervention實驗 。
- #### dissect.html截圖
![](https://i.imgur.com/3cuU4tx.jpg)

- #### layer 3, 6, 9 Unit Class Distribution

layer 3
![](https://i.imgur.com/TIxyteO.png)

layer 6
![](https://i.imgur.com/rlU0KvR.png)

layer 9 
![](https://i.imgur.com/jPgvw1s.png)
3, 6, 9 層可定義出class的unit數量比例分別是:220/512 , 227/512 , 53/128，其他IoU低於0.05無法確定與class的關係。
- #### layer 3其中幾個unit截圖

![](https://i.imgur.com/mtjitMd.jpg)
令其中一層的feature maps為 r ，u為其中一個unit(負責feature maps 其中一個channel)，u∈U,p∈P(畫面的pixels subset)，提出其中一個unit在特定pixels subset產生的feature map，透過upsample resize成照片大小，圈出activate大於特定玉質與產生圖片疊合，就能產生上述截圖。
- #### layer4 in edit.html with Intervention
![](https://i.imgur.com/bRflHsM.jpg)
透過找出特定點對應的feature maps可以確定與各個unit關聯性，可以發現當目標點設定在沙發處的時候，跟沙發有關的unit確實造成較多影響。
- #### layer4 in index.html
![](https://i.imgur.com/xC7OGdm.jpg)
透過先ablate tree-iou最高的前40個unit再對紅色標記範圍區域添加sky-iou最高的前70個unit，可以產生類似天空之城的效果。
### 分析討論

#### 1. 每一層產生的unit類型不同
- 現象：
    layer 3
![](https://i.imgur.com/TIxyteO.png)
![](https://i.imgur.com/wPmsqW7.png)
![](https://i.imgur.com/kfDQbda.png)
![](https://i.imgur.com/jlTxFrC.png)
該層多數的unit主要能將，像是天花板、窗戶或是沙發...等，這種大型且外表輪廓簡單的居家配備整體匡出。
layer 6
![](https://i.imgur.com/rlU0KvR.png)
![](https://i.imgur.com/8xTY9a7.png)
![](https://i.imgur.com/0kzpiip.png)
![](https://i.imgur.com/RivF6R4.png)
該層(layer6)可解釋unit種類較之前的層數(layer3)多了11個類別，且該曾多數的unit已經可將各個家具不同部位匡出(EX: 沙發的頂部)，顯示該層已可將家具進行語義分割，唯該次的有些大型輪廓開始出現雜訊(如上圖)。對於輪廓簡單的家庭配件來說，其IoU分數未必勝過較淺的層數。
layer 9 
![](https://i.imgur.com/jPgvw1s.png)
![](https://i.imgur.com/yaeHb58.png)
![](https://i.imgur.com/6jvLIxk.png)
![](https://i.imgur.com/izFom1j.png)
該層已經可以匡出更細部的紋理以及材質，而家庭配件上可能會有許多不同紋理以及材質，因此可辨別的unit種類大幅減少，然而因為可匡出不同紋理，在材質的辨別種類上是勝過其他前面的層數。

- 討論：
這結果跟一篇很有名的論文[Visualizing and understanding convolutional networks](https://cs.nyu.edu/~fergus/papers/zeilerECCV2014.pdf)，有相近的成果，隨著層數上升，units處理的東西越複雜，從下圖神經網路的結構上來看，是十分合理的。
![](https://i.imgur.com/w04hwoQ.png)
底層各個專注於產生特徵單純的feature maps，傳到更高層後，高層網路透過對前一層特徵單純的feature maps進行組合形成更高級更複雜的feature maps，所以底層units中只能找到對天花板這種特徵單純的class造成影響的unit，但隨著layer往後，因為結合前一層不同unit產生的簡單特徵的feature maps，開始出現能影響特定class的unit，但再更後層可定義出class關聯的unit會變少，比如把多個不同unit產生但都影響sofa的feature maps會被一個產生sofa概念的unit結合成一張feature map，再之後會出現結合sofa和floor的更複雜的unit，而這種unit因為過於複雜也已經無法定義與class的關聯，只有一些無法結合其他feature map產生破碎材質的unit被保留。
簡而言之，底層我們推測無法被定義是因為特徵太單純，而高層的unit無法被定義是因為特徵太複雜，就像是最高層的最後unit產生的feature map就是一張圖片，你沒辦法定義圖片跟什麼有關，因為他跟每個東西都有關。


#### 2. Intervention效果在不同位置會有所不同
- 現象：
![](https://i.imgur.com/3nybpZY.jpg)
我們複製了前70個對門IoU最高的unit，添加在圖片紅色範圍內，目的是添加門在圖片上，可以發現右上角添加失敗跟左圖對照仍然相同，中間則添加成功，長出了一個看起來蠻合理的門。

- 討論：
推測可能是受到不能定義出跟特定class有關的一些表現抽象概念的units集合的影響，這些unit產生的特徵，在更高層會被拿來與我們觸發的跟門最有關的那些unit產生的特徵會組合起來，可能這裡面其中一個特徵就是門不能長在天空上這種概念，當這種概念與跟門有關的unit產生的特徵在更高層被結合後，就可能使得其產生門的效果受到抑制。


## Compare Method
> - Generative Image Inpainting with Contextual Attention
> - opencv inpainting

### Generative Image Inpainting with Contextual Attention

因傳統GAN或是其他方法生成未知區域的圖片，容易在未知區域周圍產生扭曲或模糊，本文假設原因為大多方法僅在未知周圍取樣，較無法參考圖片中比較遠的區域來生成並填補未知區域。因此本文提出利用Contextual Attention的機制，使 __生成未知區域時，可參考整張照片__

該方法主架構如下：
![](https://i.imgur.com/r1GzhMd.png)
首先在前半部的Corase Network，為傳統的編碼器(encoder)與解碼器(decoder)搭配起來的架構。其產生的圖片為模糊的板本。

接下來再將較模糊的結果丟入refinement network產生較清晰的版本。Refinement network的架構如下：
![](https://i.imgur.com/XFStIqc.png)

refinement network可分為兩條路徑，其中下面一條的結構與前面Corase Network相同。比較特別的部分為其增加的Contextual Attention Layer，可讓中間mask的模糊區塊參考到離它較遠的區域，將其視覺化後，如上圖的attention map，代表著模糊區域該如何對應到整張圖片，需參考對照到Attention Map Color Coding。
舉例來說attention map中間有一片粉紅色區域(草+石子路)，可對應到Attention Map Color Coding的左下角(也是粉紅色區域)，也就代表中間模糊的區塊其實長得像整張照片的左下角。

attention layer架構圖如下：
![](https://i.imgur.com/dFefXoB.png)
其大致作法為，將非mask區域都分割成固定size的patch，再用這些patch跟mask區域做convolution。產生出的attention map再利用非mask區域做deconvolution。

### Opencv
opencv library有提供兩種方法

第一種(cv.INPAINT_NS)使用的是在Navier-Stokes, Fluid Dynamics, and Image and Video Inpainting中提出的方法，會不斷比對未知區域周圍輪廓的梯度，並且將未知點納入輪廓當中，該方法的提出是基於流體力學以及偏微分。

第二種
(INPAINT_TELEA)為使用An Image Inpainting Technique Based on the Fast Marching Method提出的方法，主要概念是利用mask像素點附近的區域來生成mask裡的未知區域。因此該方法如果是大範圍的inpainting則效果較差，該方法適合長線條或是小區塊的mask。






|Method: |Original|result|
|:--:|--|--|
|Dissect|![](https://i.imgur.com/pwhBX94.png)|![](https://i.imgur.com/0J7YXxI.png)|
|Contextual Attention|![](https://i.imgur.com/pwhBX94.png)|![](https://i.imgur.com/sdHbyom.png)|
|opencv(NS)|![](https://i.imgur.com/pwhBX94.png)|![](https://i.imgur.com/RDU7wXT.png)|
|opencv(TELEA)|![](https://i.imgur.com/pwhBX94.png)|![](https://i.imgur.com/vRNAofA.png)|

 


### 比較分析
#### 比較表格
||Dissect|Contextual Attention|opencv(NS)|opencv(TA)|
|--|--|--|--|--|
|自由選擇input照片|x|o|o|o|
|效果排名|1st|2nd|3rd|3rd|
|速度|0.0756sec|0.838sec|0.037sec|0.033sec|
|遮罩依賴|x|o|o|o|
|需要pre-train|o|o|x|x|
#### 討論
- GAN-Dissect在這次實驗，輸入來源是設定隨機產生，相較於其他方法則可以自由輸入圖片，但是其實只要在模型前面加入encoder則可以透過encoder產生自由輸入圖片的featuremap，也可以用來輸入decoder，這樣就能達成跟其他幾種方法一樣的效果。
- Contextual Attention 透過先將未知區域還原成較模糊，再將該區域與整張圖比對，如果剛好範圍不大，且該部分區域是與照片任一處相似的話，會處理的比較好。以上方照片為例，Contextual Attention消除左邊的樹後，還可生出類似窗戶的結構(Dissect則是一片黑)，推測是因為該區域可以仿製其右方的窗戶，或是塔樓二樓的窗戶。但消除右方的樹時，因為樹的體積太大，消除的部分沒有特別像哪個區域，以圖片結果推論右方消除的樹區域是想模仿左方的塔樓以及天空，故造成扭曲的現象。

- Opencv在本次實驗是不需要訓練的方法，單純靠數學計算，來填補未知區域，然而因為本次消除樹的區域過大，opencv這類方法多須仰仗未知區域周圍的區域進行計算填補，因此未知區域過大會讓這類方法效果較差

- 總結：當有遇到小污點或是缺陷的圖，且運算資源有限的時候，適合用opencv的方式來消除；當有小區域模糊或被屏蔽，且整張圖相似結構多時，適合用Contextual Attention來消除。

## 總結
- 這次因為GAN-Dissection比較陌生，所以花比較多時間做相關論文的閱讀，也因此比較沒有對模型進行trace code並修改做出一些更有創意的嘗試，而是更多的著墨在使用這個GAN-Dissection後分析出的GAN model有哪些特別的現象。
- model分析的部分只著墨兩個現象是因為這方面的整理比較明顯，但其實論文有提到一些比方說不同model training出來的unit種類不同，例如產生客廳場景的GAN model會有更多與沙發相關的unit能被定義出來，而產生廚房場景的則沒有；以及可以同unit能夠產生出來的沙發不論是顏色或材質都可以有很大的差異性，推測有學出概念的特徵，這些我們因為成果較難呈現就沒有放在分析內，有興趣的可以自己延伸閱讀或與我們討論。
- 這次助教要求用GAN-Dissection與其他inpainting的方法進行比較，其實我們也遇到不少困難，在不去特別調整程式架構下對GAN-Dissection輸入特定照片的特徵不是容易的事情，所以我們是以GAN-Dissection產生的圖片丟到其他比較方法裡，而非選用自己的照片。

## 延伸補充
GANDissection 的移出功能一種是標記特定區域移除指定unit集合產生移除特定class的效果，但同時透過完全去除所有位置特定unit集合，還能產生把畫面中所有樹移除的效果，但比較方法都是需要提供mask，也就是需要標記哪些位置需要移除，如果要達成移除畫面特定class的效果，應該還要在前面加入辨識系統，自動產生mask，最後也附上一些辨識系統的相關整理。


### [Selective Search for Object Recognition]( http://www.huppelen.nl/publications/selectiveSearchDraft.pdf)

- 五隻貓，如何辨識出其中一支kitten貓呢?
 
![](https://i.imgur.com/7GnDe59.png)


- 一種解決方案是地毯式搜尋object，但是這樣需要處理數十個成千上萬的可能object。
 
![](https://i.imgur.com/Nchw8HP.png)

-	另一種解決方案是scanning detector，成本較低，辨識速度也比較快。

![](https://i.imgur.com/MJMF7w8.png)

- 如果我們可以在物體識別前，正確分割圖片，就可使用segmentations作為候選object。
 
![](https://i.imgur.com/gbGY0YA.png)

Selective Search for Object Recognition是運用Selective Search演算法在圖片中尋找可能的object，經過訓練後，送入SVM model，再持續物體識別訓練。

![](https://i.imgur.com/CgXe12m.png)
 
-	圖a:餐桌上很多碗、湯匙、盤子，這是2D影像表示3D視界，需要有編碼註記上、下、後面等階層。
-	圖b:兩隻貓不同顏色，但紋理相同。
-	圖c:蜥蜴與葉子顏色相同，但紋理不同。
-	圖d:因為車輪附著於車體，我們可以理解車輪是汽車一部份。

![](https://i.imgur.com/c1dIVUP.png)
 
將影像透過分割方法，切割很多區塊(region)，並依顏色相似度、紋理相似度、大小相似度、吻合相似度，進行合併(grouping)，讓很多region融合成大region。

### 市面上成熟的影像辨識應用
- ADAS先進駕駛影像辨識系統
提到無人自駕車，莫過於眾所注目的美國的特斯拉。電腦透過CV(Computer Vision)，可以辨識外界真實世界的各種物體，提高自駕車安全性，這項科技稱為ADAS先進駕駛影像辨識系統。
[![](http://img.youtube.com/vi/714ryaBtmLo/0.jpg)](http://www.youtube.com/watch?v=714ryaBtmLo "")
- 形色
以辨識檸檬做為實驗，發現照片中的檸檬，必須是角度偏斜才能辨識成功，若為角度為正，經由內部Object辨識排序，與香菇草近似度最高，因此誤判為香菇草。

| 辨識失敗 | 辨識成功 |
|--|--|
|![](https://i.imgur.com/CGaFQ90.jpg)|![](https://i.imgur.com/K59kmDT.jpg)|

## Reference
1. https://gandissect.csail.mit.edu/papers/gandissect-NECV-2018-abstract.pdf
2. https://www.semanticscholar.org/paper/GP-GAN%3A-Towards-Realistic-High-Resolution-Image-Wu-Zheng/110ee8ab8f652c16fcc3bb767687e1c695c2500b
3. https://blog.csdn.net/carson2005/article/details/6844025
4. https://pdfs.semanticscholar.org/622d/5f432e515da69f8f220fb92b17c8426d0427.pdf
5. https://zhuanlan.zhihu.com/p/24833574?fbclid=IwAR00o7VXL3TFpc6FGfbah2hwKgEUYSGesIxXmkilBPjCPbiSMPDcDVuU6WU
6. https://cs.nyu.edu/~fergus/papers/zeilerECCV2014.pdf
7. https://medium.com/@xiaosean5408/gan-dissection%E7%B0%A1%E4%BB%8B-visualizing-and-understanding-generative-adversarial-networks-37125f07d1cd
8. https://www.sciencedirect.com/science/article/abs/pii/S0169743916304907
9. https://pdfs.semanticscholar.org/622d/5f432e515da69f8f220fb92b17c8426d0427.pdf
10. http://www.math.ucla.edu/~bertozzi/papers/cvpr01.pdf

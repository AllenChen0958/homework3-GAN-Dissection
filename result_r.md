 ISA525700 Computer Vision for Visual Effects<br/>Assignment 3: GAN-Dissection<br/>Team 19
===

## Abstract
GAN Dissection是一種透過深度學習網路的神經元，以神經元”已知”或”記憶”特徵像，插入圖片中，完成場景組成效果。因此GAN經過訓練，可以辨識外界真實景物。

Keyword: GAN Dissection, GANpaint

## Introdcution
- 全球資訊產業趨勢
根據2017年七月[經濟日報](https://udn.com/news/story/6811/3250155)指出，經過Gartner評估調查，在2018年Q2全球電腦銷售量為6,210萬台。顯示電腦視覺市場需求，競爭非常激烈!
![](https://i.imgur.com/TQAjCXa.png)

- ADAS先進駕駛影像辨識系統
提到無人自駕車，莫過於眾所注目的美國的特斯拉。電腦透過CV(Computer Vision)，可以辨識外界真實世界的各種物體，提高自駕車安全性，這項科技稱為ADAS先進駕駛影像辨識系統。
[![](http://img.youtube.com/vi/714ryaBtmLo/0.jpg)](http://www.youtube.com/watch?v=714ryaBtmLo "")

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

- APP:形色
本組使用手機APP形色，辨識檸檬做為實驗，發現照片中的檸檬，必須是角度偏斜才能辨識成功，若為角度為正，經由內部Object辨識排序，與香菇草近似度最高，因此誤判為香菇草。



| 辨識失敗 | 辨識成功 |
|--|--|
|![](https://i.imgur.com/CGaFQ90.jpg)|![](https://i.imgur.com/K59kmDT.jpg)|


## Method
回顧相關模型原理。

### Method: GAN
生成對抗網路(Generative Adversarial Networks, GAN)，近年來於實務應用中有許多令人矚目的成果，並且有很多新的GAN出現，在樣本的品質(Sample quality)與訓練穩定性(Training stability)上均有得到大幅改善，但是卻沒有可視化或理解。GAN有許多問題，例如如何在內部代表我們視覺世界?如何影響GAN學習?解決這些問題有助於發現更多新的解析與建立更好的模型。因此我們提出一個分析框架，GAN在單元、對象與場景等進行可視化與解析。首先使用基於分割的網路剖析方法識別與對象概念關聯度高的一組解釋單元(Unit)。然後透過測量干擾輸出特徵像(Feature map)的能力，量化成可解釋的因果效應，透過這些單位與週圍環境的背景，將追蹤特徵像插入新的圖片中。從不同層(layer)比較下產生學習、模型與資料與內部顯示，以及透過定位與偽影像單元，來改進GAN，以交互方式操控場景的特徵像。電腦可以使用GAN模型產生兩種繪製場景方式，第一種用它已認知的物體組成場景，另一方式則為記憶一種圖像複製進入場景。



### Method: CNN 
CNN model是具有空間關聯性，每個feature map皆對應到前面的feature map。

![](https://i.imgur.com/z6z0DIK.png)


## GAN Dissection論文要點:
- 可以依照我們期望，生成出期望的場景。
- 提出可視化方式，體驗GAN神經元在學習甚麼?
- 自動辨識哪些神經元輸出結果不好，或辨識到會產生扭曲的神經元， 均可-將它設為0。
- 具有可互動框架，自適性調整字元，產生新增或移除的類別(Class)功能。
- 某項類別對應神經元(Unit、Channel、Feature map)，代表意義?
- 每個物件(Object)對應哪些神經元。
- 神經元在每個場景資料集(Scene dataset)學習到甚麼特徵?可觀察出GAN。
- 是否存在Bias。

可觀看影片了解[GANDissection](https://openreview.net/pdf?id=Hyg_X2C5FX)概念。
[![](http://img.youtube.com/vi/yVCgUYe4JTM/0.jpg)](http://www.youtube.com/watch?v=yVCgUYe4JTM )

### GANpaint

GANpaint 是一套APP，左邊的各個按鈕如Tree、grass、door、sky、cloud..對應到20組神經元。GANpaint是直接啟動或停用深度學習網路的神經元集，訓練產生增加或移除影像。GANpaint讓電腦可以使用GAN模型產生兩種繪製場景方式，第一種用它已認知的物體組成場景，另一方式則為記憶一種圖像複製進入場景。




### 解開(Dissection)
![](https://i.imgur.com/218LtQe.png)


在圖片生成過程，首先輸入雜訊z，再生成圖片x。因此z是經過一層層的r(layer)生成x，逐漸轉成圖片。
![](https://i.imgur.com/JP19Gnf.png)

令其中一層的feature maps為 r ，u為其中一個unit(其負責feature maps 其中一個channel)，u∈U,p∈P(畫面的pixels subset)，提出其中一個unit在特定pixels subset產生的feature map，透過upsample resize成照片大小，圈出activate大於特定玉質與產生圖片疊合，就能產生下面的截圖。

![](https://i.imgur.com/5YEcx0w.png)
圖為layer=3 unit#54的其中一張疊合後圖片

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

## Generate images with GANPaint
1. Example Image add/remove tree
以下是使用GANPaint對別墅做增加或移除樹(Tree)的結果截圖。

|original|add|remove|
|--|--|--|
|![](https://i.imgur.com/CLyWxbs.png)|![](https://i.imgur.com/BBXwP4t.png)|![](https://i.imgur.com/FqjfMq9.png)|

## Dissect any GAN model and analyze what you find
本次使用的GANDissect可以提供兩個功能，一個功能是解析出不同層裡的unit分別代表對圖片中哪些物件造成影響，另一個則是是可以透過在產生圖片時消除特定unit中使其產生移除特定物件的效果。

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
#### 分析討論

1. 每一層產生的unit類型不同
現象：
- layer 3
![](https://i.imgur.com/TIxyteO.png)

![](https://i.imgur.com/wPmsqW7.png)

![](https://i.imgur.com/kfDQbda.png)

![](https://i.imgur.com/jlTxFrC.png)

該層多數的unit主要能將，像是天花板、窗戶或是沙發...等，這種大型且外表輪廓簡單的居家配備整體匡出。
- layer 6
![](https://i.imgur.com/rlU0KvR.png)

![](https://i.imgur.com/8xTY9a7.png)

![](https://i.imgur.com/0kzpiip.png)

![](https://i.imgur.com/RivF6R4.png)



該層(layer6)可解釋unit種類較之前的層數(layer3)多了11個類別，且該曾多數的unit已經可將各個家具不同部位匡出(EX: 沙發的頂部)，顯示該層已可將家具進行語義分割，唯該次的有些大型輪廓開始出現雜訊(如上圖)。對於輪廓簡單的家庭配件來說，其IOU分數未必勝過較淺的層數。

- layer 9 
![](https://i.imgur.com/jPgvw1s.png)

![](https://i.imgur.com/yaeHb58.png)

![](https://i.imgur.com/6jvLIxk.png)

![](https://i.imgur.com/izFom1j.png)

該層已經可以匡出更細部的紋理以及材質，而家庭配件上可能會有許多不同紋理以及材質，因此可辨別的unit種類大幅減少，然而因為可匡出不同紋理，在材質的辨別種類上是勝過其他前面的層數。

### Discuss

這結果跟一篇很有名的論文[Visualizing and understanding convolutional networks](https://cs.nyu.edu/~fergus/papers/zeilerECCV2014.pdf)，有相近的成果，隨著層數上升，units處理的東西越抽象，從下圖神經網路的結構上來看，是十分合理的。
![](https://i.imgur.com/w04hwoQ.png)
底層各個專注於產生特徵單純的feature maps，傳到更高層後，高層網路透過對前一層特徵單純的feature maps進行組合行程更高級更抽象的feature maps，所以底層unit只能對天花板這種特徵單純的class造成影響。


2. Intervention效果在不同位置會有所不同
現象：
![](https://i.imgur.com/3nybpZY.jpg)
這張圖的目標是在紅色範圍內添加門，可以發現右上角添加失敗跟左圖對照仍然相同，中間則添加成功。



## Compare method
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
其library使用方法為An Image Inpainting Technique Based on the Fast Marching Method，主要概念是利用mask像素點附近的區域來生成mask裡的未知區域。因此該方法如果是大範圍的inpainting則效果較差，該方法適合長線條或是小區塊的mask。



|Original|Dissect|Contextual Attention|opencv|
|--|--|--|--|
|![](https://i.imgur.com/pwhBX94.png)|![](https://i.imgur.com/0J7YXxI.png)|![](https://i.imgur.com/sdHbyom.png)|![](https://i.imgur.com/dMmrHSq.png)|



### 備註


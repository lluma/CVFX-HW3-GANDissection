# CVFX HW3 Report  - Team 16 

### 1. Generate images with GANPaint
1. Add object
    * add door - 在保有牆壁結構下成功新增一個門
    <br>![](https://i.imgur.com/jzY0mrc.png)
    ![](https://i.imgur.com/jXTOxXI.png)

2. Remove object
    * remove door - 移除後直接複製左側的門，雖維持真實性卻也喪失操作意義
    ![](https://i.imgur.com/Z4cpoWR.png) 
    ![](https://i.imgur.com/H5lkhD1.png)

3. Extra test: 如果移除畫面中最主要物件
    
    ![](https://i.imgur.com/PdjB0q4.png)
    ![](https://i.imgur.com/XY4LTsT.png)

    如上圖，試圖移除天空的所有雲朵時，最後仍會保留一些雲，而無法全部移除。
可推得原本雲朵佔比太高，在無法完美預測其內容下，會選擇保留一些以維持真實感。

    ![](https://i.imgur.com/GcI3p8X.png)
    ![](https://i.imgur.com/I9D4dOA.png)

    上圖則是圖刪除右半部的樹叢，可以看到天空有些不自然(染上建物的白色)，下方則填補仿造左側樹叢遮住的延伸建物。
    這個例子則顯示某些程度下Dissect能夠考慮進對稱的因素，但也因此在上半部條件不足時易發生不正常之取樣。

---

### 2. Dissect any GAN model and analyze what you find

生成的HTML
Layer 1
![](https://imgur.com/cFehwap.png)
Layer 4
![](https://imgur.com/P5bw27H.png)
Layer 7
![](https://imgur.com/ECHxNRT.png)

| |Layer 1|Layer 4|Layer 7|
|---|---|---|---|
| Image | ![](https://imgur.com/SKSTI5t.png) | ![](https://imgur.com/SKSTI5t.png) | ![](https://imgur.com/dWo002R.png) |
| Segmentation | ![](https://imgur.com/VNz5Awo.png) | ![](https://imgur.com/tXzV6tF.png) | ![](https://imgur.com/iKQtAI0.png) |
|Mask | ![](https://imgur.com/5cDyuQh.png) | ![](https://imgur.com/CybcyEr.png) | ![](https://imgur.com/hB47rfE.png) |


GAN Dissection可以讓GAN的分析視覺化，讓我們可以了解到每一個Unit生成了哪些圖片或是它的意義。GanDissect就是以生成圖片為例，利用Segmentation與Mask進行分析，判斷哪一個Unit與哪一個Class相關，進而可以修改特定的Unit。

* 上圖為GanDissect跑living room GAN model的結果
    可以看出Layer 1的結果不盡理想，時常會有誤判，也無法正確框出物件的Mask。而在Layer 4則有明顯的進步，可以正確圈出Mask  的位置，但是在Mask大小方面還是無法準確的預測，常常會太小或是超過物件的大小。最後Layer 7則可以更有效的標示出Mask的範圍，雖然比Layer 4的結果好一些，但有時還是會超出一些些範圍，同時還會誤判一些小範圍。
    
---

### 3. Compare with other method - Generative Inpainting
Reference - Generative Image Inpainting with Contextual Attention [[Paper]](https://arxiv.org/pdf/1801.07892.pdf) [[Code]](https://github.com/JiahuiYu/generative_inpainting)

* Analysis of its structure:
    
    本篇Paper的貢獻在綜合傳統方法 & CNN，提供一種可修補複數缺失區域，且快速產生結果的Inpainting method。


    * 傳統方法的優缺：在處理大範圍遺失背景時相當出色，然而對於複雜、屬於非重複性的結構物件難以預測，特別是high-level object的語意分析上。 
    * CNN的優缺：在預測非重複、高度結構性的物件(Ex. 臉、風景)上擁有傑出的效果，但由於缺乏對距離缺陷範圍較遠區塊的關聯預測，導致補齊區塊與周遭之邊界容易出現瑕疵，雖然可使用post-processing適度優化，卻也造成修復時間與GPU耗費大幅增加。
    * 採用以上兩種方法，該神經網路架構的組成上分為兩部分討論：
        1. Generative Inpainting network
            ![](https://i.imgur.com/6L4C30z.png)
            
            Inpainting的生成上分成兩階段，第一階段是Coarse network，先預測出一個模糊、大致的修復圖，然後再經由二階段的Find network進行Refinement，目的是盡可能的擴大CNN的感知區域，如此Refinement network在重建時，感知的內容會比有缺陷的Input完整，其結果及細節也會更加完美，另外值得一提的是Spacial discounted reconstruction loss於Coarse stage，由於一個良好的重建圖，並不一定會跟原圖長得一模一樣(只要滿足真實感就好)，單純考慮和原圖的相似性是不足的，此外隨著missing hole朝中心移動，與邊界的模糊程度也愈高，因此缺失部分隨著離邊界愈遠而降低loss的計算制度能幫助修復結果在視覺上更加完美。
        2. Contextual Attention
            ![](https://i.imgur.com/McwUv16.png)

            此處導入了所謂的Attention model，簡單來說就是預測該遺失pixel應該由哪個方向的其他部分來補足，例如從視覺化的角度來看，白色的代表自己(missing hole的中心區域屬於這塊，代表跟周圍較無關聯)，而粉色則指左下較為接近原圖，透過一小塊一小塊patch計算並傳遞attention score，便能提供Inpainting part補齊的最好參考目標。

* Comparison - 以 Inpainting - 移除畫面物件作比較

    這裡我們取兩張來自GAN Dissect Web Demo的圖片，第一張建築物佔比較小、以天空草地為主體，第二張則建築物占了畫面大半，以此分別再做兩種物件刪減─刪除小範圍物件 & 刪除大範圍物件，去觀察其結果。
PS: 下面表格中的Mask為Generative inpainting之Input，Dissect部分則採手動於Web Demo選擇物件和描繪範圍

#### Photo 1:
![](https://i.imgur.com/C9eKJsj.png)

|Method<br>\\<br>Mask | ![](https://i.imgur.com/1TtpwEq.png) | ![](https://i.imgur.com/XGlOvQ1.png) | ![](https://i.imgur.com/CrTG3xg.png) |
|---|---|---|---|
| GAN<br>Dissect  | ![](https://i.imgur.com/Pyt8b3s.png) | ![](https://i.imgur.com/o3Q5KLA.png) | ![](https://i.imgur.com/ZVzrlZ6.png) |
| Generative<br>inpainting  | ![](https://i.imgur.com/DBVvt8j.png) | ![](https://i.imgur.com/qZrLZjT.png) | ![](https://i.imgur.com/DNlU4zW.png) |

Photo 1的大部分物件是背景，從第一種嘗試─刪除小物件的結果看見，天空的部分兩者表現都不錯，窗戶的部分受限Door obejct沒辦法接納太廣義的Door，Dissect僅僅模糊化該區域，而Inpainting則完美的刪去，至於左邊的樹叢就凸顯出兩者方法的差異，Dissect連同下方未圈選範圍一律替換掉。
<br>第二種嘗試─刪除大物件，則更加的明顯，天空部分Inpainting出現明顯的瑕疵(誤補成建築物)，Dissect則能漂亮的轉換成較厚的雲層，然而改成刪除左方建築時，即使分開選擇Dome+Brick也幾乎沒有改變，Inpainting則是成功的以天空取代原有建物。
<br>從這裡可以發現方法間兩種不同的特性： 首先和仰賴Mask去移除特定範圍的Inpainting不同，Dissect刪除是以"整個Object"為單位執行，因此即便我們只圈選了目標的一小部分，仍會先預測物件的整體分布、再進行模糊刪除(也有可能考量到使用者不會圈選的太精準)。
<br>再來就是對於"刪除"本質上的不同，Inpainting重點在於修補原圖、盡可能在看不到此區域下重現一張真實的圖，在附近有可借鏡之情況下可有不錯的效果，但一旦遮住的區域影響到畫面物件佔比就有很高機率出錯(如刪除天空那次)；Dissect則是在感知物件後，再根據原圖畫面"替換"回合適內容，我們嘗試後發現只有在大範圍的重複區塊，如樹、天空等，擁有很完美的效果，然而小物件如窗戶&門，則不一定每次奏效，甚至當Dissect判斷該物件不符所選 or 移除將無法找到合適內容替換以維持真實感時，操作便不會發生。

#### Photo 2:
![](https://i.imgur.com/xYPncwO.png)

|Method<br>\\<br>Mask | ![](https://i.imgur.com/KRTrnpI.png) | ![](https://i.imgur.com/D38CUvJ.png) | ![](https://i.imgur.com/DhdgEHq.png) |
|---|---|---|---|
| GAN<br>Dissect | ![](https://i.imgur.com/wI8C9CF.png) | ![](https://i.imgur.com/KL5Qwoj.png) | ![](https://i.imgur.com/lemlWEk.png) |
| Generative<br>inpainting | ![](https://i.imgur.com/yOUlX2Q.png) | ![](https://i.imgur.com/HLFWOSd.png) | ![](https://i.imgur.com/JAB9ZqR.png) |

Photo 2的結果與Photo 1雷同，在第一種嘗試下同樣對，對於Dome & Door的移除，Dissect幾乎只是稍微扁平該區域，Inpainting則是成功抹去，樹的抹去也如上述推測，Inpainting是刪去該區域樹枝，Dissect則是連同未圈選範圍一同替換成顏色較深的樹，第二種嘗試也一樣，刪除Dome在Dissect中僅僅做了扁平化，而另一方面刪除整塊樹葉則讓Inpainting產生和Photo 1刪除天空一樣的問題。

<br>

* Conclusion:

    由前面比較可知，Generative inpainting在修復圖片時，第一動作是視作該範圍不存在，因此該區域一定會被刪除，再次藉由剩下區塊修復，原有物件一定會消失，但若影響範圍太大結果通常不甚滿意。相較於前者，Dissect則是以物件為單位做置換，並以整體真實性為重，所以能發現有時候在遇到連續型物件(Ex. 樹)時，結果會與我們預期的有些誤差(從刪除部分物件變成整個置換成不一樣的東西)，但至少在維持畫面真實性上，會比起Inpainting來得周全(但也很有可能反倒什麼也沒做)。

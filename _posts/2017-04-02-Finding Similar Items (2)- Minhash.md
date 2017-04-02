---
layout: post
title: Finding Similar Items (2)- Minhash
modified: 2017-04-02
categories: [data analytic]
tags: [minhash,Jaccard similarity]
comments: true
---

這篇文主要會簡介 Jaccard similarity 及 Minhash 的原理、證明及實作的技巧。  

為了降低資料的維度，且仍能保留相似度(similarity)，我們希望能找到一個hash函數h符合以下兩種特徵:

 1. 能將高維度的資料C，映射為較低維度的特徵h(C) (signature)
 2. 維持 $$Sim(C_1,C_2) ≒ Sim(h(C_1),h(C_2))$$
 
>  $$C_1 = 10110101$$   
>  $$C_2 = 10000001$$  
>  $$h(C_1) = 1011$$  
>  $$h(C_2) = 1101$$  
>  $$Sim(C_1,C_2) = \frac{4}{8}$$    
>  $$Sim(h(C_1),h(C_2)) = \frac{2}{4}$$    
  

####  **Jaccard similarity**
在這邊首先要介紹一種相似度的一種度量方法 Jaccard similarity。這是一個常用於計算集合相似度的方法:  
> $$Jac(C_1,C_2) = \frac{|C_1∩C_2|}{|C_1∪C_2|}$$   
> $$C_1 = \{a,b,c\}$$  
> $$C_2 = \{a,d\}$$  
> $$Jac(C_1,C_2) = \frac{1}{4}$$  
  

#### **Minhash 原理**
若以 Jaccard similarity 來計算相似度，Minhash 符合了上述的兩個條件，可以幫助我們降維且保留相似度。Minhash 的演算法如下:

 1. 首先找出不同文件的 similarity matrix:

| element | $$C_1$$ | $$C_2$$ | 類別 |
|:---:|:---:|:---:|:---:|
| a | 1 | 1 | X |
| b | 1 | 0 | Y |
| c | 1 | 0 | Y |
| d | 0 | 1 | Y |
| e | 0 | 0 | Z |

row 代表的是 element，column 代表的是集合(文件)。  
若值為1代表集合內包含該元素，0則是不包含。$$C_1 = {a,b,c} , C_2 = {a,d}$$

 2. 隨機調換順序

| element | $$C_1$$ | $$C_2$$ | 類別 |
|:---:|:---:|:---:|:---:|
| e | 0 | 0 | Z |
| b | 1 | 0 | Y |
| a | 1 | 1 | X |
| c | 1 | 0 | Y |
| d | 0 | 1 | Y |

 3. 以每個集合裡第一個 element 為特徵(signature)。
 $$h(C_1)=b , h(C_2)=a$$
 
 4. 重複2、3步驟K次，則得到特徵為 K-bits 的向量。  
 $$h(C_1)=badd....ab , h(C_2)=abdc....db$$  
 若元素有M個(M-dimension)，選取K<M，即可達到降維。

#### **Minhash 證明**
Minhash 的演算法看起來非常簡單，但卻能維持相似度  
並有$$P(Minhash(C_­1) = Minhash(C_2)) = Jac(C_1,C_2)$$的結論，  
以下簡單的證明一下這個結論。

首先我們對 row 的組合分成3種類別:
1. X: 均為1，在兩個集合中皆包含此元素，如a。
2. Y: 其中一個為1，只有一個集合包含此元素，如b。
3. Z: 均為0，兩個集合都不包含此元素，如e。

由於 Minhash 是採每個集合裡第一個元素作為值，所以 Minhash 的結果不會有Z類，我們可以將Z類排除。
因此$$Minhash(C_­1) = Minhash(C_2)$$ 的情形就只會發生在:當排序為X類較所有Y類前面時。
若今天有Y類排在X類前面，如上述調換順序後的情形，則 Minhash 值勢必不會相等。
由於採取隨機排序，所以X類排在Y類前面的機率則為$$\frac{|X|}{|X|+|Y|}$$ (第一個抽籤抽到X的機率)，當選取次數K夠大時，會相當接近理想機率值。我們得到結論:
$$P(Minhash(C_­1) = Minhash(C_2)) = \frac{|X|}{|X|+|Y|}$$

再來看$$Jac(C_1,C_2)$$，根據 Jaccard similarity 的定義，得到:
$$Jac(C_1,C_2) = \frac{|C_1∩C_2|}{|C_1∪C_2|} = \frac{|X|}{|X|+|Y|}$$

非常神奇的，我們得到了$$P(Minhash(C_­1) = Minhash(C_2)) = Jac(C_1,C_2)$$的結論。所以 Minhash 在保留相似度的前提下，達到了降維的效果。

#### **Minhash 實作技巧**
 - 隨機排序: 由於隨機排序本身十分花費時間，所以我們採取 row hashing，隨機選取K個 hash function，對每個 row 進行 hash，並確保不會發生 collision，則可達到隨機排序的效果。
	 - 若元素有M個，隨機選取a、b<M，c為大於X的質數，可產生隨機的 hash function: $$h(x) = (ax+b)%c $$
 - 第一個元素: 經過 row hashing 後的值越小，排在越前面。所以要找出第一個元素，只需將每個值為1的 row 丟入 hash function 中，紀錄最小的 hash 值，即為第一個元素。

 








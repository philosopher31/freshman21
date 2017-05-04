---
layout: post
title: Frequency itemset mining (2)- Association rule
modified: 2017-05-04
categories: [data analytic]
tags: [association rule]
excerpt: 我們希望尋找在眾多商品之間的關係，這種尋找多對多且同時發生的關係，我們稱之為 Market Basket Analysis。如同IF {A} then {B} ,當A發生時B會同時發生(co-occurrence)的機率很高時,我們會說A與B之間存在著關聯。本系列文會介紹找出這種關聯的技巧associationrule，以及兩種演算法來解決因資料量太大而需要大量disk IO的問題。
comments: true
---
Association rule learning 就是在資料中找尋關係，如買尿布的人很有可能會買奶粉，我們稱尿布與奶粉有"關係"。其定義如下:   
> $$X \Rightarrow Y \; where \; X,Y \subseteq I$$  
> $$I=\{i_1,i_2,...,i_n\} \ i: item. $$  

若所有在 X 中的 item 皆出現在 basket 中，則Y很有可能也會出現在 basket 中。

那如何量化並描述這個"可能"呢? 首先要先介紹一些名詞跟觀念:
- database T:  

	| basket ID | A | B | C | D | E |  
	|:---:|:---:|:---:|:---:|:---:|:---:|  
	| 1 | 1 | 1 | 0 | 0 | 0 |  
	| 2 | 0 | 0 | 1 | 0 | 1 |  
	| 3 | 0 | 1 | 0 | 1 | 1 |  
	| 4 | 1 | 1 | 1 | 0 | 0 |  
	| 5 | 0 | 1 | 1 | 1 | 1 |  

	1代表有買，0代表無  

- Support: $$supp(X) = \frac{ \mid t \in T \, ; \, X \subseteq t \mid }{\mid T \mid}$$   
意思是 itemset X 有多常出現在database中，也就是X出現的機率 $$P(E_X)$$。   
$$EX: supp({D,E}) = 2/5$$。  

- Confidence: $$ conf(X \Rightarrow Y) = \frac{supp(X \cup Y)}{supp(X)} $$   
意思是這條關聯有多常是對的，又可以說是有多少機率這條關聯成立。其實基本上這就是條件機率，在 X 出現的機率下，X,Y 同時出現的機率。  
這邊有個容易令人搞混的地方，既然他是條件機率，為什麼是 X ∪ Y 呢? 因為 X,Y 皆為集合而不是機率，故  
$$supp(X \cup Y) = P(E_X \cap E_Y)$$  
$$conf(X \Rightarrow Y) = \frac{P(E_X \cap E_Y)}{P(E_X)}$$  
EX: conf({A,B} => {C}) = 0.2/0.4 = 1/2 

- Interest: $$Interest(X \Rightarrow Y) = \mid conf(X \Rightarrow Y) - supp(Y) \mid$$  
並不是每個有高 confidence 的關聯都是我們想要找的，若 Y 出現在每個 bucket，找到 X 與 Y 關聯則沒有什麼意義。因此我們將 confidence 減掉 Y 出現的機率，這個值夠大(通常大於0.5)，我們才說這是一條有意義的關聯。

- Lift: $$ lift(X \Rightarrow Y) = \frac{supp(X \cup Y)}{supp(X) \times supp(Y)}$$  
若 lift = 1 ,則代表 X,Y 兩者獨立，此條關聯也就毫無意義。

有了上述的概念後，我們即可定義想要找的 association rule:
EX: Find all association rules with support > 0.4 and confidence > 0.5

1. 找出符合 support 限制的 itemsets 組合  
{B},{C},{E}

2. 找出符合 confidence 限制的 rule  
conf(B → C) = 0.5    (X)   
conf(C → B) = 0.667  (○)  
conf(C → E) = 0.667  (○)  
conf(E → C) = 0.667  (○)  
conf(B → E) = 0.5  	 (X)  
conf(E → B) = 0.667  (○)  

當資料量很大，在第一步上就會遇到困難，為了要找出在 I 中的所有可能的 itemsets，組合數會高達$$ 2^n -1 $$，很容易產生記憶體不足的問題。因此我們下一節會介紹幾個演算法來嘗試解決這個問題。
 









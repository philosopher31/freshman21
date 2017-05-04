---
layout: post
title: Frequency itemset mining (1)- Overall
modified: 2017-04-18
categories: [data analytic]
tags: [association rule,market basket]
excerpt: 我們希望尋找在眾多商品之間的關係，這種尋找多對多且同時發生的關係，我們稱之為 Market Basket Analysis。如同IF {A} then {B} ,當A發生時B會同時發生(co-occurrence)的機率很高時,我們會說A與B之間存在著關聯。本系列文會介紹找出這種關聯的技巧association rule learning，以及兩種演算法來解決因資料量太大而需要大量disk IO的問題。
comments: true
---
當顧客在購買一項產品時同時會購買什麼其他商品呢?  
我們希望尋找在眾多商品之間的關係，這種尋找多對多且同時發生的關係，  
稱之為 Market Basket Analysis。  

而在 Market Basket Analysis 中，我們需要界定一些名詞來描述這個 Model。
- items: 所有我們希望尋找關聯的物體，如超市中的所有商品。
- basket: 部分 items 的集合(itemset)，如超市的購買紀錄。
- association rule : itemset間的關聯，如買A的人通常會買B。

Market Basket Model 通常包含了以下的特徵: 
- 大量的 items
- 大量的 baskets
- 每個 basket 包含的 item 數不多

Market Basket Analysis 有著不少的應用，其中最著名的是: 透過一起購買的商品紀錄 (Basket)找出商品(items)間的關聯性，如美國超市 Walmart 發現最常與尿布一起購買的竟然是啤酒! 而其他還有像透過病人的服藥狀況(Basket)中找出藥與副作用(items)的關聯等。

我們後續會介紹如何尋找關聯的規則(association rule) 。我們將透過 basket 內商品的不同組合，來判斷這些商品組合是否有關聯。  

由於 basket 的數量通常非常多(EX:一個月內超市內的購買紀錄筆數)以及商品的組合數很多$${N}\choose{k}$$，基本上是無法全部讀入記憶體內做運算的，但若每次都需要
disk IO，則會耗費許多時間,因此我們會介紹 A-Priori 及 PCY 兩種演算法來嘗試對記憶體做最好的利用，進而解決這個問題。

<br/>







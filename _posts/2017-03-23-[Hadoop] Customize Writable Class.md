---
layout: post
title: Hadoop - Customize Writable Class
modified: 2017-03-23
categories: [Hadoop]
tags: [Hadoop]
comments: true
---


在 Hadoop 中，很多時候是官方提供的 Writable Class (ex: IntWritable, Text)是不夠用的，所以需要自己寫個 Class 來實作 Writable。

Writable Class 最主要的用途在於它是一個可序列化的物件 (serializable object)，由於在 Hadoop 不同階段 (Mapper、Combiner、Reducer 等)間的資料傳輸，都會把資料轉成 byte code(serialize)寫至 local dis k中，下個階段再從 disk 中將資料轉回來(deserialize)。所以 Writable 中就是由 Write 實作serialize，readFields 實現 deserialize。
以下實作了幾種常用的資料型態 :
```java
public class MyWritable implements Writable {
    private int a;
    private long b;
    private double c;
    private Text d;
    private ArrayList<Integer> e;

    public MyWritable(){
	d = new Text();
	e = new ArrayList<Integer>();
    }
    public MyWritable(int a, long b,double c,Text d, ArrayList<Integer> e){
	this.a = a;
	this.b = b;
	this.c = c;
	this.d = d;
	this.e = e;
    }
    public void write(DataOutput out) throws IOException {
	out.writeInt(a);
        out.writeLong(b);
        out.writeDouble(c);
        d.write(out);
        out.writeInt(e.size()); // write ArrayList size
	for(int data: e) {
	    out.writeInt(data);
	}		 
    }
    public void readFields(DataInput in) throws IOException {
        a = in.readInt();
        b = in.readLong();
        c = in.readDouble();
        d.readFields(in);
        int size = in.readInt(); // read ArrayList size
	e.clear();
	for(int i=0;i<size;i++) {
	    e.add(in.readInt());
	}	 
    }
}
```
小提醒: 在使用 Writalbe時，由於 Hadoop 會將資料寫至硬碟，使用較大的 Class 會造成讀寫速度變慢，所以盡量選用適合大小的 Class (IntWritable、LongWritable)。

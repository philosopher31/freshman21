---
layout: post
title: Hadoop - Wordcount example code explanation
modified: 2017-03-24
categories: [Hadoop]
tags: [Hadoop]
comments: true
---


Wordcount 就是 Hadoop 中的 Hello World，以下是官方所提供的完整程式碼。

```java
import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {

    public static class TokenizerMapper
        extends Mapper<Object, Text, Text, IntWritable>{
        
        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();
    
        public void map(Object key, Text value, Context context)
            throws IOException, InterruptedException {
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
            word.set(itr.nextToken());
            context.write(word, one);
            }
        }
    }

    public static class IntSumReducer
        extends Reducer<Text,IntWritable,Text,IntWritable> {
        
        private IntWritable result = new IntWritable();
        public void reduce(Text key, Iterable<IntWritable> values,
            Context context) throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
                sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```
首先我們將這個範例程式碼拆成三個部分解析，最一開始就從 main function 開始吧!
### Main function

Configuration: 用來讀取 Hadoop resourc e的 Class，預設會讀入基本的 Hadoop 設定。  
Job: 一個 job 會包含一個完整的 Map-Reduce，需要設定 Mapper、Reducer 等 Class。  
其中比較需要注意的是 setMapOutputKeyClass 及 setMapOutputValueClass，預設的 Mapper ouput key、value 與最後 Reducer output 的 Class 是一樣的，若沒有在 job 中設定就改動的話會出現 error。  

### Mapper function

在自訂 Mapper Class 時需要繼承 Mapper
```java
Mapper<Object, Text, Text, IntWritable>
Mapper<KEYIN,VALUEIN,KEYOUT,VALUEOUT>
```
若沒有在 job 中設定InputFormat，預設的 Class 為 TextInputFormat<LongWritable, Text>，這個 Class 規範了如何讀取 input file，預設即為將輸入檔案一次讀取一行，key 是在檔案中的位置，value 則是文字內容。

```java
StringTokenizer itr = new StringTokenizer(value.toString());
while (itr.hasMoreTokens()) {
    word.set(itr.nextToken());
    context.write(word, one);
}
```
用 StringTokenizer 將一行文字分割(預設用空白分割)，將 key、value 包入 Writable 中傳遞到下個階段。  
EX:
input: 
> I have a pen  
> I have an apple  

output (key,value):
> I  1  
> have  1  
> a  1  
> pen  1  
> I  1  
> have  1  
> an  1  
> apple  1  

### Reducer function

在自訂 Reducer Class 時需要繼承 Reducer
```java
Reducer<Text,IntWritable,Text,IntWritable>
Reducer<KEYIN,VALUEIN,KEYOUT,VALUEOUT>
```
由於 Reduecer 的 input 是承接 Mapper 的 output，所以 KEYIN,VALUEIN 需要與 Mapper 的 KEYOUT,VALUEOUT 相同。  
```java
public void reduce(Text key, Iterable<IntWritable> values,
    Context context) throws IOException, InterruptedException {
    int sum = 0;
    for (IntWritable val : values) {
        sum += val.get();
    }
    result.set(sum);
    context.write(key, result);
}
```
reduce function 中，key 值相同的 value 會被收集起來成為 Iterable<IntWritable> values，對相同 key 的值作加總並輸出結果。  
EX:
input:
> I  [1,1]  
> have  [1,1]  
> a  [1]  
> pen  [1]  
> an  [1]  
> apple  [1]  

output:
> I  2  
> have  2  
> a  1  
> pen  1  
> an  1  
> apple  1  




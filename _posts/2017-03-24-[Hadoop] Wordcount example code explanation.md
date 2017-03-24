---
layout: post
title: Hadoop - Wordcount example code explanation
modified: 2017-03-24
categories: [Hadoop]
tags: [Hadoop]
comments: true
---


Wordcount 就是Hadoop中的Hello World，以下是官方所提供的完整程式碼。

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
首先我們將這個範例程式碼拆成三個部分解析，最一開始就從main function開始吧!
### Main function

Configuration: 用來讀取Hadoop resource的Class，預設會讀入基本的Hadoop設定。  
Job: 一個job會包含一個完整的Map-Reduce，需要設定Mapper、Reducer等Class。  
其中比較需要注意的是 setMapOutputKeyClass 及 setMapOutputValueClass，預設的Mapper ouput key、value與最後Reducer output的Class是一樣的，若沒有在job中設定就改動的話會出現error。  

### Mapper function

在自訂Mapper Class時需要繼承Mapper
```java
Mapper<Object, Text, Text, IntWritable>
Mapper<KEYIN,VALUEIN,KEYOUT,VALUEOUT>
```
若沒有在job中設定InputFormat，預設的Class為TextInputFormat<LongWritable, Text>，這個Class規範了如何讀取input file，預設即為將輸入檔案一次讀取一行，key是在檔案中的位置，value則是文字內容。

```java
StringTokenizer itr = new StringTokenizer(value.toString());
while (itr.hasMoreTokens()) {
    word.set(itr.nextToken());
    context.write(word, one);
}
```
用StringTokenizer將一行文字分割(預設用空白分割)，將key、value包入Writable中傳遞到下個階段。  
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

在自訂Reducer Class時需要繼承Reducer
```java
Reducer<Text,IntWritable,Text,IntWritable>
Reducer<KEYIN,VALUEIN,KEYOUT,VALUEOUT>
```
由於Reduecer的input是承接Mapper的output，所以KEYIN,VALUEIN 需要與 Mapper的 KEYOUT,VALUEOUT相同。  
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
reduce function中，key值相同的value會被收集起來成為Iterable<IntWritable> values，對相同key的值作加總並輸出結果。  
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




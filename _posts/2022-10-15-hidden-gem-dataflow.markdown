---
layout: post
title:  "DataFlow, a hidden gem of TPL"
date:   2022-10-11 22:35:35 +0300
categories: Task parallel library, dotnet
---
Do you need to execute tasks in a chain like a pipeline and configure these to work multithread without having a headache? In that case, DataFlow may help you.

Last week I was looking for a library for my ETL(Extract Transform Load) tasks. The task is based on transferring data from different kinds of sources to another. So, data sources and targets were completely different kinds of animals. While looking for mature ETL libraries I also read DataFlow documents because most dotnet libraries rely on that. I already used DataFlow for other cases in the past few years of course because it is efficient to have control over parallel tasks.

I have implemented a benchmark tool named [WebBen](https://github.com/omerfarukz/WebBen)chmark. An action block to fetch/invoke an http(s) page store statistics over throughput. I have to send a concurrent request as I can. But I never need to have a chain of them. I confess I am impressed. DataFlow may your favourite for your next project.

It is very easy to have a flow using the internal API of dotnet. No need to use another library for solving dozens of tasks on your own. If you have a library not based on DataFlow, do not worry about it. Using an existing code base is probably easier than you think.

In basic, there are three kinds of interface. These are called a block because they are designed to link one or more to the next one or another. We can configure all blocks for their parallelism and bounded capacities individually.

- ISourceBlock<TOutput>
- ITargetBlock<TInput>
- ITransformBlock<TInput, TOutput>

## ISourceBlock<TOutput>
The source block is designed to produce TOutput typed data.

## ITargetBlock<TInput>
Target block is like a method that has TInput typed method.

## ITransformBlock<TInput, TOutput>
Transform block consumes TInput and produces TOutput.

In this article, I planned to demonstrate an implementation of a simple hash calculator flow.

The following code sample downloads bytes of data for a given Uri. It will work in 50(max) parallel threads.

```csharp
var sharedHttpClient = new HttpClient();
var downloadBytesBlock = new TransformBlock<Uri, byte[]>(async f =>
{
    var response = await sharedHttpClient.GetAsync(f);
    return await response.Content.ReadAsByteArrayAsync();
}, new ExecutionDataflowBlockOptions() {MaxDegreeOfParallelism = 50});
```

Next block to calculate hash of content bytes. Only had 100 inputs at a time and processes in 30(max) parallel threads.

```csharp
var calculateHashBlock = new TransformBlock<byte[], string>(input =>
{
    var hashBytes = MD5.Create().ComputeHash(input);
    return BitConverter.ToString(hashBytes).Replace("-", "");
}, new ExecutionDataflowBlockOptions() {BoundedCapacity = 100, MaxDegreeOfParallelism = 30});
```

Print hashes to console out.
```csharp
var printContentBlock = new ActionBlock<string>(
    (input) => Console.WriteLine($"Content Hash: {input}"),
    new ExecutionDataflowBlockOptions() { BoundedCapacity = 1000, MaxDegreeOfParallelism = 1}
);
```

Lets pass Uri's to downloadBytesBlock

```csharp
// mark download block as completed. so executing will start
downloadBytesBlock.Complete();
// wait for completion of print content block
await printContentBlock.Completion;
```

All blocks are implemented to work with different parallelism and bounded capacity configurations. Bounded capacity is the maximum number of messages that may be buffered by the block. There are other built-in blocks like TansformManyBlock. I might be to describe how does dataflow actually works under the hood on later posts.
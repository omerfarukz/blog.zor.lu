---
layout: post
title:  "DataFlow, a hidden gem of TPL"
date:   2022-10-11 22:35:35 +0300
categories: Task parallel library, dotnet
---

## TL;DR;
This is a page to test jekyll's abilities and also helps me to improve my skills. So, it is not ready to read.

## Story
Do you need to solve chain of taks in parallel and configure individually? Data flow may help you.

On last week I was looking for a library for my ETL(Extract Transform Load) tasks. Task is based on transferring data from multiple and different data sources to anothers. Sources and targets are completely different kind of animals. While looking for mature ETL libraries I also read DataFlow documents because most dotnet libraries are relies on that. I already used DataFlow for other cases in past few years of course because it is efficient to have a control over parallel tasks.

I have implemented a bencmark tool named WebBen[chmark]. An action block to fetch/invoke an http(s) page store statistics over througtput. I have to send concurrent request as I can. But I never need to have a chain of them. I confess, I am impressed. DataFlow may your favorite on your next project.


It is very easy to have a flow using internal API of dotnet. Not need to use another library for solving a dozens of task your own. If you have a library not based on DataFlow, do not worry about it. Using existing code base is probably easier than you think.

TODO;
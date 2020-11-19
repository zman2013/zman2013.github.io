---
title: Mockito invoking callback
tags:
  - Mockito
id: '342'
categories:
  - - Java
  - - Unit Test
date: 2020-05-29 21:57:35
---

@Mock
BiConsumer<Integer, Runnable> onNext;

doAnswer((Answer<Void>) invocation ->{
    Runnable readNext = invocation.getArgument(1);
    readNext.run();
    return null;
}).when(onNext).accept(anyInt(), any(Runnable.class));
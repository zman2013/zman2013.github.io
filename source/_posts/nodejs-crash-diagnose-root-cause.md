---
title: Nodejs Crash Diagnose Root Cause
date: 2021-02-18 15:46:45
tags: nodejs domain crash error exception
---

Nodejs 运行期间遇到未捕获的异常会导致进程 crash，此时可以使用 `process.on('uncaughtException')` 捕获异常，但会丢失错误上下文，导致无法定位根本原因。

通过 Nodejs 提供的 `domain` 模块，可以捕获异常同时保留错误上下文。

示例如下:
```typescript
import domain from 'domain'

function main(){
  // biz code
}

domain.create()
  .on('error', console.error)
  .run(main)
```

## 注意
1. If any of the event emitters or callbacks registered to a domain emit an 'error' event, or throw an error, then the domain object will be notified, rather than losing the context of the error in the `process.on('uncaughtException')` handler, or causing the program to exit immediately with an error code.
2. Domain error handlers are not a substitute for closing down a process when an error occurs.
3. By the very nature of how throw works in JavaScript, there is almost never any way to safely "pick up where it left off", without leaking references, or creating some other sort of undefined brittle state.The safest way to respond to a thrown error is to shut down the process. 

## 参考
1. https://nodejs.org/dist/latest-v15.x/docs/api/domain.html



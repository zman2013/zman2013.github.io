---
title: nodejs-smart-libraries
date: 2022-08-29 15:59:19
tags:
---

# stmux
https://www.npmjs.com/package/stmux

Simple Terminal Multiplexing for Node Environments

This utility is primarily intended to be used from within a package.json script to easily side-by-side run various NPM-based commands in a Node.js build-time environment.

# blessed、blessed-contrib
https://www.npmjs.com/package/blessed
https://www.npmjs.com/package/blessed-contrib

A [curses-like](https://en.wikipedia.org/wiki/Curses_(programming_library)) library with a high level terminal interface API for node.js.

# shimmer
https://www.npmjs.com/package/shimmer

shimmer does a bunch of the work necessary to wrap other methods in a wrapper you provide.

# v8-profiler-next
https://www.npmjs.com/package/v8-profiler-next

v8-profiler-next provides node bindings for the v8 profiler.

# heapdump
https://www.npmjs.com/package/heapdump

Make a dump of the V8 heap for later inspection.

# yargs
https://github.com/yargs/yargs

Yargs helps you build interactive command line tools, by parsing arguments and generating an elegant user interface.

```typescript
# tsnd src/index.ts add --title 'title-A' --body 'body-A' 

import yargs, { Arguments, Argv, CommandModule } from 'yargs'
import { hideBin } from "yargs/helpers";


console.log(process.argv)

type PrepareArgs = { title: string, body: string }
class ABC implements CommandModule<{}, PrepareArgs>{
    constructor(
      public command = "add", 
      public describe = "Add a new note"){
    }
    builder(argv: Argv<{}>): Argv<PrepareArgs>{
        return argv
      .option("title", {
        desc: "name prefix",
        type: "string",
        demandOption: true,
      })
      .option("body", {
        desc: "directory containing video files",
        type: "string",
        demandOption: true,
      });
    }
    async handler(args: Arguments<PrepareArgs>){
        console.log(args.title, args.body)
    }
}    

// Create add command 
yargs(hideBin(process.argv))    // 删除 process.argv 包含的头两个字段: /bin/node, filePath
    .scriptName("yargs-test")
    .command(new ABC() as any)
    .command(new XXX() as any)  // XXX is another command
    .parseAsync();
```
---
title: JavaScript - JS Engine and Runtime
tags:
  - JavaScript
categories:
  - JavaScript
date: 2023-08-01 10:42:42
---


JS 引擎中包含 Call Stack 與 Heap，在程式碼進入 Call Stack 之前，會先進行解析與編譯的過程，JavaScript 為即時編譯語言，可以比直譯語言更快的執行程式碼，提升運行速度。

<!-- more -->

# JS 引擎

引擎中包含兩個部分 :

1.  呼叫堆疊(Call Stack) : 程式的執行區域。
2.  堆積(Heap) : 物件儲存的區域。

![JavaScript 引擎](js_engine.webp)

# 直譯 & 編譯 & 即時編譯

將程式語言轉換成機器語言的過程。

直譯(Interpretation) : 程式碼一行一行的執行，同時轉換成機器碼。

![Interpretation](interpretation.webp)

編譯(Compilation) : 程式碼先編譯成機器碼，建立 protable file，之後隨時執行程式。

![Compilation](compilation.webp)

即時編譯(Just-in-time compilation) : 程式碼先編譯成機器碼，之後立刻執行程式。

![JIT compilation](jit.webp)

# 即時編譯過程

![JS engine process](js_engine_process.webp)

JavaScript 的即時編譯過程 :

1.  解析(Parsing) : 將程式碼解析成 AST(abstract syntax tree ，抽象語法樹)，並檢查語法錯誤。
2.  編譯(Compilation) : 將 AST 編譯成機器語言。
    - 為了盡速執行，所以先產生未優化的機器碼。
3.  執行(Execution) : 立即於 call stack 中執行。
4.  優化(Optimization) : 將最初編譯的未優化機器碼進行優化，並重新編譯。

# Runtime

瀏覽器中的 Runtime :

![瀏覽器中的 Runtime](browser_runtime.webp)

Node.js 中的 Runtime :

![Node.js 中的 Runtime](node_runtime.webp)

# 參考資料

- [The Complete JavaScript Course 2023: From Zero to Expert!](https://www.udemy.com/course/the-complete-javascript-course/)

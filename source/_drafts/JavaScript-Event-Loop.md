---
title: JavaScript - Event Loop
tags:
  - JavaScript
categories:
  - JavaScript
---

<!-- more -->

# JavaScript Runtime

JS runtime 中包含 JS engine、Web API 與 Callback queue。

JS engine 為 runtime 的核心，其中 :

- Heap : 物件儲存的地方。
- Call stack : 程式執行的地方。

Web API : DOM、Timers、Fetch API...等異步任務執行的地方。

Callback queue : 異步回調函式等待執行的地方。

# Event Loop

JS engine 為單線程，所以一次只能執行一項任務，為了使某些任務不會阻塞程式的運行，JS 使用異步的方式執行程式。

異步程式會在 Web API 環境中執行，當執行完畢它的回調函式推到 callback queue 中排隊等待，當 Call stack 中沒有程式在執行時，Callback queue 中的函式會被推到 Call stack 中執行。

Promise 的異步回調函式與其他異步回調函式不同，不會被推到 Callback queue 中，而是到 Microtasks queue 中。

> Microtasks queue 與 Callback queue 的差別在於，前者的執行先於後者。

# 參考資料

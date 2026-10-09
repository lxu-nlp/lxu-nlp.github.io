---
layout: post
title: "Notes on Agent Loop"
date: 2026-09-17
categories: [Notes]
tags: [agent,harness]
math: true
---

A source-level walkthrough of nanobot's agent loop — from `AgentLoop.run()`
consuming the first message to a complete Turn lifecycle (scheduling layer
`loop.py` and execution layer `runner.py`):

- <a href="/notes/agent_loop_walkthrough.html" target="_blank" rel="noopener">nanobot Agent Loop 深度剖析 (standalone HTML)</a>

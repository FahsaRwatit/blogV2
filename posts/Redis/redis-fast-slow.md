---
title: Redis 的快与慢详解
date: 2025-11-20
tags: [Redis, 性能]
excerpt: Redis 为何如此快，以及它存在哪些慢操作。
---

# Redis 的快与慢

## 为什么快

1. **内存数据库** — 所有操作在内存中完成
2. **高效数据结构** — String、List、Hash、Set、ZSet
3. **单线程模型** — 避免了锁竞争

## 慢操作有哪些

- `KEYS *` — 全量扫描，生产禁用
- `SORT` — 时间复杂度 O(N+M*log(M))
- 大 Value 的读写

```bash
# 用 SCAN 替代 KEYS
SCAN 0 MATCH user:* COUNT 100
```

> 用 `SLOWLOG GET` 排查慢查询。

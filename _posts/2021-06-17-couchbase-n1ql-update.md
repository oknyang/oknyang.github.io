---
title: couchbase array field update
date: 2021-03-16
---


UPDATE orders ord USE KEYS "doc_id"  
SET od.cs = 11111 FOR od IN detailList WHEN od.accountCS = 12345 END  
RETURNING ord;

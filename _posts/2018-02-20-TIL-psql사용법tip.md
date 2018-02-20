---
layout: post
title:  "2018-02-19 TIL"
author: "lastgleam"
---

# psql 사용법

- `\d+;` : 모든 Relation(table, sequence) 정보 확인
  - `\dt;` : table 정보만 확인가능(sequence 확인불가) 
- `\d+ [name];` : 특정 Relation의 정보 확인 
- `select version();` : psql version 확인
  - `psql --version` 으로도 확인 가능

 **출처** : https://hacknote.jp/archives/1574/

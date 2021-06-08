---
title: "RDB"
date: 2021-01-30T15:26:01+09:00
draft: false
hide:
- toc
- nextpage
subpage: false
weight: 1
---

1. [SQL](#1-sql)

<!--more-->

## 1. SQL

### 結合

- INNER JOIN
    - 指定カラムの値が一致するレコード同士を結合（一致するものだけであり、一致しないものは選択・表示されない）
    - `SELECT * FROM users INNER JOIN accounts ON users.id = accounts.user_id`
- LEFT OUTER JOIN
    - 指定のカラムの値が一致するレコード同士を結合
    - 左のテーブルに存在し、右のテーブルに存在しない値はNULLでパディングされる
    - `SELECT * FROM users LEFT OUTER JOIN accounts ON users.id = accounts.id`
- RIGHT OUTER JOIN
    - LEFT OUTER JOINの逆
    - 指定のカラムの値が一致するレコード同士を結合
    - 右のテーブルに存在し、左のテーブルに存在しない値はNULLでパディングされる
    - `SELECT * from users RIGHT OUTER JOIN accounts ON users.id = accounts.id`
- CROSS JOIN
    - MySQL では INNER JOIN と同等
- UNION
    - 複数の結果セットをひとつの結果セットに結合
    - カラムの数、順番、データ型が一致していなければならない
    - **重複する行は削除されてユニークな値のみ** が結果として得られる
- UNION ALL
    - UNION と異なり、 **重複する行も結果として得られる**
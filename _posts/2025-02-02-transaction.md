---
title: Database - Transaction
date: 2025-02-02 19:23:00 +0900
categories: [Database, More about Database]
tags: [transaction, database]
render_with_liquid: false
---

이전 포스트인 'ACID'를 작성하기 앞서 사실 트랜잭션을 먼저 정의하는 작업이 필요했다.

## What is Transaction?

트랜잭션(Transaction)은 데이터베이스에서 상태를 변경시키기 위해 수행하는 작업 단위를 의미합니다. 이는 여러 SQL 명령문을 하나의 논리적 작업으로 묶어 처리하며, 작업이 모두 성공하거나 모두 실패하도록 보장합니다. 트랜잭션은 데이터 무결성을 유지하고, 데이터베이스의 안정성을 확보하기 위한 핵심 개념입니다.

트랜잭션은 ACID 속성을 만족해야 한다고 한다.

## 주요 연산

1. COMMIT: 트랜잭션의 모든 작업이 성공적으로 완료되었음을 선언하고, 변경 내용을 데이터베이스에 영구적으로 반영합니다.

2. ROLLBACK: 트랜잭션 중 오류가 발생하거나 실패했을 때, 이미 수행된 작업을 취소하고 데이터베이스를 이전 상태로 복원합니다.

이러한 연산은 실제 MySql를 배우면서 볼 수 있었다.

## Transaction 상태

1. Active: 트랜잭션이 실행 중인 상태

2. Partially Committed: 마지막 명령문까지 실행되었지만 아직 COMMIT되지 않은 상태

3. Failed: 오류로 인해 실행이 중단된 상태

4. Aborted: ROLLBACK이 수행된 상태

5. Committed: 성공적으로 완료되어 COMMIT된 상태

## Isolation level

트랜잭션의 격리 수준은 동시성과 데이터 일관성 사이의 균형을 결정합니다. 주요 격리 수준은 다음과 같습니다:

+ Read Uncommitted: 다른 트랜잭션의 커밋되지 않은 데이터를 읽을 수 있습니다.

+ Read Committed: 커밋된 데이터만 읽을 수 있습니다.

+ Repeatable Read: 트랜잭션 내에서 동일한 데이터를 여러 번 읽을 때 일관된 결과를 보장합니다.

+ Serializable: 가장 높은 격리 수준으로, 트랜잭션을 순차적으로 실행합니다.



오늘은 간단히 'Transaction'에 대해 알아보았다.
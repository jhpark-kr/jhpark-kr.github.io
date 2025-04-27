---
title: Oracle의 ROWID 구성
description: >-
  Get started with Chirpy basics in this comprehensive overview.
  You will learn how to install, configure, and use your first Chirpy-based website, as well as deploy it to a web server.
author: jhpark
date: 20250427 22:10:00 +0900
categories: [Database]
tags: [Database Oracle]
pin: true
media_subpath: '/posts/202504272210'
---
### **📌 ORACLE의 ROWID 구성**
`ROWID`는 Oracle에서 **각 행(Row)** 을 고유하게 식별하는 **물리적 주소**입니다.  
이 값은 테이블 내에서 각 행의 위치를 나타내며, **16진수 문자열 형태**로 표시됩니다.

#### **📌 ROWID의 기본 형식**
```
ROWID = OOOOOOFFFBBBBBBRRR
```
각 부분의 의미는 다음과 같습니다:

| 구성 요소 | 길이(비트) | 설명 |
|-----------|----------|------|
| **Object ID (OOOOOO)** | 32비트 | 테이블 또는 클러스터의 객체 번호 |
| **File Number (FFF)** | 10비트 | 행이 저장된 데이터 파일 번호 |
| **Block Number (BBBBBB)** | 22비트 | 데이터 파일 내 블록 번호 |
| **Row Number (RRR)** | 16비트 | 블록 내 행 번호 |

Oracle에서 `ROWID`는 **Base64** 형태로 인코딩된 18자리 문자열로 표현됩니다.

---

### **📌 ROWID 예제**
#### **ROWID 조회**
```sql
SELECT ROWID, EMPLOYEE_ID, LAST_NAME 
FROM EMPLOYEES
WHERE ROWNUM <= 5;
```

#### **결과 예시**
```
ROWID              EMPLOYEE_ID  LAST_NAME
------------------ ------------ ----------
AAAB12AADAAAAUjAAA        100    King
AAAB12AADAAAAUjAAB        101    Kochhar
AAAB12AADAAAAUjAAC        102    De Haan
AAAB12AADAAAAUjAAD        103    Hunold
AAAB12AADAAAAUjAAE        104    Ernst
```
---

### **📌 ROWID 디코딩 예제**
Oracle에서 `DBMS_ROWID` 패키지를 사용하면 `ROWID`를 분석할 수 있습니다.

```sql
SELECT 
    ROWID, 
    DBMS_ROWID.ROWID_OBJECT(ROWID) AS OBJECT_ID,  -- 테이블의 객체 ID
    DBMS_ROWID.ROWID_RELATIVE_FNO(ROWID) AS FILE_NO, -- 데이터 파일 번호
    DBMS_ROWID.ROWID_BLOCK_NUMBER(ROWID) AS BLOCK_NO, -- 블록 번호
    DBMS_ROWID.ROWID_ROW_NUMBER(ROWID) AS ROW_NO -- 블록 내 행 번호
FROM EMPLOYEES
WHERE ROWNUM = 1;
```

#### **결과 예시**
```
ROWID             OBJECT_ID FILE_NO BLOCK_NO ROW_NO
----------------- --------- ------- -------- ------
AAAB12AADAAAAUjAAA   12345      4      1357    2
```

- **OBJECT_ID = 12345** → EMPLOYEES 테이블의 객체 ID  
- **FILE_NO = 4** → 데이터 파일 번호  
- **BLOCK_NO = 1357** → 해당 데이터가 저장된 블록 번호  
- **ROW_NO = 2** → 블록 내 2번째 행  

---

### **📌 ROWID의 활용**
#### 1️⃣ **행을 고유하게 식별**
- `ROWID`는 각 행의 물리적 위치를 나타내므로, **고유한 키 역할**을 할 수 있음.
- 하지만, `ROWID`는 테이블 리오그(재구성) 또는 인덱스 리빌드 후 변경될 수 있음.

#### 2️⃣ **빠른 데이터 조회 (ROWID 기반 접근)**
- `ROWID`는 인덱스를 거치지 않고 **직접 행을 조회**하기 때문에 성능이 매우 빠름.
- 예제:
  ```sql
  SELECT * FROM EMPLOYEES WHERE ROWID = 'AAAB12AADAAAAUjAAA';
  ```

#### 3️⃣ **중복 제거 및 특정 행 삭제**
- 중복 데이터가 존재할 때 `ROWID`를 활용하면 개별 행을 삭제할 수 있음.
- 예제: 중복된 LAST_NAME을 하나만 남기고 삭제
  ```sql
  DELETE FROM EMPLOYEES
  WHERE ROWID NOT IN (
      SELECT MIN(ROWID)
      FROM EMPLOYEES
      GROUP BY LAST_NAME
  );
  ```

---

### **📌 ROWID vs ROWNUM 차이점**
| 비교 항목 | ROWID | ROWNUM |
|----------|------|------|
| 의미 | 행의 물리적 주소 | 결과 집합에서 행의 논리적 번호 |
| 값의 고유성 | 테이블 내에서 고유 | 실행 계획에 따라 변할 수 있음 |
| 변경 가능성 | 테이블 재구성 시 변경될 수 있음 | 쿼리 실행 방식에 따라 변경됨 |
| 성능 | 인덱스 없이 빠르게 조회 가능 | 쿼리 실행 방식에 따라 성능이 달라짐 |

---

### **📌 정리**
- **ROWID는 행의 물리적 주소**이며, 테이블 내에서 **각 행을 고유하게 식별**하는 값.
- **OBJECT_ID + FILE_NO + BLOCK_NO + ROW_NO** 로 구성됨.
- `DBMS_ROWID` 패키지를 사용하면 `ROWID`를 해석 가능.
- **빠른 조회 및 중복 데이터 정리**에 유용하지만, 테이블 구조 변경 시 값이 변경될 수 있음.

**➡ ROWID를 직접 PK로 사용하지는 않지만, 성능 최적화나 특정 행을 고유 식별할 때 유용하게 활용할 수 있음! 🚀**

# SQL_MASTER 4주차 정규과제

📌SQL MASTER 정규과제는 매주 정해진 분량의 『*데이터 분석을 위한 SQL 레시피*』 를 읽고 학습하는 것입니다. 이번 주는 아래의 **SQL_MASTER_4th_TIL**에 나열된 분량을 읽고 공부하시면 됩니다.

아래 실습을 수행하며 학습 내용을 직접 적용해보세요. 단순히 결과를 재현하는 것이 아니라, SQL을 직접 작성하는 과정에서 개념을 스스로 정리하는 것이 중요합니다.

필요한 경우 교재와 추가 자료를 참고하여 이해를 보완하시기 바랍니다.

## SQL_MASTER_4th_TIL

### 5장 사용자를 파악하기 위한 데이터 추출
#### 1. 사용자 전체의 특징과 경향 찾기


## Study Schedule

| 주차  | 공부 범위     | 완료 여부 |
| ----- | ------------- | --------- |
| 1주차 | p.20~50    | ✅         |
| 2주차 | p.52~136   | ✅         |
| 3주차 | p.138~184  | ✅         |
| 4주차 | p.186~232 | ✅         |
| 5주차 | p.233~321 | 🍽️         |
| 6주차 | p.324~406 | 🍽️         |
| 7주차 | p.408~464 | 🍽️         |

<br>

<!-- 여기까진 그대로 둬 주세요-->


# 실습

## 0. 실습 규칙

1. 샘플 데이터 생성 코드는 **07_SQL_MASTER_Template/src** 경로에 장별로 정리되어 있습니다.
2. 아래 목차에 맞춰 해당 코드를 실행하여 샘플 데이터를 생성한 후, 각 장에서 요구하는 쿼리를 직접 작성해보시기 바랍니다.
3. 작성한 쿼리의 **실행 결과 화면도 함께 제출**해 주세요.
4. 단순히 교재의 예시 코드를 그대로 작성하는 것이 아니라, **제시된 로직을 충분히 이해한 뒤 교재를 보지 않고 스스로 쿼리를 구성**해보는 것을 권장합니다.
5. 교재 예시는 PostgreSQL, Hive, BigQuery 등 다양한 DBMS 기준으로 제시되어 있기 때문에, **MySQL이 아닌 다른 SQL 환경을 사용하여 실습을 진행해도 무방합니다.**
6. 다만, 사용 중인 DBMS에 맞는 문법으로 적절히 변환하여 작성하시기 바랍니다.

## 1. 사용자 전체의 특징과 경향 찾기

### 1-1 사용자의 액션 수 집계하기

<img width="1390" height="801" alt="image" src="https://github.com/user-attachments/assets/94833e06-af52-4de9-adb4-53164ea8e9b4" />

```sql
SELECT
    action,
    action_uu,
    action_count,
    total_uu,
    100.0 * action_uu / total_uu AS usage_rate,
    1.0 * action_count / action_uu AS count_per_user
FROM (
    SELECT
        action,
        COUNT(DISTINCT session) AS action_uu,
        COUNT(*) AS action_count
    FROM ch11_1_action_log
    GROUP BY action
) AS a
JOIN (
    SELECT COUNT(DISTINCT session) AS total_uu
    FROM ch11_1_action_log
) AS b
ON 1=1
ORDER BY action_uu DESC;
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->

### 1-2 연령별 구분 집계하기

<img width="1138" height="754" alt="image" src="https://github.com/user-attachments/assets/108dc2d7-8720-4b14-a10f-af17985de28b" />


```sql
SELECT
    age_category,
    COUNT(*) AS user_count
FROM (
    SELECT
        user_id,
        CASE
            WHEN TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) < 20 THEN '20세 미만'
            WHEN TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) BETWEEN 20 AND 29 THEN '20대'
            WHEN TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) BETWEEN 30 AND 39 THEN '30대'
            WHEN TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) BETWEEN 40 AND 49 THEN '40대'
            WHEN TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) BETWEEN 50 AND 59 THEN '50대'
            ELSE '60세 이상'
        END AS age_category
    FROM ch11_2_mst_users
) AS age_table
GROUP BY age_category
ORDER BY
    CASE age_category
        WHEN '20세 미만' THEN 1
        WHEN '20대' THEN 2
        WHEN '30대' THEN 3
        WHEN '40대' THEN 4
        WHEN '50대' THEN 5
        ELSE 6
    END;
```

<!-- 이 부분을 지우고 실행 결과 화면을 제출해주세요. -->

### 1-3 연령별 구분의 특징 추출하기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
SELECT
    user_group,
    COUNT(*) AS user_count
FROM (
    SELECT
        user_id,
        CASE
            WHEN age BETWEEN 4 AND 12 THEN 'C'
            WHEN age BETWEEN 13 AND 19 THEN 'T'
            WHEN age BETWEEN 20 AND 34 THEN CONCAT(sex, '1')
            WHEN age BETWEEN 35 AND 49 THEN CONCAT(sex, '2')
            WHEN age >= 50 THEN CONCAT(sex, '3')
            ELSE '기타'
        END AS user_group
    FROM (
        SELECT
            user_id,
            sex,
            TIMESTAMPDIFF(YEAR, birth_date, register_date) AS age
        FROM ch11_3_mst_users
    ) AS base
) AS grouped_users
GROUP BY user_group
ORDER BY
    CASE user_group
        WHEN 'C' THEN 1
        WHEN 'T' THEN 2
        WHEN 'M1' THEN 3
        WHEN 'F1' THEN 4
        WHEN 'M2' THEN 5
        WHEN 'F2' THEN 6
        WHEN 'M3' THEN 7
        WHEN 'F3' THEN 8
        ELSE 9
    END;
```

<img width="972" height="582" alt="image" src="https://github.com/user-attachments/assets/a315b4f0-0a81-4442-8a35-a1a6d007906c" />

 
### 1-4 사용자의 방문 빈도 집계하기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
SELECT
    visit_group,
    COUNT(*) AS user_count
FROM (
    SELECT
        user_id,
        CASE
            WHEN session_cnt = 1 THEN '1회'
            WHEN session_cnt = 2 THEN '2회'
            WHEN session_cnt = 3 THEN '3회'
            ELSE '4회 이상'
        END AS visit_group
    FROM (
        SELECT
            user_id,
            COUNT(DISTINCT session) AS session_cnt
        FROM ch11_4_action_log
        GROUP BY user_id
    ) AS visit_summary
) AS freq_table
GROUP BY visit_group
ORDER BY
    CASE visit_group
        WHEN '1회' THEN 1
        WHEN '2회' THEN 2
        WHEN '3회' THEN 3
        ELSE 4
    END;
```

<img width="912" height="645" alt="image" src="https://github.com/user-attachments/assets/2110483a-abe8-420a-b2ca-731146eed1d4" />


### 1-5 벤 다이어그램으로 사용자 액션 집계하기


```sql
SELECT
    viewed,
    favorited,
    purchased,
    COUNT(*) AS user_count
FROM (
    SELECT
        user_id,
        MAX(CASE WHEN action = 'view' THEN 1 ELSE 0 END) AS viewed,
        MAX(CASE WHEN action = 'favorite' THEN 1 ELSE 0 END) AS favorited,
        MAX(CASE WHEN action = 'purchase' THEN 1 ELSE 0 END) AS purchased
    FROM ch11_5_action_log
    GROUP BY user_id
) AS action_flag
GROUP BY viewed, favorited, purchased
ORDER BY viewed DESC, favorited DESC, purchased DESC;
```
<img width="955" height="552" alt="image" src="https://github.com/user-attachments/assets/ef336db9-0a7b-4a55-805c-67ef41f4d3c2" />


### 1-6 Decile 분석을 사용해 사용자를 10단계 그룹으로 나누기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
SELECT
    decile_no,
    SUM(user_amount) AS total_amount,
    ROUND(100.0 * SUM(user_amount) / SUM(SUM(user_amount)) OVER (), 2) AS amount_rate,
    ROUND(
        100.0 * SUM(SUM(user_amount)) OVER (
            ORDER BY decile_no
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) / SUM(SUM(user_amount)) OVER (),
        2
    ) AS cumulative_rate,
    ROUND(AVG(user_amount), 2) AS avg_amount_per_user
FROM (
    SELECT
        user_id,
        user_amount,
        NTILE(10) OVER (ORDER BY user_amount DESC) AS decile_no
    FROM (
        SELECT
            user_id,
            SUM(amount) AS user_amount
        FROM ch11_6_action_log
        WHERE action = 'purchase'
        GROUP BY user_id
    ) AS purchase_summary
) AS decile_base
GROUP BY decile_no
ORDER BY decile_no;
```

<img width="1006" height="581" alt="image" src="https://github.com/user-attachments/assets/d0073ca9-8c60-4387-b760-99a650be1454" />


### 1-7 RFM 분석으로 사용자를 3가지 관점의 그룹으로 나누기

<!-- 이 부분을 지우고 새롭게 배운 내용을 자유롭게 정리해주세요. -->

```sql
SELECT
    CONCAT('R', r_rank, 'F', f_rank, 'M', m_rank) AS rfm_segment,
    COUNT(*) AS user_count
FROM (
    SELECT
        user_id,
        CASE
            WHEN recency_days <= 1 THEN 5
            WHEN recency_days <= 3 THEN 4
            WHEN recency_days <= 7 THEN 3
            WHEN recency_days <= 14 THEN 2
            ELSE 1
        END AS r_rank,
        CASE
            WHEN freq >= 3 THEN 5
            WHEN freq = 2 THEN 4
            WHEN freq = 1 THEN 3
            ELSE 1
        END AS f_rank,
        CASE
            WHEN monetary >= 3000 THEN 5
            WHEN monetary >= 2000 THEN 4
            WHEN monetary >= 1000 THEN 3
            ELSE 1
        END AS m_rank
    FROM (
        SELECT
            user_id,
            DATEDIFF('2016-11-05', DATE(MAX(stamp))) AS recency_days,
            COUNT(*) AS freq,
            SUM(amount) AS monetary
        FROM ch11_7_action_log
        WHERE action = 'purchase'
        GROUP BY user_id
    ) AS rfm_base
) AS rfm_score
GROUP BY CONCAT('R', r_rank, 'F', f_rank, 'M', m_rank)
ORDER BY user_count DESC, rfm_segment;
```

<img width="786" height="404" alt="image" src="https://github.com/user-attachments/assets/bde576d6-a3a8-45a3-8380-ab714ab56a0b" />



### 🎉 수고하셨습니다.

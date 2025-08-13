# 牛客SQL快速入门

## SQL22

```mysql
SELECT
    up.university,
    # 某学校用户平均答题数量计算方式为 -- 该学校用户答题总次数除以答过题的不同用户个数
    ROUND(COUNT(qpd.question_id) / COUNT(DISTINCT qpd.device_id), 4) AS avg_answer_cnt
FROM
    user_profile up JOIN question_practice_detail qpd
ON
	up.device_id = qpd.device_id
GROUP BY
    up.university
ORDER BY
    up.university;
```

> **distinct 不能放在 `/`、`+` 等表达式内部**, 示例如下:
>
> `SELECT COUNT(question_id) / DISTINCT(device_id)` -- 错误语法

## SQL29

提要: **`DISTINCT` 会去除结果集中<font style="color:red">所有选中列组合值完全相同的行</font>**, 只保留唯一的一条

- **`DISTINCT` 并不只作用于它后面紧跟的第一个字段, <font style="color:red">而是会作用于所有 SELECT 出来的字段组合</font>**
  - `SELECT DISTINCT device_id, date FROM table;` -- 这并不是只对于 `device_id` 字段进行的去重, **而是对 `device_id + date` 两个列的组合去重**

`DATEDIFF(date1, date2)`: 表示 `date1 - date2` (**单位是天**), 返回值就是 date1 比 date2 晚多少天

```mysql
# 加 DISTINCT 是为了避免同一用户同一天多次刷题被重复计算
-- 解法一
SELECT COUNT(DISTINCT q2.device_id, q2.date) / count(DISTINCT q1.device_id, q1.date) as avg_ret
from
	question_practice_detail as q1
	left outer join
	question_practice_detail as q2
on
	q1.device_id = q2.device_id and DATEDIFF(q2.date, q1.date) = 1

-- 解法二
SELECT
	# 此处 sum 函数: 统计有次日刷题记录的用户数量. 如果 p2.device_id 不为空, 说明用户在次日有刷题行为, 计数加1. count(*): 统计总的刷题记录数, 即总的 (device_id, date) 组合数量
    ROUND(SUM(CASE WHEN p2.device_id IS NOT NULL THEN 1 ELSE 0 END) / COUNT(*), 4) AS avg_ret
FROM
	# p1 -- 提取每个用户每天的唯一刷题记录
    (SELECT DISTINCT device_id, date FROM question_practice_detail) p1
LEFT JOIN
	# p2 -- 同样提取每个用户每天的唯一刷题记录, 用于与 p1 进行自连接
    (SELECT DISTINCT device_id, date FROM question_practice_detail) p2
ON
	# 连接条件为同一 device_id 且 p2.date 是 p1.date 的次日
	# 通过这种方式, 可以判断用户在某天刷题后次日是否有刷题记录
    p1.device_id = p2.device_id
    AND p2.date = DATE_ADD(p1.date, INTERVAL 1 DAY);
```









# 自选优质题目整理

## 牛客快速入门SQL26

```mysql
-- 选择并划分年龄段, 统计每个年龄段的用户数量
SELECT
    CASE 
        WHEN age < 25 OR age IS NULL THEN '25岁以下'
        ELSE '25岁及以上'
    END AS age_cut,
    COUNT(*) AS number
FROM
    user_profile
GROUP BY
    age_cut
ORDER BY # 根据需求将'25岁以下'的结果排在前面, '25岁及以上'的结果排在后面
    CASE # 通过 CASE WHEN 语句实现自定义排序
        WHEN age_cut = '25岁以下' THEN 1
        ELSE 2
    END;
-- 因为 ORDER BY 默认是升序, 所以 1 对应的"25岁以下"就排前面了
-- 想降序排列在END后加 desc 即可
```







# 上次做题位置记录

SQL快速入门 -- SQL34


































































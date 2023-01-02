1. 创建所有保险资产的详细记录视图

```postgresql
CREATE VIEW v_insurance_detail AS SELECT
    c_name, 
    c_id_card,
    i_name,
    i_project,
    pro_status,
    pro_quantity,
    i_amount,
    i_year,
    pro_income,
    pro_purchase_time
FROM client, property, insurance
WHERE c_id = pro_c_id and pro_type = 2 and pro_pif_id = i_id;
```

2. 基于视图的查询

```postgresql
SELECT 
    c_name, 
    c_id_card, 
    sum(pro_quantity * i_amount) as insurance_total_amount, 
    sum(pro_income) as insurance_total_revenue 
FROM v_insurance_detail
GROUP BY c_name, 
    c_id_card
ORDER BY insurance_total_amount DESC
```


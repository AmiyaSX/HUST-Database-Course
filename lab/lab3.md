1. 金融应用场景介绍,查询客户主要信息
```PostgreSQL
select c_name,c_phone,c_mail from client order by c_id;
```

2. 邮箱为null的客户
```PostgreSQL
select c_id, c_name, c_id_card, c_phone from client where c_mail is null
```

3. 查询既买了保险又买了基金的客户的名称和邮箱
```PostgreSQL
select c_name, c_mail, c_phone from client
	where c_id in (
		select pro_c_id from property where pro_type = 2
	)
	and c_id in (
		select pro_c_id from property where pro_type = 3
	)
	order by c_id;
```

4. 办理了储蓄卡的客户信息
```PostgreSQL
select
	c_name,
	c_phone,
	b_number
	from client INNER JOIN bank_card
	ON client.c_id = bank_card.b_c_id AND b_type = '储蓄卡'
	order by c_id;
```

5. 每份金额在30000～50000之间的理财产品
```PostgreSQL
-- 5) 查询理财产品中每份金额在30000～50000之间的理财产品的编号,每份金额，理财年限，并按照金额升序排序，金额相同的按照理财年限降序排序。
-- 请用一条SQL语句实现该查询：
select
	p_id,
	p_amount,
	p_year
	from finances_product
	where p_amount between 30000 and 50000
	order by p_amount, p_year DESC;
```

6. 商品收益的众数
``` PostgreSQL
-- 6) 查询资产表中所有资产记录里商品收益的众数和它出现的次数。
-- 请用一条SQL语句实现该查询：
select pro_income, count(pro_income) as presence
	from property
	group by pro_income
	having count(pro_income) >= all(
	select count(pro_income)
	from property
	group by pro_income
	)
```

7. 未购买任何理财产品的武汉居民
``` PostgreSQL
-- 7) 查询身份证隶属武汉市没有买过任何理财产品的客户的名称、电话号、邮箱。
-- 请用一条SQL语句实现该查询：
select c_name, c_phone, c_mail
	from client
	where c_id_card like '4201%'
		and c_id not in(
			select pro_c_id
			from property
			where pro_type = 1
		);
```

8. 持有两张及以上信用卡的客户
``` PostgreSQL
-- 8) 查询持有两张(含）以上信用卡的用户的名称、身份证号、手机号。
-- 请用一条SQL语句实现该查询：
select c_name, c_id_card, c_phone
	from client
	where c_id in (
		select b_c_id
		from bank_card
		group by b_type, b_c_id
		having count(b_c_id) >=2 and b_type = '信用卡'
	);
```

9. 购买了货币型基金的客户信息
``` PostgreSQL
-- 9) 查询购买了货币型(f_type='货币型')基金的用户的名称、电话号、邮箱。
-- 请用一条SQL语句实现该查询：
select c_name, c_phone, c_mail
	from client
	where c_id in (
		select pro_c_id from property
			where pro_type=3 and pro_pif_id in (
			select f_id from fund
			where f_type='货币型'
			)
	);
```

10. 投资总收益前三名的客户
``` PostgreSQL
-- 10) 查询当前总的可用资产收益(被冻结的资产除外)前三名的客户的名称、身份证号及其总收益，按收益降序输出，总收益命名为total_income。不考虑并列排名情形。
-- 请用一条SQL语句实现该查询：
select c_name, c_id_card, sum(pro_income) as total_income
	from client right JOIN property
	ON client.c_id=property.pro_c_id
	group by c_id, pro_status
	having property.pro_status='可用'
	order by total_income DESC limit 3;
```

11. 黄姓客户持卡数量
``` PostgreSQL
-- 11) 给出黄姓用户的编号、名称、办理的银行卡的数量(没有办卡的卡数量计为0),持卡数量命名为number_of_cards,
-- 按办理银行卡数量降序输出,持卡数量相同的,依客户编号排序。
-- 请用一条SQL语句实现该查询：
select c_id, c_name, count(b_c_id) AS number_of_cards
	from client full outer join bank_card
	on client.c_id=bank_card.b_c_id
	where c_name like '黄%'
	group by c_id
	order by number_of_cards DESC, c_id;
```

12. 客户理财、保险与基金投资总额
``` PostgreSQL
-- 12) 综合客户表(client)、资产表(property)、理财产品表(finances_product)、保险表(insurance)和
-- 基金表(fund)，列出客户的名称、身份证号以及投资总金额（即投资本金，
-- 每笔投资金额=商品数量*该产品每份金额)，注意投资金额按类型需要查询不同的表，
-- 投资总金额是客户购买的各类资产(理财,保险,基金)投资金额的总和，总金额命名为total_amount。
-- 查询结果按总金额降序排序。
-- 请用一条SQL语句实现该查询：
select
	c_name,
	c_id_card,
	COALESCE(sum(pro_account), 0) AS total_amount
	from client
	full join (
		select
		pro_c_id,
		pro_quantity * p_amount as pro_account
		from property, finances_product
		where pro_pif_id = p_id
		and pro_type = 1

		union all

		select
		pro_c_id,
		pro_quantity * i_amount as pro_account
		from property, insurance
		where pro_pif_id = i_id
		and pro_type = 2

		union all
		
		select
		pro_c_id,
		pro_quantity * f_amount as pro_account
		from property, fund
		where pro_pif_id = f_id
		and pro_type = 3
	) as pro on (pro.pro_c_id = c_id)
	group by c_id
	order by total_amount DESC;
```

13. 客户总资产
``` PostgreSQL
-- 13) 综合客户表(client)、资产表(property)、理财产品表(finances_product)、
-- 保险表(insurance)、基金表(fund)和投资资产表(property)，
-- 列出所有客户的编号、名称和总资产，总资产命名为total_property。
-- 总资产为储蓄卡余额，投资总额，投资总收益的和，再扣除信用卡透支的金额
-- (信用卡余额即为透支金额)。客户总资产包括被冻结的资产。
-- 请用一条SQL语句实现该查询：
select
    c_id,
    c_name, 
    COALESCE(sum(amount), 0) as total_property 
from client 
    left join (
        select
            pro_c_id, 
            pro_quantity * p_amount as amount 
        from property, finances_product 
        where pro_pif_id = p_id
        and pro_type = 1
        union all
        select
            pro_c_id,
            pro_quantity * i_amount as amount
        from property, insurance
        where pro_pif_id = i_id
        and pro_type = 2
        union all
        select
            pro_c_id,
            pro_quantity * f_amount as amount
        from property, fund
        where pro_pif_id = f_id
        and pro_type = 3
        union all
        select
            pro_c_id,
            sum(pro_income) as amount
        from property
        group by pro_c_id
        union all
        select
            b_c_id,
            sum(b_balance) as amount
        from bank_card
        where b_type = '储蓄卡'
        group by b_c_id
        union all
        select
            b_c_id,
            sum(-b_balance) as amount
        from bank_card
        where b_type = '信用卡'
        group by b_c_id
    ) pro
    on c_id = pro.pro_c_id
group by c_id
order by c_id;
```

14. 第N高问题
``` PostgreSQL
-- 14) 查询每份保险金额第4高保险产品的编号和保险金额。
--     在数字序列8000,8000,7000,7000,6000中，
--     两个8000均为第1高，两个7000均为第2高,6000为第3高。
-- 请用一条SQL语句实现该查询：
select i_id, i_amount 
    from insurance
    where i_amount = (
        select distinct i_amount
        from insurance
        order by i_amount DESC
        limit 3,1
    ) 
```

15. 基金收益两种方式排名
``` PostgreSQL
-- 15) 查询资产表中客户编号，客户基金投资总收益,基金投资总收益的排名(从高到低排名)。
--     总收益相同时名次亦相同(即并列名次)。总收益命名为total_revenue, 名次命名为rank。
--     第一条SQL语句实现全局名次不连续的排名，
--     第二条SQL语句实现全局名次连续的排名。

-- (1) 基金总收益排名(名次不连续)
select
    pro_c_id,
    sum(pro_income) as total_revenue,
    rank() over(order by total_revenue desc) as rank
    from property
    where pro_type = 3
    group by pro_c_id
    order by total_revenue desc;


-- (2) 基金总收益排名(名次连续)
select
    pro_c_id,
    sum(pro_income) as total_revenue,
    dense_rank() over(order by total_revenue desc) as rank
    from property
    where pro_type = 3
    group by pro_c_id
    order by total_revenue desc;
```

16. 持有完全相同基金组合的客户
``` PostgreSQL
-- 16) 查询持有相同基金组合的客户对，如编号为A的客户持有的基金，编号为B的客户也持有，反过来，编号为B的客户持有的基金，编号为A的客户也持有，则(A,B)即为持有相同基金组合的二元组，请列出这样的客户对。为避免过多的重复，如果(1,2)为满足条件的元组，则不必显示(2,1)，即只显示编号小者在前的那一对，这一组客户编号分别命名为c_id1,c_id2。

-- 请用一条SQL语句实现该查询：
select 
    pro_c_id1 as c_id1,
    pro_c_id2 as c_id2
    from(
        select 
            pro_c_id,
            string_agg(pro_pif_id , ' ' order by pro_pif_id) as pifset
        from property
        where pro_type = 3
        group by pro_c_id
    ) as p1(pro_c_id1,pifset1),
    (
        select 
            pro_c_id,
            string_agg(pro_pif_id, ' ' order by pro_pif_id) as pifset
        from property
        where pro_type = 3
        group by pro_c_id
    ) as p2(pro_c_id2,pifset2)
    where p1.pifset1 = p2.pifset2 and p1.pro_c_id1 < p2.pro_c_id2
    order by pro_c_id1;
```

17. 购买基金的高峰期
``` PostgreSQL

```

18. 至少有一张信用卡余额超过5000元的客户信用卡总余额
``` PostgreSQL

```

19. 以日历表格式显示每日基金购买总金额
``` PostgreSQL

```

20. 查询销售总额前三的理财产品
``` PostgreSQL

```

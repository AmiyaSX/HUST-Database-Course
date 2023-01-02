1. 插入多条完整的客户信息
```PostgreSQL
insert into client values
(1,'林惠雯','960323053@qq.com','411014196712130323','15609032348','Mop5UPkl'),
(2,'吴婉瑜','1613230826@gmail.com','420152196802131323','17605132307','QUTPhxgVNlXtMxN'),
(3,'蔡贞仪','252323341@foxmail.com','160347199005222323','17763232321','Bwe3gyhEErJ7');
```

2. 插入不完整的客户信息
```PostgreSQL
insert into client(c_id,c_name,c_phone,c_id_card,c_password) values
(33,'蔡依婷','18820762130','350972199204227621','MKwEuc1sc6');
```

3. 批量插入数据
```PostgreSQL
insert into client 
  select * from new_client
```

4. 删除没有银行卡的客户信息
```PostgreSQL
delete from client
    where not exists (
        select 1
        from bank_card
        where client.c_id = bank_card.b_c_id
    );
```

5. 冻结客户资产
```PostgreSQL
update property
    set pro_status='冻结'
    where pro_c_id in (
        select c_id 
            from client
            where c_phone='13686431238'
    );
```

6. 连接更新
``` PostgreSQL
update property
    set pro_id_card=(
        select c_id_card from client where pro_c_id=c_id
    );
```


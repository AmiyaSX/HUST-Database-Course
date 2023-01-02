1. 创建函数并在语句中使用它
```postgresql
CREATE OR REPLACE FUNCTION get_deposit(client_id integer) 
 returns numeric(10,2)
 LANGUAGE plpgsql
AS
$$
begin
    return (
        select
            sum(b_balance)
        from bank_card
        where b_type = '储蓄卡'
        group by b_c_id
        having b_c_id = client_id
    );
end;
$$;

/*  应用该函数查询存款总额在100万(含)以上的客户身份证号，姓名和存储总额(total_deposit)，
    结果依存款总额从高到代排序  */
    select * from (
        select c_id_card, c_name, get_deposit(c_id) as total_deposit
        from client
    ) a where total_deposit >= 1000000
        order by total_deposit desc;
```


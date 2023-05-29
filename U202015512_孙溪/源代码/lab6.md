1. 使用流程控制语句的存储过程
```postgresql
create procedure sp_fibonacci(in m int)
as
declare
 n int := 2;
 a int := 0;
 fibn int;
 b int := 1;
begin
if m > 0 then
	insert into fibonacci values(0,0);
end if;
if m > 1 then
	insert into fibonacci values(1,1);
end if;
    while n<m loop
	fibn := a + b;
	insert into fibonacci values(n,fibn);
	a := b;
	b := fibn;
	n := n + 1; 
    end loop;

end;
/
```

2. 使用游标的存储过程

```postgresql
create procedure sp_night_shift_arrange(in start_date date, in end_date date)
AS
    declare tp int default false;
    declare wk int default false;
    declare doc char(30);
    declare nur1 char(30);
    declare nur2 char(30);
    declare head char(30);
    cursor cur1 for select e_name from employee where e_type = 3;
    cursor cur2 for select e_type, e_name from employee where e_type < 3;
begin
    
    open cur1;
    open cur2;
    while start_date <= end_date loop
        fetch cur1 into nur1;
        if not found then
            close cur1;
            open cur1;
            fetch cur1 into nur1;
        end if;
        fetch cur1 into nur2;
        if not found then
            close cur1;
            open cur1;
            fetch cur1 into nur2;
        end if;
         wk := (extract(DOW FROM cast(start_date as TIMESTAMP)) + 7) % 8;
        if wk = 0 and head is not null then
            doc := head;
            head := null;
        else
            fetch cur2 into tp, doc;
            if not found then
                close cur2;
                open cur2;
                fetch cur2 into tp, doc;
            end if;
            if wk > 4 and tp = 1 then
                head := doc;
                fetch cur2 into tp, doc;
                if not found then
                    close cur2;
                    open cur2;
                    fetch cur2 into tp, doc;
                end if;
            end if;
        end if;
        insert into night_shift_schedule values (start_date, doc, nur1, nur2);
        SELECT start_date + INTERVAL '1 day' into start_date;
    end loop;
end;
```

3. 使用事务的存储过程

```postgresql
create procedure sp_transfer(IN applicant_id int,      
                     IN source_card_id char(30),
					 IN receiver_id int, 
                     IN dest_card_id char(30),
					 IN	amount numeric(10,2),
					 OUT return_code int)
as
begin
    update bank_card set b_balance = b_balance - amount 
        where b_number = source_card_id and b_c_id = applicant_id and b_type = '储蓄卡';
    update bank_card set b_balance = b_balance + amount 
        where b_number = dest_card_id and b_c_id = receiver_id and b_type = '储蓄卡';
    update bank_card set b_balance = b_balance - amount 
        where b_number = dest_card_id and b_c_id = receiver_id and b_type = '信用卡';

    if not exists(select * from bank_card 
        where b_number = source_card_id and b_c_id = applicant_id and b_type = '储蓄卡' and b_balance >= 0) then
        return_code := 0;
        rollback;
    elseif not exists(select * from bank_card where b_number = dest_card_id and b_c_id = receiver_id) then
        return_code := 0;
        rollback;
    else
        return_code := 1;
        commit;
    end if;
end; 
```

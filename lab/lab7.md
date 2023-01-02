1. 为投资表property实现业务约束规则-根据投资类别分别引用不同表的主码

```postgresql
CREATE OR REPLACE FUNCTION TRI_INSERT_FUNC() RETURNS TRIGGER AS
$$
DECLARE
   --此处用declare语句声明你所需要的变量
   declare tp int default new.pro_type;
   declare id int default new.pro_pif_id;
   declare msg varchar(50);
BEGIN
   --此处插入触发器业务
    if tp = 1 then
        if id not in (select p_id from finances_product) then
            msg := concat('finances product #', id, ' not found!');
        end if;
    elseif tp = 2 then
        if id not in (select i_id from insurance) then
            msg := concat('insurance #', id, ' not found!');
        end if;
    elseif tp = 3 then
        if id not in (select f_id from fund) then
            msg := concat('fund #', id, ' not found!');
        end if;
    else
        msg = concat('type ', tp, ' is illegal!');
    end if;
    if msg is not null then
        raise exception '%', msg;
    end if;

   --触发器业务结束
   return new;--返回插入的新元组
END;
$$ LANGUAGE PLPGSQL;

-- 创建before_property_inserted触发器，使用函数TRI_INSERT_FUNC实现触发器逻辑：
CREATE  TRIGGER before_property_inserted BEFORE INSERT ON property
FOR EACH ROW 
EXECUTE PROCEDURE TRI_INSERT_FUNC();
```


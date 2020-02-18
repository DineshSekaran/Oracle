# Oracle Stored function with DMl operation can be queried in select clause

create table t(id number,msg varchar2(255));

create sequence s;


create or replace function f1 return number as

v_return number;

pragma autonomous_transaction;

begin

begin

insert into t values(s.next_val,'called');  commit;

exception when others then 

v_return:=1;

end;

select max(id) into v_return from t;

return v_return;

end;

/


# oracle Function cant be query in below case

create or replace function myfunc return varchar2 as

begin

update emp set empno=empno+1  where empno=0;

commit;

return 'Yes';

end;

# Calling

set serveroutput on;

declare

v_var varchar2(60);

begin v_var:=myfunc();

dbms_output.put_line(v_var);

end;




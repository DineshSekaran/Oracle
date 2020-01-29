# ROW PROCESSING

declare

r1 emp%rowtype;

cursor c1 is

select * from emp;

begin

open c1 ; loop

fetch c1 into r1 ;

dbms_output.put_line(r1.ename||'-'||r1.sal);

exit when c1%notfound;

end loop;

close c1;

end;


# Bulk Processing

set serveroutput on;

declare

type t1 is ref cursor;

v_t1 t1;

type t_emp is table of emp%rowtype index by pls_integer;

v_t_emp t_emp;

c1 emp%rowtype;

begin

open v_t1 for select * from emp ; loop

fetch v_t1 bulk collect into v_t_emp limit 10;

for i in 1..v_t_emp.count loop

c1=v_t_emp(i);

dbms_output.put_line(c1.ename||'-'||c1.sal);

dbms_output.put_line(v_t_emp(i).ename||'-'||v_t_emp(i).sal);

exit when v_t_emp=0;

end loop;

exit when v_t1%notfound;

end loop;

close v_t1;
end;

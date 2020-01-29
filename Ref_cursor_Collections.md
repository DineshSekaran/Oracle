# ROW PROCESSING with Normal Cursor

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


# Bulk Processing With Ref Cursor

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


# Bulk Processing With normal Cursor and cursor%rowType and Type method


set serveroutput on;

declare

cursor c_name is

select * from emp;

type c_emp is table of c_name%rowtype index by pls_integer;

v_c_emp c_emp;


begin

open c_name ; loop

fetch c_name bulk collect into v_c_emp limit 10;

for i in 1..v_c_emp.count loop


dbms_output.put_line(v_c_emp(i).ename||'-'||v_c_emp(i).sal);

exit when v_c_emp=0;

end loop;

exit when c_name%notfound;

end loop;

close c_name;

end;


# BULK COLLECT With USER defined Exception and BULK Exceptions

 type array is table of t%rowtype index by binary_integer; 
 
 data array; 
 
 errors NUMBER; 
 
 dml_errors EXCEPTION; 
 
 l_cnt number := 0; 
 
 PRAGMA exception_init(dml_errors, -24381); 
 
 cursor c is select * from t; 
 
 BEGIN 
 
 open c; 
 
 loop 
 
 fetch c BULK COLLECT INTO data LIMIT 100; 
 
 begin 
 
 FORALL i IN 1 .. data.count SAVE EXCEPTIONS 
 
 insert into t2 values data(i); 
 
 EXCEPTION  WHEN dml_errors THEN 
 
 errors := SQL%BULK_EXCEPTIONS.COUNT; 
 
 l_cnt := l_cnt + errors; 
 
 FOR i IN 1..errors LOOP 
 
 dbms_output.put_line  ('Error occurred during iteration ' ||  SQL%BULK_EXCEPTIONS(i).ERROR_INDEX ||  ' Oracle error is ' || 
 SQL%BULK_EXCEPTIONS(i).ERROR_CODE ); 
 
 end loop; 
 
 end; 
 
exit when c%notfound; 

 END LOOP; 
 
 close c; 
 
 dbms_output.put_line( l_cnt || ' total errors' ); 
 
 end; 
/ 



# Collections Methods

set serveroutput on;

declare

type nt_tab is table of number;

col_var nt_tab:=nt_tab(10,20,30,40,50);

begin

for i in col_var.first.. col_var.last loop

dbms_output.put_line(col_var(i));

end loop;

end;

o/p: 10,20,30,40,50

# Delete

set serveroutput on;

declare

type nt_tab is table of number;

col_var nt_tab:=nt_tab(10,20,30,40,50);

begin
col_var.delete(1);

for i in col_var.first.. col_var.last loop

dbms_output.put_line(col_var(i));

end loop;

end;

o/p: 20,30,40,50

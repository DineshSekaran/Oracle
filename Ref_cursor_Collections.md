# ROW PROCESSING

declare

r1 emp%rowtype;

cursor c1 is

select * from emp;

begin

open c1 ; loop

fetch c1 into r1 ;

dbms_output.put_line(r1.name||'-'||r1.sal);

exit when c1%notfound;

end loop;

close c1;

end;

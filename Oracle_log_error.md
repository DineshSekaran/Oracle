# Error Log for Select insert into statement

Create table raises(empid number,sal number constraint check_sal check(sal)>8000));

Execute DBMS_ERRLOG.CREATE_ERROR_LOG('raises','errlog')

insert into raises

select empno,sal*1.8 sal from emp

LOG ERRORS into errlog('mybad');


select ORA_ERR_MESG$,emp_id,sal from errlog;


Note: If data volume is higher, row by row processing will be slower.so opting for above solution

------------------------------------------------

# Avoid termination of error in middle of loop and capture in log table

# Autonomous Error log

create or replace procedure log_error is

PRAGMA AUTONOMOUS_TRANSACTION;

c_code constant interger:=SQLCODE;

begin

insert into error_log(created_on,craeted_by,errcode,callstack,errorstack,backtrace,error_info)

values(SYSTimestamp,user,c_code,DBMS_UTILITY.format_CALL_SATCK,DBMS_UTILITY.FORMAT_ERROR_STACK,DBMS_UTILITY.FORMAT_ERROR_BACKTRACE,'N');

commit;

end;

/

DECLARE

cursor c1 is

selct * from emp;

begin

for i in c1 loop

begin

insert into raises(emp_id,sal) values(i.empno,i.sal*1.8);

excpetion when others then

log_Error;  --------> Error log Autonomous Procedure

end;

end loop;

commit;

end;

/

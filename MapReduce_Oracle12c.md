# MapReduce +Oracle =Table Functions

Ref:https://blogs.oracle.com/datawarehousing/mapreduce-oracle-tablefunctions

CREATE TABLE sls (salesman VARCHAR2(30), quantity number);

INSERT INTO sls VALUES('Tom', 100);
INSERT INTO sls VALUES('Chu', 200);
INSERT INTO sls VALUES('Tom', 300);
INSERT INTO sls VALUES('Mike', 100);
INSERT INTO sls VALUES('Scott', 300);
INSERT INTO sls VALUES('Tom', 250);
INSERT INTO sls VALUES('Scott', 100);

commit;


# Sample Package

create or replace package oracle_map_reduce
is

 type sales_t is table of sls%rowtype;
    type sale_cur_t is ref cursor return sls%rowtype;
    type sale_rec_t is record (name varchar2(30), total number);
    type total_sales_t is table of sale_rec_t;
    
    function mapper(inp_cur in sys_refcursor) return sales_t
    pipelined parallel_enable (partition inp_cur by any);
    
      function reducer(in_cur in sale_cur_t) return total_sales_t
    pipelined parallel_enable (partition in_cur by hash(salesman))
    cluster in_cur by (salesman);

end;



create or replace package body oracle_map_reduce
is

 function mapper(inp_cur in sys_refcursor) return sales_t
 pipelined parallel_enable (partition inp_cur by any)
    is
        sales_rec sls%ROWTYPE;
        begin
     
            loop
                fetch inp_cur into sales_rec;
                exit when inp_cur%notfound;
          
                pipe row (sales_rec);
        
            end loop;
            return;
        end mapper;
 

function reducer(in_cur in sale_cur_t) return total_sales_t
 pipelined parallel_enable (partition in_cur by hash(salesman))
 
    cluster in_cur by (salesman)
        IS

        sale_rec sls%ROWTYPE;
        total_sale_rec sale_rec_t;

          begin

            total_sale_rec.total := 0;
            total_sale_rec.name := NULL;

             loop
                fetch in_cur into sale_rec;
                exit when in_cur%notfound;

                if (total_sale_rec.name is null) then
      
                    total_sale_rec.name := sale_rec.salesman;
                    total_sale_rec.total := total_sale_rec.total + 
                    sale_rec.quantity;

                elsif ( total_sale_rec.name <> sale_rec.salesman) then

                     pipe row (total_sale_rec);
                    total_sale_rec.name := sale_rec.salesman;
                    total_sale_rec.total := sale_rec.quantity;

                else

                    total_sale_rec.total := total_sale_rec.total +
                    sale_rec.quantity;

                end if;
            end loop;

            if total_sale_rec.total<> 0 then
                pipe row (total_sale_rec);
            end if;

            return; 

        end reducer;
    end;
    
    
    
  # Sql Query
    
 select *
from table(oracle_map_reduce.reducer(cursor(
          select * from table(oracle_map_reduce.mapper(cursor(
                 select * from sls))) map_result)));

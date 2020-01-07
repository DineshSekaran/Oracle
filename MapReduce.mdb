#MapReduce +Oracle =Table Functions

CREATE TABLE sls (salesman VARCHAR2(30), quantity number)
/

INSERT INTO sls VALUES('Tom', 100);
INSERT INTO sls VALUES('Chu', 200);
INSERT INTO sls VALUES('Tom', 300);
INSERT INTO sls VALUES('Mike', 100);
INSERT INTO sls VALUES('Scott', 300);
INSERT INTO sls VALUES('Tom', 250);
INSERT INTO sls VALUES('Scott', 100);

commit;
/


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
/


create or replace package body oracle_map_reduce
is

    function mapper(inp_cur in sys_refcursor) return sales_t
    pipelined parallel_enable (partition inp_cur by any)
    is
        sales_rec sls%ROWTYPE;
        -- construct a record to hold an entire row from the SLS table
        begin
            -- First loop over all records in the table
            loop
                fetch inp_cur into sales_rec;
                exit when inp_cur%notfound;
                -- Place the found records from SLS into the variable
                -- end the loop when there are no more rows to loop over
                pipe row (sales_rec);
                -- by using pipe row here we are giving back rows in streaming
                -- fashion as you would expect from a table
                -- this in combination with pipelined in the definition allows
                -- the pipelining (e.g. giving data as it comes on board) of
                -- a table function
            end loop;
            return;

-- Return is a mandatory piece that allows the consumer of data (our reducer
-- in this case)
-- to ensure all data has been sent. After return the rowsource is exhausted
-- and no more data comes from this function.

        end mapper;

-- The above mapper does in effect nothing other than streaming data
-- partitioned

-- over to the next step. In MR the stream would be written to a file and then -- redistributed to the reducers

-- The reducer below computes and emits the sales figures
 

    function reducer(in_cur in sale_cur_t) return total_sales_t
    pipelined parallel_enable (partition in_cur by hash(salesman))
    -- The partition by clause indicates that all instances of a particular
    -- salesman must be sent to one instances of the reducer function

    cluster in_cur by (salesman)

    -- The cluster by clause tells the parallel query framework to cluster
    -- all instances of a particular salesman together.

        IS

        sale_rec sls%ROWTYPE;
        total_sale_rec sale_rec_t;

        -- two containers are created, one as input the other as output

        begin

            total_sale_rec.total := 0;
            total_sale_rec.name := NULL;

            -- reset the values to initial values
            loop
                fetch in_cur into sale_rec;
                exit when in_cur%notfound;

           -- some if then logic to ensure we pipe a row once all is processed

                if (total_sale_rec.name is null) then
           -- The first instance is arriving, set the salesman value to that
           -- input value
           -- update 0 plus the incoming value for total

                    total_sale_rec.name := sale_rec.salesman;
                    total_sale_rec.total := total_sale_rec.total + 
                    sale_rec.quantity;

                elsif ( total_sale_rec.name <> sale_rec.salesman) then

                -- We now switch sales man, and are done with the first
                -- salesman (as rows are partitioned and clustered)
                -- First pipe out the result of the previous salesman we
                -- processed
                -- then update the information to work on this new salesman
                    pipe row (total_sale_rec);
                    total_sale_rec.name := sale_rec.salesman;
                    total_sale_rec.total := sale_rec.quantity;

                else

                -- We get here when we work on the same salesman and just add
                -- the totals, the move on to the next record

                    total_sale_rec.total := total_sale_rec.total +
                    sale_rec.quantity;

                end if;
            end loop;

            -- The next piece of code ensures that any remaining rows that
            -- have not been piped out
            -- are piped out to the consumer. If there is a single salesman,
            -- he is only piped out
            -- in this piece of logic as we (in the above example code) only
            -- pipe out upon a change
            -- of salesman

            if total_sale_rec.total<> 0 then
                pipe row (total_sale_rec);
            end if;

            return;

            -- Again, we are now done and have piped all rows to our consumer

        end reducer;
    end;
    /
    
    
    # Sql Query
    
    select *
from table(oracle_map_reduce.reducer(cursor(
          select * from table(oracle_map_reduce.mapper(cursor(
                 select * from sls))) map_result)));

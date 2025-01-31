# Credit Card Transaction

* https://canarabank.in

## features
* view the master_table
### feature 1: list the master_table details


```sql
create table master_table(

    account_number number not null, constraint master_acc_no_pk primary key(account_number), 
    customer_name varchar2(25) not null,
    card_expired date not null,
    card_limit number not null, constraint card_limit_ck check(card_limit > 0),
    card_type varchar2(15) not null,
    card_number number not null, constraint master_card_number_unique unique(card_number),
    saveing_amount number not null);


---query

insert into master_table(account_number,customer_name,card_expired,card_limit,card_type,card_number,saveing_amount) 
    values (98765432101,'santhosh','01-nov-23',20000,'platinum',4315810647561004,2000);
insert into master_table(account_number,customer_name,card_expired,card_limit,card_type,card_number,saveing_amount)
    values (98765432564,'sam','01-jan-24',25000,'gold',3315510647561894,1000);
insert into master_table(account_number,customer_name,card_expired,card_limit,card_type,card_number,saveing_amount) 
    values (98765432876,'bala','01-aug-23',35000,'silver',45158226475210890,1080);

select * from master_table;



+--------------+---------------+-----------+------------+-----------+------------------+-----------------+
|account_number| customer_name | card_exp  | card_limit | card_type | card_number      | saveing_account |
+--------------+---------------+-----------+------------+-----------+------------------+-----------------+
| 98765432101  | santhosh      | 01-NOV-23 | 20300      | platinum  | 4315810647561004 | 2000            |
+--------------+---------------+-----------+------------+-----------+------------------+-----------------+
| 98765432564  | sam           | 01-JAN-24 | 25000      | gold      | 3315510647561894 | 1000            |
+--------------+---------------+-----------+------------+-----------+------------------+-----------------+
| 98765432876  | bala          | 01-AUG-23 | 35000      | silver    | 4515822647521004 | 1080            |
+--------------+---------------+-----------+------------+-----------+------------------+-----------------+
```
##features
* view credit_card detail
### feature 2: list the credit_card details

```sql
create table credit_details(
    credit_amount number(20),
    credit_date timestamp default sysdate,
    account_number number(20) not null,
    constraint credit_table_account_no_fk foreign key(account_number) references master_table(account_number),
    card_number number(20) not null);



insert into credit_details (credit_amount,account_number,card_number) values (1200,98765432101,4315810647561004);
insert into credit_details (credit_amount,account_number,card_number) values (5000,98765432564,3315510647561894);
insert into credit_details (credit_amount,account_number,card_number) values (4000,98765432876,4515822647521004);


select * from credit_details;

+---------------+---------------------------------+----------------+------------------+
| credit_amount | creadit_date                    | account_number | card_number      |
+---------------+---------------------------------+----------------+------------------+
| 1200          | 03-JAN-20 12.59.33.000000000 AM | 98765432101    | 4315810647561004 |
+---------------+---------------------------------+----------------+------------------+
| 5000          | 03-JAN-20 12.59.33.000000000 AM | 98765432564    | 3315510647561894 |
+---------------+---------------------------------+----------------+------------------+
| 4000          | 03-JAN-20 12.59.33.000000000 AM | 98765432876    | 4515822647521004 |
+---------------+---------------------------------+----------------+------------------+
```
### features 3: list the transaction details
```sql
create table transaction_details(
    account_number number(20)not null,
    constraint tran_table_acc_no_fk foreign key(account_number) references master_table(account_number),
    transaction_type varchar2(20) not null,
    transaction_amount number not null,
    constraint tran_amount_ck check(transaction_amount > 0),
    transaction_date timestamp default sysdate);


insert into transaction_details(account_number,transaction_type,transaction_amount) values (98765432101,'credit_bill',500);
insert into transaction_details(account_number,transaction_type,transaction_amount) values (98765432101,'credit_bill',300);


select * from transaction_details;

+----------------+------------------+--------------------+---------------------------------+
| account_number | transaction_type | transaction_amount | transaction_date                |
+----------------+------------------+--------------------+---------------------------------+
| 98765432101    | credit_bill      | 1200               | 01-DEC-19 12.00.00.000000000 AM |
+----------------+------------------+--------------------+---------------------------------+
| 98765432564    | credit_bill      | 5000               | 10-DEC-19 12.00.00.000000000 AM |
+----------------+------------------+--------------------+---------------------------------+
```



### features 4:check the credit limit using procedure

 ```sql
create or replace procedure prc_swipe_check(
    prc_account in number,             
    prc_card_number in number,
    prc_expired_date in date,
    prc_amount in number,
    status out varchar2) as
    v_check number;
    begin
        select count(1) into v_check from master_table where account_number= prc_account and card_number= prc_card_number and card_expired= prc_expired_date and (card_limit - prc_amount)>=0;
        if(v_check == 0) then
        status := 'transaction declind';
        return;
        end if;
        status := 'sucess';
        commit;
        end prc_swipe_check;


declare
    prc_account number(20) := 98765432101;      
    prc_card_number number(20) := 4315810647561004;
    prc_expired_date date := '01-NOV-23';
    prc_amount number(10) := 1000;
    status varchar2(23); 
    BEGIN
    prc_swipe(prc_account,prc_card_number,prc_expired_date,prc_amount,status);
    DBMS_OUTPUT.PUT_LINE(status);
    end; ```     
      
   ### features 5: update the credit limit into master_table using procedure
 
 ```sql 
 create or replace procedure prc_swipe(
    prc_account in number,             
    prc_card_number in number,
    prc_expired_date in date,
    prc_amount in number,
    status out varchar2) as
    v_check number;
    begin
              select count(1) into v_check from master_table where account_number= prc_account and card_number= prc_card_number and card_expired= prc_expired_date and (card_limit - prc_amount)>=0;
        if(v_check = 0) then
        status := 'transaction declined';
        return;
        end if;
        update master_table set card_limit=(card_limit - prc_amount) where account_number= prc_account and card_number= prc_card_number and card_expired= prc_expired_date and (card_limit - prc_amount)>=0;
        insert into credit_details (credit_amount,account_number,card_number)values (prc_amount,prc_account,prc_card_number);
        status := 'sucess ';
        commit;
        end prc_swipe;
        /

   
   
   
    run query:
    
    declare
    prc_account number(20) := 98765432101;      
    prc_card_number number(20) := 4315810647561004;
    prc_expired_date date := '01-NOV-23';
    prc_amount number(10) := 1000;
    status varchar2(23); 
    BEGIN
    prc_swipe(prc_account,prc_card_number,prc_expired_date,prc_amount,status);
    DBMS_OUTPUT.PUT_LINE(status);
    end;
   
 ```  
   
    
  ### features 5: triggers
  
  ```sql
  
create or replace trigger bill_record after insert on transaction_details for each row 
declare
begin
if(inserting) then
if(:new.transaction_type)='credit_bill' then 
update master_table set card_limit=(:new.transaction_amount + card_limit) where account_number=:new.account_number;
end if;
end if;
end;
/
```


### features 6: increaseing the credit limits


```sql

update master_table set card_limit=25000,card_type=gold where account_number=98765432101 and card_number=4315810647561004;

```

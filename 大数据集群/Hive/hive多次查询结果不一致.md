





```sql
select count(1) from 
(select b.*,d.* from 
(select a.* from 
(select *,row_number() over(partition by owner_code,balance_no , customer_no ,balance_date  order by  updated_at  desc ) as num from ods_st_paig_tst.ods_st_bmw_ac102011009 where owner_code ='D06C' and dt>='2023-03-01' and dt<'2023-04-01' )a  
where a.num=1)b
inner  join 
(select c.* from 
(select *,row_number() over(partition by owner_code, batch_no ,pub_pay_no,created_at  order by  updated_at  desc ) as num from ods_st_paig_tst.ods_st_bmw_ac102011009_item_list  where owner_code ='D06C' and payment_way_name <> '减免')c 
where c.num=1)d
on b.id=d.te_id )e;
```



![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230512173757.png)

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230512173821.png)

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230512173854.png)

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230512173914.png)

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230512175342.png)







# 排查过程
## 先查看两张表是是否有唯一值的字段
通过查询表数据，观察到id字段的值应该是唯一的

通过sql查询我们观察的是否为真

**ods_st_paig_tst.ods_st_bmw_ac102011009 **表的id列的值都是唯一的

```sql
select count(*) from (
select id from ods_st_paig_tst.ods_st_bmw_ac102011009 where owner_code ='D06C' and dt>='2023-03-01' and dt<'2023-04-01' GROUP BY id 
HAVING COUNT(*) > 1)a;
```

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230512182802.png)

**ods_st_paig_tst.ods_st_bmw_ac102011009_item_list** 表的te_id列的值都是唯一的

```sql
select count(*) from (
select te_id from ods_st_paig_tst.ods_st_bmw_ac102011009_item_list  where owner_code ='D06C' and payment_way_name <> '减免' GROUP BY te_id 
HAVING COUNT(*) > 1)a;
```

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230512183143.png)

## 多次查询id，并导出数据查看是否查询结果不一致

### 查看ds_st_paig_tst.ods_st_bmw_ac102011009 表的id列

```sql
with tmp_ods_st_bmw as (
select 
        id,
        row_number() over(partition by owner_code,balance_no , customer_no ,balance_date  order by  updated_at  desc ) as num 
        from ods_st_paig_tst.ods_st_bmw_ac102011009 
        where owner_code ='D06C' 
          and dt>='2023-03-01' 
          and dt<'2023-04-01' 
),
tmp_ods_st_bmw_t as (
select 
        te_id,
        row_number() over(partition by owner_code, batch_no ,pub_pay_no,created_at  order by  updated_at  desc ) as num 
from ods_st_paig_tst.ods_st_bmw_ac102011009_item_list  
where owner_code ='D06C' 
  and payment_way_name <> '减免')
select id from tmp_ods_st_bmw where num = 1 order by id;
```

- 通过文本对比软件对比，发现**ds_st_paig_tst.ods_st_bmw_ac102011009 **表的id列并没有变化

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230512185213.png)

### 查看ods_st_paig_tst.ods_st_bmw_ac102011009_item_list表的te_id列

```sql
with tmp_ods_st_bmw as (
select 
        id,
        row_number() over(partition by owner_code,balance_no , customer_no ,balance_date  order by  updated_at  desc ) as num 
        from ods_st_paig_tst.ods_st_bmw_ac102011009 
        where owner_code ='D06C' 
          and dt>='2023-03-01' 
          and dt<'2023-04-01' 
),
tmp_ods_st_bmw_t as (
select 
        te_id,
        row_number() over(partition by owner_code, batch_no ,pub_pay_no,created_at  order by  updated_at  desc ) as num 
from ods_st_paig_tst.ods_st_bmw_ac102011009_item_list  
where owner_code ='D06C' 
  and payment_way_name <> '减免')
select te_id from tmp_ods_st_bmw_t where num = 1 order by te_id
```

通过对比，发现**ods_st_paig_tst.ods_st_bmw_ac102011009_item_list**表多次查询出来的te_id的值不一致

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230512185702.png)

拿到对比的不一样的值的id再次查询，观察数据有什么区别

```sql
with tmp_ods_st_bmw as (
select 
        id,
        row_number() over(partition by owner_code,balance_no , customer_no ,balance_date  order by  updated_at  desc ) as num 
        from ods_st_paig_tst.ods_st_bmw_ac102011009 
        where owner_code ='D06C' 
          and dt>='2023-03-01' 
          and dt<'2023-04-01' 
),
tmp_ods_st_bmw_t as (
select 
        te_id,owner_code, batch_no ,pub_pay_no,created_at,updated_at,
        row_number() over(partition by owner_code, batch_no ,pub_pay_no,created_at  order by  updated_at  desc ) as num 
from ods_st_paig_tst.ods_st_bmw_ac102011009_item_list  
where owner_code ='D06C' 
  and payment_way_name <> '减免')
select owner_code, batch_no ,pub_pay_no,created_at,updated_at from tmp_ods_st_bmw_t where te_id = '1669876957110904' 
```

sql查询只有一条数据

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230512185945.png)

因为sql中有对update_at字段进行排序，我们就修改sql，使用上面sql查询出来的数据做条件，看看是否有两条相同的数据是te_id值不一样的

```sql
with tmp_ods_st_bmw as (
select 
        id,
        row_number() over(partition by owner_code,balance_no , customer_no ,balance_date  order by  updated_at  desc ) as num 
        from ods_st_paig_tst.ods_st_bmw_ac102011009 
        where owner_code ='D06C' 
          and dt>='2023-03-01' 
          and dt<'2023-04-01' 
),
tmp_ods_st_bmw_t as (
select 
        te_id,owner_code, batch_no ,pub_pay_no,created_at,updated_at,
        row_number() over(partition by owner_code, batch_no ,pub_pay_no,created_at  order by  updated_at  desc ) as num 
from ods_st_paig_tst.ods_st_bmw_ac102011009_item_list  
where owner_code ='D06C' 
  and payment_way_name <> '减免')
select te_id,owner_code, batch_no ,pub_pay_no,created_at,updated_at,num
from tmp_ods_st_bmw_t 
where owner_code = 'D06C'
 and batch_no = 'RS2022110400043-1'
 and pub_pay_no = 'RSI2022110400032'
 and created_at= '2022-11-04 11:35:10'
```

通过上面的sql可以看到相同的数据只是te_id的值不一样

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230512190337.png)



多次执行，发现num=1行的te_id是会经常变化的

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230512191052.png)

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230512191121.png)

![](https://niuzhan-1306014148.cos.ap-beijing.myqcloud.com/Typora/20230512191155.png)





# 结论

窗口函数在按照 **owner_code, batch_no ,pub_pay_no,created_at** 做分组的时候，updated_at字段的值是相同的，但是又因为id值不一样，所以取出来的num=1的id就不一样，也就导致了在inner join做关联的时候，数据不一致


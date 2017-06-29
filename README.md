# vue_lazy_table
a Vue table component using easy syntax to display complex data with minimum configutation

### Why vue-lazy table? 
Sometimes we get so many types of data from backend. When we feel lazy of Lableing all exist column , vue-lazy-table is a nice choice that require minimum settings be specified, all you need to do is pass the data.

![](http://i.imgur.com/YS0OAUc.png)

### minimum setting , auto key labeling
```
vue_lazy_table(:table_data = "devices")
```
![](http://i.imgur.com/ssqx26G.png)
### settings with row , support alias and post handler

```
vue_lazy_table(:table_data = "conclude_table"
                    :rows = "conclude_table_rows"

conclude_table_rows: [
  'total -> 人數總計',
  'avg -> 平均度數',
  'device_count -> 平均填寫電器'
]

//or 
conclude_table_rows: [
  {
    name: "total",
    as: "人數總計",
    handler: (num)=>(num*10)
  },
  'avg -> 平均度數',
  'device_count -> 平均填寫電器'
]

```
![](http://i.imgur.com/pw4DTXO.png)

### settings config

#### show_id -> auto add id in column if not found or not
#### show_search -> show search bar

```
vue_lazy_table(:table_data = "conclude_table"
               :rows = "conclude_table_rows"
               :configs = "{show_id: false, show_search: false}")
          

```
![](http://i.imgur.com/undefined.png)

## first commit with feature:
1. auto key extraction from raw json array data
2. easy row alias assignment  e.g. 'size -> TableSize' (if rows not assigned, auto generate label with key)
3. easy row hide set e.g. 'size -> TableSize | will not display'
4. allow row-setting syntax in string but also json object {name: '...', as: '...', handler: function(){}} , handler works like computed to post-progress column value, allowing adding unit or post caculation
5. auto id labeling / label sortable / searchable and paging (like Jquery data table)
6.allow :config {show_id: false, show_search: false}

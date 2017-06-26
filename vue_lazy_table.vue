<template lang="pug">
  .power_table
    div(v-if="conf.show_search")
      .form-group-inline
        label Search: 
        input(v-model="search_keyword")
    table.table.table-hover
      thead
        th(v-for = "row_key in (row_keys || default_row_keys)",
           @click = "set_sort_key(row_key)",
           v-if = "row_name_alias(row_key)!='__hide'")
          | {{ row_name_alias(row_key) }}
          span(v-if="row_key==sort_key && sort_direction") ▼
          span(v-if="row_key==sort_key && !sort_direction") ▲
          span(v-if="row_key!=sort_key ") 　
      tbody
        tr(v-for="(row,rid) in sliced_data", :key="row")
          td(v-for = "row_key in (row_keys || default_row_keys)",
             v-if = "row_name_alias(row_key)!='__hide'")
            | {{ row[row_key] }}
    .page_nav
      .btn.btn-default(v-if="pages.length>1",
                       v-for="p in pages",
                       :class="{active: page==p}",
                       @click="page=p") {{p}}
</template>

<script>
import Vue from 'vue'
// sorted -> sliced
export default {
  name: 'vue_lazy_table',
  props: ["table_data","row_keys","rows","configs"],
  data () {
    return {
      sort_key: null,
      sort_direction: true,
      conf: {
        show_id: true,
        show_search: true
      },
      search_keyword: "",
      page_split_num: 10,
      page: 1
    }
  },
  watch:{
  },
  mounted(){
    this.conf = Object.assign(this.conf,this.configs)
    console.log(this.configs)
  },
  computed: {
    default_row_keys(){
      //使用reduce 過濾初步重複欄位名稱（預設抓資料）
      let row_keys = Array.from(this.table_data).concat({}).reduce(
        (keys,data_row)=>keys.concat(Object.keys(data_row)),[]
      ).filter(
        (d,i,arr)=>arr.indexOf(d)==i
      )

      if (this.conf.show_id && row_keys.indexOf("id")==-1 ){
        row_keys.unshift("id")
      }

      return row_keys
    },
    sorted_data(){
      let data_clone = JSON.parse(JSON.stringify(this.table_data))

      let handlers = this.default_row_keys.map(key=>({key,handler: this.get_key_handler(key)}) )
      console.log(handlers)
      //if handler of key exist , compute handled value
      data_clone.forEach( (obj)=>{
        this.default_row_keys.forEach( (key)=>{
          let computed_handler= handlers.find(o=>o.key==key).handler
          if (computed_handler){
            obj[key]=computed_handler(obj[key])
          }
        })
      })

      if (data_clone){
        //add id col
        if (this.conf.show_id){data_clone.forEach( (o,id)=> {
            if (!o.id) { o.id = id+1}
          }
        )}

      
        //依照sortkey 排序
        let raw_sort = this.sort_key?data_clone.sort(
          (a,b)=>{
            var [var_a,var_b]= [a[this.sort_key],b[this.sort_key]]
            if (!isNaN(var_a))
              return parseFloat(var_a)< parseFloat(var_b)?1:-1
            else
              return var_a<var_b?1:-1
          }
        ):data_clone
        //search by content
        raw_sort=this.search_keyword==""?raw_sort:raw_sort.filter((o)=>Object.values(o).reduce((a,b)=>(a||(b+"").indexOf(this.search_keyword)!=-1),false)
        )

        return raw_sort
      }else{
        return []
      }
      
    },
    sliced_data(){
      let raw_sort = this.sorted_data
      let slice_pre = (this.sort_direction?raw_sort:raw_sort.reverse());
      let slice_post = slice_pre.slice( (this.page-1)*this.page_split_num,(this.page)*this.page_split_num )
      return slice_post
    },
    pages(){
      return Array.from( {length: Math.ceil(1.0*this.sorted_data.length / this.page_split_num) } , (d,i)=>i+1)
    },
    parse_items_list(){
      let list = this.rows?Array.from(this.rows):[];
      //可以用string "name | as" 也可以 {name: "name..",as: "as.."}
      let parse_items_list = 
        list.slice().map(o=>{
          if (typeof o == "string"){
            return {
              name: o.split(" -> ")[0],
              as: o.split(" -> ")[1].split(' | ')[0],
              arg: (o.split(" -> ")[1].split(' | ')[1]||"").split(",")
            }
          }else if  (typeof o == "object"){
            return Object.assign({
              as: null,
              arg: []
            },o)
          }
        })
      return parse_items_list
    }
  },
  methods: { 
    //取得表格欄位的暱稱
    row_name_alias(row_name){
      let parse_items_list = this.parse_items_list
      //尋找每一行的別名
      var find_row = parse_items_list.find(o=>o.name==row_name)
      // console.log(list,find_row)
      if (find_row){
        return find_row.arg.find(o=>o.indexOf("hide")!=-1)?"__hide":(find_row.as || row_name)
      }
      return row_name
    },
    set_sort_key(key){
      if (this.sort_key!=key){
      }else{
        this.sort_direction=!this.sort_direction
        
      }
      this.sort_key=key
    },
    get_key_handler(key){
      let key_obj = this.parse_items_list.find(o=>o.name==key)
      return key_obj?key_obj.handler:null
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="sass?indentedSyntax">
  *
    // border: solid 1px black
  th
    cursor: pointer
    user-select: none
    padding: 10px

  .fade-enter-active, .fade-leave-active 
    transition: opacity .5s

  .fade-enter, .fade-leave-to /* .fade-leave-active in <2.1.8 */ 
    opacity: 0

</style>

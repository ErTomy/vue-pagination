<script setup>
    import { ref, onMounted, reactive  } from 'vue';
    import axios from 'axios';

    const props = defineProps({
        search: {type:String, required: true},
        fields: {type:Object, required: true},
        buttons: Object, 
        tablebuttons: Object, 
        extras: Object,
        inputbox: Boolean, 
        translates: Object 
    });

    let loading = false;

    let headers = reactive({});
    let items = reactive({rows:[]});
    let pagination = {
        term: '',
        page: 1,
        last_page: 1,
        page_size: 10,
        total: 0,
        from: 0,
        to: 0
    };

    let language = {
      search:'Buscar',
      actions:'Acciones',
      next: 'Siguiente',
      previous: 'Anterior',
      showing: 'Mostrando',
      to: 'a',
      of: 'de',
      records: 'registros',
    };



    function loadData(){
        loading = true;
        axios
            .post(props.search, {page: pagination.page, page_size : pagination.page_size, term: pagination.term})
            .then(response => {
                items.rows = response.data.data;                               
                pagination = {
                    term: pagination.term,
                    page: response.data.current_page,
                    last_page: response.data.last_page,
                    page_size: response.data.per_page,
                    total: response.data.total,
                    from: response.data.from,
                    to: response.data.to
                }
                loading = false;
            })
            .catch(error => {
                console.log(error)                
            })
    }

    function gotoPage(page){
        pagination.page = page;
        loadData();
    }

    function search(){
      pagination.page = 1;
      loadData()

    }


    onMounted(() => {        

        // traducir los literales si llegan traducciones
        if(props.translates){
          for (const property in props.translates) {
            language[property] = props.translates[property];            
          }
        }

        headers = props.fields;
        loadData();
    });


</script>

<template>
   

    <div class="card mb-4" id="contact-list">
                  <div class="card-header d-lg-flex justify-content-between ">
                    <div class="d-grid d-lg-block">
                      <a v-for="(url, action) in tablebuttons" :key="action" :href="url" class="btn btn-primary me-2">{{ action }}</a>                       
                    </div>

                    <div class="d-flex mt-3 mt-lg-0">
                      <form action="#">
                        <div class="input-group " v-if="inputbox">
                          <input class="form-control rounded-3 search" type="search" v-model="pagination.term" id="searchInput" :placeholder="language.search" @keyup.enter="search()">
                          <span class="input-group-append">
                            <button class="btn  ms-n10 rounded-0 rounded-end" type="button" @click.stop.prevent="search()">
                              <svg xmlns="http://www.w3.org/2000/svg" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-search text-dark">
                                <circle cx="11" cy="11" r="8"></circle>
                                <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
                              </svg>
                            </button>
                          </span>
                        </div>
                      </form>
                      
                    </div>
                  </div>
                  <div class="card-body">
                    <div class="table-responsive table-card">
                      


<table class="table text-nowrap mb-0 table-centered" v-if="!loading">
        <thead class="table-light">
            <tr>
                <th >#</th>
                <th class="sort" data-sort="index" v-for="(field, index) in headers" :key="index">{{field}}</th>
                <th v-if="Object.keys(buttons).length > 0 || Object.keys(extras).length > 0" >{{language.actions}}</th>
            </tr>
        </thead>
        <tbody class="list contact-list-container">
            <tr v-for="item in items.rows" :key="item.id">                
                <td>{{ item.id }}</td>
                <td v-for="(field, index) in headers" :key="index">{{ item[index] }}</td>   
                <td v-if="Object.keys(buttons).length > 0 || Object.keys(extras).length > 0" >
                    <div class="d-flex align-items-center">

                    <template v-for="(url, action) in buttons" :key="action">
                      <a :href="url.replace('ID', item.id)" class="btn btn-icon btn-sm">
                          <i class="bi " :class="action"  ></i>
                      </a>
                    </template>  
                    

                    <div class="dropdown" v-if="Object.keys(extras).length > 0">
                        <a class="btn btn-ghost btn-icon btn-sm rounded-circle" href="#!" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-more-vertical icon-xs"><circle cx="12" cy="12" r="1"></circle><circle cx="12" cy="5" r="1"></circle><circle cx="12" cy="19" r="1"></circle></svg>
                        </a>
                        <ul class="dropdown-menu">
                          <li v-for="(url, action) in extras" :key="action"><a class="dropdown-item d-flex align-items-center view-item-btn" :href="url.replace('ID', item.id)">{{action}}</a></li>                        
                        </ul>
                    </div>
                    </div>
                </td>             
            </tr>
        </tbody>
    </table>




                    </div>
                  </div>

                  <div class="card-footer d-md-flex justify-content-between align-items-center ">
                    <span id="pagination-status">{{language.showing}} {{ pagination.from }} {{language.to}} {{ pagination.to }} {{language.of}} {{ pagination.total }} {{language.records}}}</span>
                    <nav class="mt-2 mt-md-0">
                      <div class="d-flex justify-content-end mt-3">
                        <div class="pagination-wrap hstack">
                          <a class="page-item pagination-prev" href="#" @click.stop.prevent="gotoPage(pagination.page -1)" v-if="pagination.page > 1">
                            {{language.previous}}
                          </a>
                          <ul class="pagination listjs-pagination mb-0">
                                <li v-for="index in pagination.last_page" :key="index" :class="index == pagination.page? 'active':'' ">
                                    <a class="page" href="#" @click.stop.prevent="gotoPage(index)">{{index}}</a>
                                </li>
                          </ul>
                          <a class="page-item pagination-next" href="#" @click.stop.prevent="gotoPage(pagination.page +1)" v-if="pagination.page < pagination.last_page">
                            {{language.next}}
                          </a>
                        </div>
                      </div>
                    </nav>
                  </div>

                </div>








    
    

</template>

<style>

.hstack, .vstack
{
  align-self: stretch;
  display: flex;
}
.hstack
{
  align-items: center;
  flex-direction: row;
}

.pagination-wrap a.disabled
{
  background-color: #ccc;
  color: var(--dashui-pagination-disabled-color);
  pointer-events: none;
}
.pagination-wrap .page-item
{
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: .375rem;
  color: #999;
  display: block;
  font-size: .875rem;
  margin-right: .25rem;
  padding: .375rem .75rem;
  position: relative;
}

.pagination
{
  --dashui-pagination-padding-x: 0.75rem;
  --dashui-pagination-padding-y: 0.375rem;
  --dashui-pagination-font-size: 0.875rem;
  --dashui-pagination-color: #999999;
  --dashui-pagination-bg: #fff;
 /* --dashui-pagination-border-width: var(--dashui-border-width); */
  --dashui-pagination-border-color: #624bff;
 /* --dashui-pagination-border-radius: var(--dashui-border-radius); */
  --dashui-pagination-hover-color: #999;
  --dashui-pagination-hover-bg: #ccc;
  --dashui-pagination-hover-border-color: #ccc;
  --dashui-pagination-focus-color: #fff;
  --dashui-pagination-focus-bg: #624bff;
  --dashui-pagination-focus-box-shadow: 0 0 0 0.25rem rgba(98,75,255,.25);
  --dashui-pagination-active-color: #fff;
  --dashui-pagination-active-bg: #624bff;
  --dashui-pagination-active-border-color: #624bff;
  --dashui-pagination-disabled-color: #ddd;
  --dashui-pagination-disabled-bg: #ccc;
  --dashui-pagination-disabled-border-color: #ccc;
  display: flex;
  list-style: none;
  padding-left: 0;
}

.pagination-wrap .listjs-pagination li.active
{
  background-color: #624bff;
  border: 1px solid #624bff;
  color: #fff !important;

}
.pagination-wrap .listjs-pagination li
{
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: .375rem;
  color: #999;
  font-size: .875rem;
  margin-right: .25rem;
  position: relative;
}
.pagination-wrap .listjs-pagination li.active .page
{
  color: #fff !important;
}
.pagination-wrap .listjs-pagination .page
{
  color: #999;
  display: block;
  padding: .375rem .75rem;
}

.pagination-wrap .listjs-pagination li
{
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: .375rem;
  color: #999;
  font-size: .875rem;
  margin-right: .25rem;
  position: relative;
}
</style>
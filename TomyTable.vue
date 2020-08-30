<template>
    
    <div class="card">

        <div class="modal fade bd-example-modal-lg" data-backdrop="static" data-keyboard="false" tabindex="-1">
            <div class="modal-dialog modal-sm">
                <div class="modal-content" style="width: 48px">
                    <span class="fa fa-spinner fa-spin fa-3x"></span>
                </div>
            </div>
        </div>


        <div class="card-header container-fluid">
            <div class="row">
                <div class="col-md-6">
                    <h4>{{label}}</h4>
                </div>
                <div class="col-md-6 float-right">
                    <div class="input-group mb-3">
                        <input type="text" class="form-control" placeholder="Buscar" v-model="search" v-on:keyup.enter="paginate(1)">
                        <div class="input-group-append">
                            <button class="btn btn-primary" type="button" v-on:click.stop.prevent="paginate(1)">
                                <i class="fa fa-search" aria-hidden="true"></i>
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="card-body">
            <table class="table table-hover">
            <thead>
                <tr>
                    <th scope="col" v-for="column of columns">{{column}}</th>
                    <th scope="col" v-if="detail">#</th>                
                </tr>
            </thead>
            <tbody>
                <tr v-for="item of items">
                    <td v-for="column of columns">{{item[column]}}</td>
                    <th scope="row" v-if="detail">
                        <a class="btn btn-primary"  :href="detail.replace('ID', item.id)">Ver</a>                    
                    </th>
                </tr>                
            </tbody>
            </table>

            <nav aria-label="Page navigation example" v-if="pages > 1">
                <ul class="pagination justify-content-center">
                    <li class="page-item" :class="{ disabled: (current == 1) }">
                        <a class="page-link" href="#" tabindex="-1" v-on:click.stop.prevent="paginate(current - 1)">Anterior</a>
                    </li>
                    <li class="page-item disabled" v-if="start > 1"><a class="page-link" href="#">...</a></li> 
                    <li class="page-item" v-for="(n,index) in pages" :class="{ active: (current == index + 1) }" v-if="(index + 1) >= start && (index + 1) <= end">
                        <a class="page-link" href="#" v-on:click.stop.prevent="paginate(index + 1)">{{index + 1}}</a>
                    </li>                
                    <li class="page-item disabled" v-if="end < pages"><a class="page-link" href="#">...</a></li> 
                    <li class="page-item" :class="{ disabled: (current == pages) }">
                        <a class="page-link" href="#" v-on:click.stop.prevent="paginate(current + 1)">Siguiente</a>
                    </li>
                </ul>
            </nav>

            <small class="form-text text-muted">Mostrando de {{total.from}} a {{total.to}} de {{total.total}}</small>

        </div>
    </div>
    
</template>

<script>
    export default {
        props: {
            label: {
                type: String,
                required: true
            },
            url: {
                type: String,
                required: true
            },
            detail: {
                type: String
            },
            records: {
                type: Number,
                default: 10
            }            
        },
        data: function (){
            return {
                search: '',
                columns: [],
                items: [], 
                pages: 0,
                current: 0,
                start: 0,
                end: 0,
                total:{
                    from:0,
                    to:0,
                    total:0
                }
            }
        },
        methods: {
            paginate: function (page) {
                
                let _self = this;
                $('.modal').modal('show');
                axios.post(this.url + '?page=' + page, {
                    search: _self.search,
                    records: _self.records
                })
                .then(function (response) {
                    if(_self.columns.length === 0 && response.data.data.length > 0){
                        Object.getOwnPropertyNames(response.data.data[0]).forEach(function(val, idx, array) {
                            _self.columns.push(val);
                        });
                    }
                    _self.items = response.data.data;
                    _self.pages = response.data.last_page;
                    _self.current = response.data.current_page;
                    _self.total.from = (response.data.from === null)?0:response.data.from;
                    _self.total.to = (response.data.to === null)?0:response.data.to;
                    _self.total.total = response.data.total;
                    _self.start = (_self.current > 3)? (_self.current - 3):1; 
                    _self.end = ((_self.current + 3) < _self.pages)?(_self.current + 3):_self.pages; 
                    $('.modal').modal('hide');
                })
                .catch(function (error) {
                    console.error(error);
                });
            }
        },
        mounted() {
            this.paginate(1);            
        }
    }
</script>

<style scope>
    .bd-example-modal-lg .modal-dialog{
        display: table;
        position: relative;
        margin: 0 auto;
        top: calc(50% - 24px);
    }
  
    .bd-example-modal-lg .modal-dialog .modal-content{
        background-color: transparent;
        border: none;
    }
</style>
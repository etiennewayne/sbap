<template>
    <div>
        <div class="section">
            <div class="columns is-centered">
                <div class="column is-11-widescreen is-12-desktop is-12-tablet">
                    <div class="box">

                        <div class="is-flex is-justify-content-center mb-2" style="font-size: 20px; font-weight: bold;">SUPPLEMENTAL BUDGET RECORDS</div>

                        <div class="level">
                            <div class="level-left">
                                <b-field label="Page">
                                    <b-select v-model="perPage" @input="setPerPage">
                                        <option value="5">5 per page</option>
                                        <option value="10">10 per page</option>
                                        <option value="15">15 per page</option>
                                        <option value="20">20 per page</option>
                                    </b-select>
                                    <b-select v-model="sortOrder" @input="loadAsyncData">
                                        <option value="asc">ASC</option>
                                        <option value="desc">DESC</option>

                                    </b-select>
                                </b-field>
                            </div>

                            <div class="level-right">
                                <div class="level-item">
                                    <b-field label="Search">
                                        <b-input type="text"
                                                 v-model="search.key" placeholder="Search..."
                                                 @keyup.native.enter="loadAsyncData"/>
                                        <p class="control">
                                             <b-tooltip label="Search" type="is-success">
                                            <b-button type="is-primary" icon-right="account-filter" @click="loadAsyncData"/>
                                             </b-tooltip>
                                        </p>
                                    </b-field>
                                </div>
                            </div>
                        </div>

                        <div class="buttons mt-3 is-right" v-if="propUser.role !== 'STAFF'">
                            <b-button tag="a"
                                href="/augmentation-budgets/create"
                                icon-right="mower"
                                class="is-primary">ADD RECORD</b-button>
                        </div>

                        <b-table
                            :data="data"
                            :loading="loading"
                            paginated
                            backend-pagination
                            :total="total"
                            :bordered="true"
                            :hoverable="true"
                            :per-page="perPage"
                            @page-change="onPageChange"
                            aria-next-label="Next page"
                            aria-previous-label="Previous page"
                            aria-page-label="Page"
                            aria-current-label="Current page"
                            backend-sorting
                            :default-sort-direction="defaultSortDirection"
                            @sort="onSort">

                            <b-table-column field="Id" label="ID" v-slot="props">
                                {{ props.row.augmentation_budget_id }}
                            </b-table-column>

                            <b-table-column field="date_time" label="Date" v-slot="props">
                                {{ new Date(props.row.created_at).toLocaleDateString() }}
                            </b-table-column>

                            <b-table-column field="remarks" label="Remarks" v-slot="props">
                                {{ props.row.remarks }}
                            </b-table-column>

                            <b-table-column field="object_expenditure_id_from" label="Account (OE)" v-slot="props">
                                {{ props.row.object_expenditure.object_expenditure }}
                            </b-table-column>

                            <b-table-column field="amount_transfer" label="Amount" v-slot="props">
                                {{ props.row.amount_transfer | numberWithCommas }}
                            </b-table-column>

                            <!-- <b-table-column label="Action" v-slot="props" v-if="propUser.role !== 'STAFF'">
                                <div class="is-flex">
                                    <b-tooltip label="Edit" type="is-warning">
                                        <b-button class="button is-small is-warning mr-1 is-outlined" 
                                            tag="a" 
                                            icon-right="pencil" 
                                            :href="`/procurements/${props.row.accounting_id}/edit`"></b-button>
                                    </b-tooltip>
                                    <b-tooltip label="Delete" type="is-danger">
                                        <b-button class="button is-small is-danger mr-1 is-outlined" 
                                            icon-right="delete"
                                            @click="confirmDelete(props.row.accounting_id)"></b-button>
                                    </b-tooltip>
                                </div>
                            </b-table-column> -->


                        </b-table>

                        <!-- <div class="buttons">
                            <download-excel
                                class="button is-primary"
                                size="is-small"
                                :fetch="fetchData"
                                type="csv"
                                name="filename.xls"
                            >
                                Download to Excel
                            </download-excel>
                        </div> -->

                    </div>
                </div><!--col -->
            </div><!-- cols -->
        </div><!--section div-->


    </div>
</template>

<script>

export default{

    props: {
        propUser: {
            type: Object,
            return: function(){
                return {}
            }
        }
    },

    data() {
        return{
            data: [],
            total: 0,
            loading: false,
            sortField: 'augmentation_budget_id',
            sortOrder: 'desc',
            page: 1,
            perPage: 20,
            defaultSortDirection: 'asc',

            global_id : 0,

            search: {
                key: '',
            },

            isModalCreate: false,

            fields: {
                financial_year_id: null,
                remarks: null,
                object_expenditure_id_from: null,
                object_expenditure_id_to: null,
                amount_transfer: null,
            },
            errors: {},


            btnClass: {
                'is-success': true,
                'button': true,
                'is-loading':false,
            },



        }

    },

    methods: {
        /*
        * Load async data
        */
        loadAsyncData() {
            const params = [
                `sort_by=${this.sortField}.${this.sortOrder}`,
                `key=${this.search.key}`,
                `perpage=${this.perPage}`,
                `page=${this.page}`
            ].join('&')

            this.loading = true
            axios.get(`/get-augmentation-budgets?${params}`)
                .then(({data}) => {
                    this.data = [];
                    let currentTotal = data.total
                    if (data.total / this.perPage > 1000) {
                        currentTotal = this.perPage * 1000
                    }

                    this.total = currentTotal
                    data.data.forEach((item) => {
                        //item.release_date = item.release_date ? item.release_date.replace(/-/g, '/') : null
                        this.data.push(item)
                    })
                    this.loading = false
                })
                .catch((error) => {
                    this.data = []
                    this.total = 0
                    this.loading = false
                    throw error
                })
        },
        /*
        * Handle page-change event
        */
        onPageChange(page) {
            this.page = page
            this.loadAsyncData()
        },

        onSort(field, order) {
            this.sortField = field
            this.sortOrder = order
            this.loadAsyncData()
        },

        setPerPage() {
            this.loadAsyncData()
        },

        openModal() {
            this.isModalCreate = true;
            this.clearFields()
            this.errors = {};
        },

        clearFields(){
            this.fields.financial_year_id = null
            this.fields.remarks = null
            this.fields.object_expenditure_id = 0
            this.fields.amount_transfer = null
        },

        //alert box ask for deletion
        confirmDelete(delete_id) {
            this.$buefy.dialog.confirm({
                title: 'DELETE!',
                type: 'is-danger',
                message: 'Are you sure you want to delete this data?',
                cancelText: 'Cancel',
                confirmText: 'Delete',
                onConfirm: () => this.deleteSubmit(delete_id)
            });
        },
        //execute delete after confirming
        deleteSubmit(delete_id) {
            axios.delete('/procurements/' + delete_id).then(res => {
                this.loadAsyncData();
            }).catch(err => {
                if (err.response.status === 422) {
                    this.errors = err.response.data.errors;
                }
            });
        },

        async fetchData(){
            const res = await axios.get('/fetch-procurements')
            return res.data
        }
    },


    mounted() {
        this.loadAsyncData();

    }
}
</script>


<style>


</style>

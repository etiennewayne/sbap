<template>
    <div>
        <div class="section">
            <div class="columns is-centered">
                <div class="column is-10-widescreen is-12-desktop is-12-tablet">
                    <div class="box">

                        <div class="is-flex is-justify-content-center mb-2" style="font-size: 20px; font-weight: bold;">PROCUMENT RECORDS</div>

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
                                href="/procurements/create"
                                icon-right="mower"
                                class="is-primary">ADD RECORD</b-button>
                        </div>

                        <div>
                            <span class="">UTILIZE BUDGET: </span>
                            <span class="has-text-weight-bold" style="font-family: Consolas, monaco, monospace;">{{ utilizeAmount | numberWithCommas }}</span>
                        </div>

                        <b-table
                            :data="data"
                            :loading="loading"
                            paginated
                            backend-pagination
                            :total="total"
                            :bordered="true"
                            :hoverable="true"
                            detailed
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
                                {{ props.row.accounting_id }}
                            </b-table-column>

                            <b-table-column field="date_time" label="Date & Time" v-slot="props">
                                {{ props.row.date_transaction }}
                            </b-table-column>

                            <b-table-column field="training_control_no" label="Training/Activity No" v-slot="props">
                                {{ props.row.training_control_no }}
                            </b-table-column>

                            <b-table-column field="pr_number" label="PR No" v-slot="props">
                                {{ props.row.pr_no }}
                            </b-table-column>

                            <b-table-column field="payee" label="Payee" v-slot="props">
                                <span v-if="props.row.payee"> {{ props.row.payee.bank_account_payee }}</span>
                            </b-table-column>
                            
                            <b-table-column field="pr_amount" label="PR Amount" v-slot="props">
                                <span style="font-family: Consolas, monaco, monospace;">
                                    {{ props.row.total_amount | numberWithCommas}}
                                </span>
                            </b-table-column>

                            <b-table-column field="particulars" label="Particulars" v-slot="props">
                                {{ props.row.particulars }}
                            </b-table-column>


                            <b-table-column label="Action" v-slot="props" v-if="propUser.role !== 'STAFF'">
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
                            </b-table-column>

                            <template #detail="props">
                                <table>
                                    <tr>
                                        <th>Documentary Attachment</th>
                                        <th>File</th>
                                    </tr>
                                    <tr v-for="(i, ix) in props.row.accounting_documentary_attachments" :key="ix">
                                        <td>{{ i.documentary_attachment.documentary_attachment }}</td>
                                        <td>
                                            <a :href="`/storage/doc_attachments/${i.doc_attachment}`" target="_blank">
                                                Go to
                                            </a>
                                        </td>
                                    </tr>
                                </table>

                                <table class="mt-2">
                                    <tr>
                                        <th>Allotment Class</th>
                                        <th>Allotment Class Account</th>
                                        <th>Object Expenditure</th>
                                        <th>Priority Program</th>
                                        <th>Amount</th>
                                    </tr>
                                    <tr v-for="(i, ix) in props.row.accounting_expenditures" :key="ix">
                                        <td>{{ i.object_expenditure.allotment_class.allotment_class }}</td>
                                        <td>({{ i.object_expenditure.allotment_class.allotment_class_code }})</td>
                                        <td>{{ i.object_expenditure.object_expenditure }}</td>
                                        <td>{{ i.priority_program }}</td>
                                        <td>{{ i.amount }}</td>
                                    </tr>
                                </table>

                            </template>

                        </b-table>

                        <div class="buttons">
                            <download-excel
                                class="button is-primary"
                                size="is-small"
                                :fetch="fetchData"
                                type="xls"
                                name="procurement.xls"
                            >
                                Download to Excel
                            </download-excel>
                        </div>

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
            sortField: 'accounting_id',
            sortOrder: 'desc',
            page: 1,
            perPage: 20,
            defaultSortDirection: 'asc',

            global_id : 0,

            search: {
                key: '',
            },

            isModalCreate: false,


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
            axios.get(`/get-procurements-records?${params}`)
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

    },

    computed: {
        utilizeAmount(){
            let total = 0;
            this.data.forEach(item => {
                total += Number(item.total_amount)
            });
            return total;
        }
    }
}
</script>


<style>


</style>

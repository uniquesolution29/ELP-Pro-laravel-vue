<template>
	<div v-if="show">
		<div class="card">
			<div class="card-header card-header-flex pb-2">
				<div class="w-100 mt-2">
					<div class="row">
						<div class="col-8">
							<h5 class="mt-3 ml-0 mr-3 mb-2">
								<strong>
									<template v-if="operation === null">{{$t('title.contacts')}}</template>
									<template v-else>{{ $t(operationTitle) }}</template>
								</strong>
							</h5>
						</div>
						<div class="col-4 text-right">
							<div v-if="operation === null">
								<div class="mt-3">
									<!-- Using components -->
									<b-input-group class="mt-3">
										<b-form-input type="search" class="form-control form-control-sm"
										              :placeholder="$t('input.whatAreYouLookingFor')"
										              v-on:keyup.enter="handleSearch" v-model="search"></b-form-input>
										<b-input-group-append>
											<b-button @click="setOperation('add')" variant="info" size="sm"
											          :title="$t('button.title.addNewItem')"
											          v-if="$global.hasPermission('contactsstore')" v-b-tooltip.hover>
												<i class="fe fe-plus"></i> {{$t('button.addNew')}}
											</b-button>
											<b-button size="sm" :title="$t('button.title.filterRecords')"
											          variant="outline-info"
											          @click="filters.visible = !filters.visible" v-b-tooltip.hover
											          v-if="$global.hasPermission('contactsview')">
												<i class="fa fa-filter"></i>
											</b-button>
											<b-button size="sm" :title="$t('button.title.resetList')"
											          variant="outline-info"
											          @click="handleResetClick()" v-b-tooltip.hover>
												<i class="fa fa-refresh"></i>
											</b-button>
										</b-input-group-append>
									</b-input-group>
								</div>
							</div>
							<div v-else>
								<b-button variant="warning" size="sm" class="mt-3"
								          @click="handleOperationClose()"
								          v-b-tooltip.hover :title="$t('button.title.cancel')">
									<i class="fa fa-arrow-left"></i> {{$t('button.cancel')}}
								</b-button>
							</div>
						</div>
					</div><!-- /.row -->
				</div><!-- /.w-100 -->
			</div><!-- /.card-header -->
			<div class="card-body">
                <div class="contacts-table">
                    <b-table hover responsive
                             ref="table"
                             :busy="listingLoading"
                             :items="dataSource"
                             :fields="columns"
                             :sort-by.sync="sortField"
                             @sort-changed="handleSortChange">
                        <template v-slot:cell(id)="record">
                            {{ record.index + 1 }}
                        </template>
                        <template v-slot:cell(action)="record">
                            <a @click="setOperation('edit', record.item.id)"
                               :title="$t('button.title.editItem')" v-if="$global.hasPermission('contactsupdate')"
                               v-b-tooltip.hover>
                                <i class="fe fe-edit"></i>
                            </a>
                            <a-popconfirm title="Are you sure？" @confirm="handleDeleteClick(record.item.id)"
                                          v-if="$global.hasPermission('contactsdestroy')">
                                <i slot="icon" class="fe fe-trash"></i>
                                <a class=" ml-1"
                                   :title="$t('button.title.deleteItem')"
                                   v-b-tooltip.hover>
                                    <i class="fe fe-trash"></i>
                                </a>
                            </a-popconfirm>
                        </template>
                    </b-table><!-- /.b-table -->
                    <div class="clearfix">
                        <div class="float-right">
                            <b-pagination
                                v-model="pagination.current"
                                :total-rows="pagination.total"
                                :per-page="pagination.perPage"
                                :first-text="$t('paginations.first')"
                                :prev-text="$t('paginations.prev')"
                                :next-text="$t('paginations.next')"
                                :last-text="$t('paginations.last')"
                            ></b-pagination>
                        </div><!-- /.pull-right -->
                    </div><!-- /.clearfix -->
                </div><!-- /.contacts-table -->
                <div class="contacts-operation">
                    <a-drawer
                        placement="right"
                        :width="'650px'"
                        :wrapStyle="{overflow: 'auto',paddingBottom: '108px'}"
                        :closable="false"
                        @close="handleOperationClose"
                        :visible="operation !== null && operation !== 'detail'"
                        :zIndex="10"
                        :title="$t(operationTitle)"
                    >
                        <form @submit.prevent="handleSubmit" autocomplete="off">
                            <b-row>
                                <b-col cols="12">
	                                <b-row>
                                        <b-col sm="6">
                                            <b-form-group
                                                :label="$t('input.firstName')+' *'"
                                                label-for="first_name"
                                                :invalid-feedback="formErrors.first('first_name')"
                                            >
                                                <b-form-input
                                                    id="first_name"
                                                    v-model="formFields.first_name"
                                                    type="text"
                                                    :state="((formErrors.has('first_name') ? false : null))"
                                                    @focus="$event.target.select()"
                                                ></b-form-input>
                                            </b-form-group>
                                        </b-col><!--/b-col-->
                                        <b-col sm="6">
                                            <b-form-group
                                                :label="$t('input.lastName')"
                                                label-for="last_name"
                                                :invalid-feedback="formErrors.first('last_name')"
                                            >
                                                <b-form-input
                                                    id="last_name"
                                                    v-model="formFields.last_name"
                                                    type="text"
                                                    :state="((formErrors.has('last_name') ? false : null))"
                                                    @focus="$event.target.select()"
                                                ></b-form-input>
                                            </b-form-group>
                                        </b-col><!--/b-col-->
                                    </b-row><!--/b-row-->
                                    <b-row>
                                        <b-col sm="12">
                                            <b-form-group
                                                :label="$t('input.email')+' *'"
                                                label-for="email"
                                                :invalid-feedback="formErrors.first('email')"
                                            >
                                                <b-form-input
                                                    id="email"
                                                    v-model="formFields.email"
                                                    type="text"
                                                    :state="((formErrors.has('email') ? false : null))"
                                                    @focus="$event.target.select()"
                                                ></b-form-input>
                                            </b-form-group>
                                        </b-col><!--/b-col-->
                                    </b-row><!--/b-row-->
	                                <b-row>
                                        <b-col sm="6">
                                            <b-form-group
                                                :label="$t('input.mobile')+' *'"
                                                label-for="mobile"
                                                :invalid-feedback="formErrors.first('mobile')"
                                            >
                                                <b-form-input
                                                    id="mobile"
                                                    v-model="formFields.mobile"
                                                    type="number"
                                                    pattern="^[0-9-+()]*"
                                                    title="+(XXX) XXX"
                                                    :state="((formErrors.has('mobile') ? false : null))"
                                                    @focus="$event.target.select()"
                                                ></b-form-input>
                                            </b-form-group>
                                        </b-col><!--/b-col-->
                                        <b-col sm="6">
                                            <b-form-group
                                                :label="$t('input.phone')+' *'"
                                                label-for="phone"
                                                :invalid-feedback="formErrors.first('phone')"
                                            >
                                                <b-form-input
	                                                pattern="^[0-9-+()]*"
	                                                title="+(XXX) XXX"
                                                    id="phone"
                                                    v-model="formFields.phone"
                                                    type="number"
                                                    :state="((formErrors.has('phone') ? false : null))"
                                                    @focus="$event.target.select()"
                                                ></b-form-input>
                                            </b-form-group>
                                        </b-col><!--/b-col-->
                                    </b-row><!--/b-row-->
	                                <b-row>
		                                <b-col sm="12">
			                                <b-form-group
				                                :label="$t('input.function')+' '"
				                                label-for="function"
				                                :invalid-feedback="formErrors.first('functions')"
			                                >
				                                <b-form-input
					                                id="function"
					                                v-model="formFields.functions"
					                                type="text"
					                                :state="((formErrors.has('functions') ? false : null))"
					                                @focus="$event.target.select()"
				                                ></b-form-input>
			                                </b-form-group>
		                                </b-col><!--/b-col-->
	                                </b-row><!--/b-row-->
                                </b-col><!--/b-col-->
                                <div class="drawer-footer">
                                    <b-button
                                        size="sm"
                                        type="submit"
                                        variant="primary"
                                        :disabled="global.pendingRequests > 0"
                                        v-b-tooltip.hover :title="$t('button.title.save')"
                                    >
                                        <clip-loader style="display: inline" :loading="true" color="#fff"
                                                     size="12px"
                                                     v-if="global.pendingRequests > 0"></clip-loader>
                                        <i class="fa fa-save mr-1"></i>
                                        {{$t('button.save')}}
                                    </b-button>
                                    <b-button variant="warning" class="ml-3"
                                              size="sm" @click="handleOperationClose()"
                                              v-b-tooltip.hover :title="$t('button.title.cancel')">
                                        <i class="fa fa-arrow-left mr-1"></i> {{$t('button.cancel')}}
                                    </b-button>
                                </div>
                            </b-row><!--/b-row-->
                        </form><!--/form-->
                    </a-drawer>
                </div><!--/.contacts-operation-->
				<div class="filter-container">
					<a-drawer
						placement="left"
						:width="'360px'"
						:wrapStyle="{overflow: 'auto',paddingBottom: '108px'}"
						:closable="false"
						@close="filters.visible = !filters.visible"
						:visible="!operation && filters.visible"
						:zIndex="10"
						:title="$t('title.advanceFilters')"
					>
						<form @submit.prevent="loadList(listQueryParams, true)" autocomplete="off">
							<b-row>
								<b-col sm="12">
									<b-form-group
											:label="$t('input.fromAddedDate')"
											label-for="fromDate">
										<b-form-datepicker placeholder="" id="fromDate" v-model="filters.from_date"
										                   class="mb-2"></b-form-datepicker>
									</b-form-group>
								</b-col><!--/b-col-->
								<b-col sm="12">
									<b-form-group
											:label="$t('input.toAddedDate')"
											label-for="toDate">
										<b-form-datepicker placeholder="" id="toDate" v-model="filters.to_date"
										                   class="mb-2"></b-form-datepicker>
									</b-form-group>
								</b-col><!--/b-col-->
							</b-row>
							<div class="drawer-footer">
								<b-button size='sm' variant="info" @click="filters.visible = !filters.visible"
								          class="mr-2" :title="$t('button.title.closePanel')" v-b-tooltip.hover>
									{{$t('button.close')}}
								</b-button>
								<b-button size='sm' variant="warning" @click="handleResetFilterClick" class="mr-2"
								          :title="$t('button.title.resetFilter')" v-b-tooltip.hover>
									{{$t('button.reset')}}
								</b-button>
								<b-button size='sm' variant="primary" button="submit" type="filled"
								          :title="$t('button.title.filterRecords')" v-b-tooltip.hover>
									{{$t('button.apply')}}
								</b-button>
							</div><!-- /.drawer-footer -->
						</form>
					</a-drawer>
				</div><!-- /.filter-container -->
			</div><!-- /.card-body-->
		</div><!-- /.card -->
	</div>
</template>
<script>
    import ListingMixin from '../../util/ListingMixin'
    import Error from '../../util/Error'
    import {mapState} from 'vuex'
    import Datepicker from 'vuejs-datepicker'
    import {request} from '../../util/Request'
    import {itemAdded, itemUpdated, itemDeleted, itemDeleteFails, itemEditFails} from '../../util/Notify'
    import {handleServerError} from '../../util/Utils'
    import Treeselect from '@riophae/vue-treeselect'

    const FORM_STATE = {
        first_name: null,
        last_name: null,
        email: null,
        phone: null,
        mobile: null,
        functions: null,
        _method: 'post',
    };

    const FILTER_STATE = {
        visible: false,
        from_date: null,
        to_date: null,
    };

    const COLUMN_DEFINATION = (self) => [
        {
            label: '#',
            key: 'id',
            sortable: true,
            sortKey: 'id',
        },
        {
            label: self.$t('column.firstName'),
            key: 'first_name',
            sortable: true,
            sortKey: 'first_name',
        },
        {
            label: self.$t('column.lastName'),
            key: 'last_name',
            sortable: true,
            sortKey: 'last_name',
        },
        {
            label: self.$t('column.email'),
            key: 'email',
            sortable: true,
            sortKey: 'email',
        },
        {
            label: self.$t('column.phone'),
            key: 'phone',
            sortable: true,
            sortKey: 'phone',
        },
        {
            label: self.$t('column.mobile'),
            key: 'mobile',
            sortable: true,
            sortKey: 'mobile',
        },
        (self.$global.hasAnyPermission(['contactsupdate', 'contactsdestroy'])
        ? {
            label: self.$t('column.action'),
            class: 'text-right',
            key: 'action',
            width: 150,
        } : {}),
    ];

    export default {
        mixins: [ListingMixin],
        components: {
            Datepicker,
            Treeselect
        },
        data() {
            return {
                operationTitle: 'title.contacts',
                formFields: {...FORM_STATE},
                filters: {...FILTER_STATE},
                listUrl: 'contacts',
                columns: COLUMN_DEFINATION(this),
                dropdowns: {},
                show: true,
            }
        },
        mounted() {
            if (this.$route.query && this.$route.query.operation && this.$route.query.oToken) {
                this.handleEditClick(this.$route.query.oToken)
            }
        },
        methods: {
            setOperation(operation = 'add', operationToken = null) {
                this.operationTitle = (operation === 'add' ? 'title.addContact' : 'title.editContact')
                this.$router.replace({
                    query: Object.assign({},
                        this.$route.query,
                        {
                            ...this.listQueryParams,
                            operation: operation,
                            oToken: operationToken,
                        },
                    ),
                }).then(() => {
                }).catch(() => {
                })
            },
            async handleSubmit() {
                this.formErrors = new Error({})
                try {
                    const response = await request({
                        url: this.formFields.id ? 'contacts/update' : 'contacts/create',
                        method: 'post',
                        data: this.formFields,
                    })

                    if (this.formFields.id) {
                        itemUpdated(this.$notification)
                    } else {
                        itemAdded(this.$notification)
                    }

                    this.handleOperationClose()
                } catch (error) {
                    if (error.request && error.request.status && error.request.status === 422) {
                        this.formErrors = new Error(JSON.parse(error.request.responseText).errors)
                        return false
                    }

                    handleServerError(error, this.$notification)
                }
            },
            async handleDeleteClick(id) {
                try {
                    const response = await request({
                        method: 'post',
                        url: '/contacts/delete',
                        data: {
                            id: id,
                        },
                    })
                    this.loadList(this.listQueryParams)
                    itemDeleted(this.$notification)
                } catch (errors) {
                    itemDeleteFails(this.$notification)
                }
            },
            async handleEditClick(id) {
                try {
                    const response = await request({
                        method: 'get',
                        url: `/contacts/detail/${id}`,
                    })
                    this.operationTitle = this.$t('title.editContact')
                    const {data} = response
                    this.formFields = {...data}
                } catch (e) {
                    itemEditFails(this.$notification)
                    this.formFields = {...FORM_STATE}
                }
            },
            hasListAccess() {
                return this.$global.hasPermission('contactsview')
            },
            getExtraParams() {
                return {
                    filters: {
                        ...this.filters,
                        from_date: ((this.filters.from_date) ? this.$global.dateToUtcDate(this.filters.from_date, 'YYYY-MM-DD', 'YYYY-MM-DD') : ''),
                        to_date: ((this.filters.to_date) ? this.$global.dateToUtcDate(this.filters.to_date, 'YYYY-MM-DD', 'YYYY-MM-DD') : ''),
                    },
                }
            },
            handleResetFilterClick() {
                this.filters = {...FILTER_STATE}
                this.isFilterApplied = 'reset'
                this.loadList(this.listQueryParams)
            },
            afterCloseOperation() {
                this.formFields = {...FORM_STATE}
            },
            handleNameBlur() {
                let names = this.formFields.name.split(' ');
                if(names.length > 0) {
                    if(!this.formFields.first_name) {
                        this.formFields.first_name = names[0];
                    }

                    delete names[0]
                    names = names.join(" ");
                    if(!this.formFields.last_name) {
                        this.formFields.last_name = names;
                    }
                }
            }
        },
        computed: {
            ...mapState([
                'global',
                'settings'
            ]),
        },
    }
</script>


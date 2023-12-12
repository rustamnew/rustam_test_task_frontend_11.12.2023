<script setup lang="ts">
import { ref } from 'vue'

import { DateRange } from '~/ui/date-range';
import { Select } from '~/ui/select';
import { Button } from '~/ui/button';
import { StatusCard } from '~/ui/status-card';
import { Empty } from '~/ui/empty';
import { SpinnerLoader } from '~/ui/spinner-loader';

import { Search } from '@/components/shared/search';
import FixedLeftColumnLayout from '@/views/layout/fixed-left-column.vue';


const years = ref([
    {
        value: '2023',
        title: '2023'
    },
    {
        value: '2022',
        title: '2022'
    },
    {
        value: '2021',
        title: '2021'
    },
])

const props = defineProps({
    orderId: { 
        type: String, 
        required: false 
    },
})

console.log(props)
</script>   

<template>
    <FixedLeftColumnLayout>
        <template v-slot:fixed>
            <div class="controls">

                <Button :size="'large'" :color="'purple-reverse'" class="block mobile" @click="navigateTo('/manager/orders')">Назад</Button>
                
                
                <div class="block">
                    <DateRange @change="changeDateRange($event)"/> 

                    <Search 
                    :searchQuery="searchParams.searchQuery" 
                    :types="types" 
                    :type="searchParams.searchType" 
                    @update:type="changeOrderSearchType($event)"
                    @update:searchQuery="changeOrderSearchQuery($event)"/>


                    <div class="row">
                        <Button :size="'s'" :color="'purple-reverse'" :class="searchParams.searchTags.length > 5 ? 'active' : ''" @click="setSearchTag(['client', 'type_pay', 'in_work', 'status', 'ps_status', 'unload'])">
                            Все
                        </Button>

                        <Button :size="'s'" :color="'purple-reverse'" :class="searchParams.searchTags.includes('client') ? 'active' : ''" @click="setSearchTag('client')">
                            <i class="fa-regular fa-crown"></i>
                        </Button>

                        <Button :size="'s'" :color="'purple-reverse'" :class="searchParams.searchTags.includes('type_pay') ? 'active' : ''" @click="setSearchTag('type_pay')">
                            <i class="fa-regular fa-thumbs-up"></i>
                        </Button>

                        <Button :size="'s'" :color="'purple-reverse'" :class="searchParams.searchTags.includes('in_work') ? 'active' : ''" @click="setSearchTag('in_work')">
                            <i class="fa-regular fa-briefcase"></i>
                        </Button>

                        <Button :size="'s'" :color="'purple-reverse'" :class="searchParams.searchTags.includes('status') ? 'active' : ''"  @click="setSearchTag('status')">
                            <i class="fa-solid fa-cloud-arrow-up"></i>
                        </Button>

                        <Button :size="'s'" :color="'purple-reverse'" :class="searchParams.searchTags.includes('ps_status') ? 'active' : ''"  @click="setSearchTag('ps_status')">
                            <i class="fa-regular fa-money-bill"></i>
                        </Button>

                        <Button :size="'s'" :color="'purple-reverse'" :class="searchParams.searchTags.includes('unload') ? 'active' : ''"  @click="setSearchTag('unload')">
                            <i class="fa-regular fa-truck"></i>
                        </Button>
                    
                    </div>
                    

                </div>

                <div class="block">
                    <Select 
                    :items="years" 
                    :modelValue="searchParams.searchYear" 
                    @update:model-value="changeOrderSearchYear($event)"
                    :name="'name'" 
                    :description="'Выберите год'"
                    :placeholder="'Выберите год'"/>

                    <div class="row">
                        <Button :size="'large'" :color="'purple'" @click="searchOrder()">
                            Показать
                        </Button>

                        <Button :size="'large'" :color="'purple-reverse'" @click="restoreSearchDefault()">
                            Сбросить
                        </Button>
                    </div>
                    
                </div>
            </div>
        </template>

        <template v-slot>
            <div class="status-card-list">
                <SpinnerLoader v-if="isLoading" />

                <Empty v-if="orders.length === 0" class="empty-message">Заказов еще нет</Empty>
                
                <template v-if="orderId">
                    <StatusCard v-for="card in orders" :key="card.id">
                        <div class="inner">
                            <span>№ {{ card.id }}</span>

                            <!--
                            <br>

                            <span v-if="card.date !== '0'">
                                Дата: {{ card.date }}
                            </span>

                            <span v-if="card.psid !== '0'">
                                Номер фотосессии: {{ card.psid }}
                            </span>

                            <span v-if="card.uid !== '0'">
                                Клиент ID: {{ card.uid }}
                            </span>

                            <span v-if="card.phone !== '0'">
                                Телефон: {{ card.phone }}
                            </span>

                            <span v-if="card.email_payer !== '0'">
                                Email: {{ card.email_payer }}
                            </span>

                            <span v-if="card.fi_child !== '0'">
                                Ребенок: {{ card.fi_child }}
                            </span>
                            -->
                        </div>
                    </StatusCard>
                </template>

                <template v-else>
                    <StatusCard v-for="card in orders" :key="card.id">
                        <div class="inner">
                            <span>№ {{ card.id }}</span>

                            <!--
                            <br>

                            <span v-if="card.date !== '0'">
                                Дата: {{ card.date }}
                            </span>

                            <span v-if="card.psid !== '0'">
                                Номер фотосессии: {{ card.psid }}
                            </span>

                            <span v-if="card.uid !== '0'">
                                Клиент ID: {{ card.uid }}
                            </span>

                            <span v-if="card.phone !== '0'">
                                Телефон: {{ card.phone }}
                            </span>

                            <span v-if="card.email_payer !== '0'">
                                Email: {{ card.email_payer }}
                            </span>

                            <span v-if="card.fi_child !== '0'">
                                Ребенок: {{ card.fi_child }}
                            </span>
                            -->
                        </div>
                    </StatusCard>
                </template>
            </div>
        </template>
    </FixedLeftColumnLayout>
</template>

<script lang="ts">
    export default {

        data() {
            const route = useRoute()

            const types: any = {
                order_number: {
                    name: 'id',
                    placeholder: 'Номер заказа',
                    title: 'Номер заказа'
                },
                psid: {
                    name: 'psid',
                    placeholder: 'Номер фотосессии',
                    title: 'Номер фотосессии'
                },
                client_id: {
                    name: 'uid',
                    placeholder: 'Клиент ID',
                    title: 'Клиент ID'
                },
                phone: {
                    name: 'phone',
                    placeholder: 'Телефон',
                    title: 'Телефон'
                },
                email: {
                    name: 'email_payer',
                    placeholder: 'Email',
                    title: 'Email'
                },
                fi_child: {
                    name: 'fi_child',
                    placeholder: 'Ребенок',
                    title: 'Ребенок'
                }
            }
            
            const orders: any = []

            const searchTags: any = []

            return {
                isLoading: false,

                default: {
                    searchParams: {
                        searchQuery: '',
                        searchType: 'order_number',
                        searchYear: '',
                        searchTags,
                        searchItems: []
                    },
                    orders: [],
                    
                },

                searchParams: {
                    searchQuery: '',
                    searchType: 'order_number',
                    searchYear: '',
                    searchTags,
                    searchItems: []
                },

                orders,
                types,
                route
            }
        },

        async mounted() {
            await this.fetchOrders()

            if (this.orderId) {
                console.log(1)
                this.getOrderById(this.orderId)
                return
            }
            if (this.checkUrlParams()) {
                this.searchOrder()
            }
        },

        methods: {
            async fetchOrders() {
                this.isLoading = true
                const orders = await useFetch('/method/orders.getTest')

                const jsonStr: any = orders.data.value
                const jsonObj: any = JSON.parse(jsonStr)

                const ordersArray: any = jsonObj.response.data.orders

                this.isLoading = false

                this.default.orders = ordersArray
                this.orders = ordersArray
            },

            checkUrlParams() {
                const query: any = this.$route.query
                let needSearch = false;

                if (query.search_type) {
                    this.searchParams.searchType = query.search_type
                    needSearch = true
                }
                if (query.search_value) {
                    this.searchParams.searchQuery = query.search_value
                    needSearch = true
                }

                return needSearch
            },

            setUrlParams() {
                this.$router.push({query: {
                    search_type: this.searchParams.searchType,
                    search_value: this.searchParams.searchQuery,
                    year: this.searchParams.searchYear
                }})
            },

            clearUrlParams() {
                this.$router.push({query: {}})
            },

            changeOrderSearchType(value: string) {
                this.searchParams.searchType = value
            },

            changeOrderSearchQuery(value: string) {
                this.searchParams.searchQuery = value
            },

            changeOrderSearchYear(value: string) {
                this.searchParams.searchYear = value
            },       

            searchOrder() {
                navigateTo('/manager/orders')

                this.$router.push({query: {
                    search_type: this.searchParams.searchType,
                    search_value: this.searchParams.searchQuery,
                    year: this.searchParams.searchYear
                }})

                this.orders = this.default.orders

                const type: any = this.searchParams.searchType
                const query = this.searchParams.searchQuery
                const year = this.searchParams.searchYear

                const typeSearchName = this.types[type].name

                this.orders = this.orders.filter((item: any) => {
                    let returnVal = false
                    const itemYear = item.date.slice(6, 10)
                    if (itemYear.includes(year)) {
                        returnVal = true

                        if (item[typeSearchName].includes(query)) {
                            returnVal = true
                        } else {
                            returnVal = false
                        }
                    }
                    
                    return returnVal
                })

                
                this.searchParams.searchItems = this.orders
                this.setUrlParams()
                this.filterByTags()
            },

            restoreSearchDefault() {
                this.orders = this.default.orders
                this.searchParams = this.default.searchParams
                this.clearUrlParams()
            },

            filterByTags() {
                if (!this.searchParams.searchTags) {
                    return
                }
                if (!this.searchParams.searchItems.length) {
                    this.searchParams.searchItems = this.orders
                }
                this.orders = this.searchParams.searchItems.filter((
                    (data: any) => this.searchParams.searchTags.every(
                        (tag: any) => data[tag] !== '0'
                    )
                ))
            },

            setSearchTag(tag: string | string[]) {
                if (typeof tag === 'string') {
                    if(!this.searchParams.searchTags.includes(tag)) {
                        this.searchParams.searchTags.push(tag)
                    } else {
                        const index: number = this.searchParams.searchTags.indexOf(tag)
        
                        if (index > -1) {
                            this.searchParams.searchTags.splice(index, 1);
                        }

                    }
                } else {
                    if (this.searchParams.searchTags.length > 5) {
                        this.searchParams.searchTags = []

                        return
                    }
                    tag.forEach((item) => {
                        if(!this.searchParams.searchTags.includes(item)) {
                            this.searchParams.searchTags.push(item)
                        }
                    })
                    
                }
                
                this.filterByTags()
            },

            changeDateRange(event: any) {
                this.checkDateRange(event)
            },

            checkDateRange(dateRange: any) {
                const dateRanged = this.default.orders.filter((item: any) => {

                    const day = item.date.slice(0, 2)
                    const month = item.date.slice(3, 5)
                    const year = item.date.slice(6, 10)

                    const startDate = new Date(dateRange.date_start * 1000).getTime()
                    
                    const itemDate = new Date(year, month - 1, day).getTime()

                    const finishDate = new Date(dateRange.date_finish * 1000).getTime()

                    let returnValue = false

                    if (startDate < itemDate && itemDate < finishDate) {
                        returnValue = true
                    }

                    return returnValue
                }) 

                this.orders = dateRanged
            },

            getOrderById(id: string) {
                this.orders = this.orders.filter( (item: any) => {
                    return item.id === id
                })
            }
        }
    }
</script>


<style lang="scss" >
    .block {
        padding: 16px;
        margin-bottom: 15px;  

        &:last-child {
            margin-bottom: 0;
        }

        .inner {
            display: flex;
            flex-direction: column;
        }

        .row {
            & > button {
                margin-right: 5px;
                margin-bottom: 5px;

                &:last-child {
                    margin-right: 0;
                }
            }
        }
    }

    .controls {
        .block {
            & > * {
                margin-bottom: 15px;

                &:last-child {
                    margin-bottom: 0;
                }
            }
        }
    } 


    .button.active[data-color=purple-reverse] {
        border-color: transparent;
        color: var(--color_btn);
        background-color: var(--bg_hover_btn_purple_reverse);
    }

    .mobile {
        display: none;
    }

    @include active-by("lg") {    
        .mobile {
            display: block;
            width: 100%;
        }
    }
    
</style>
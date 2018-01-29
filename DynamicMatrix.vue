<template>
    <div>
        <v-data-table
                :items="rows"
                :headers="headers"
                hide-actions
        >
            <template slot="items" slot-scope="props" >
                <td>{{ props.item.name }}</td>
                <td
                        v-for="(column, index) in columns"
                        class="text-xs-center"
                >

                    <v-btn
                            v-if="matrix[props.item.key][column.key].type === 'boolean'"
                            :color="matrix[props.item.key][column.key].value ? 'primary' : ''"
                            :loading="matrix[props.item.key][column.key].loading"
                            @click="updatePivot(column.key, props.item.key, matrix[props.item.key][column.key])"
                            flat
                            icon

                    >
                        <v-icon
                                v-if="matrix[props.item.key][column.key].value"
                        >
                            radio_button_checked
                        </v-icon>
                        <v-icon v-else>
                            radio_button_unchecked
                        </v-icon>
                    </v-btn>
                </td>
            </template>
        </v-data-table>
    </div>
</template>

<script lang="ts">
    import Vue, { ComponentOptions } from 'vue';

    import axios, {AxiosStatic} from 'axios';

    import _ from 'lodash'

    interface Column{
        key: string;
        name: string;
    }

    interface Row{
        key: string;
        name: string;
    }

    interface Element {
        type?: string;
        value: any;
        loading: boolean;
    }

    interface Header {
        text: string;
        align?: string;
        sortable: boolean;
        value: string;
    }

    interface DynamicMatrixComponent extends Vue{
        axios: AxiosStatic;
        axiosChoosen: AxiosStatic;
        columns: Column[];
        defaultType: string;
        headers: Array<Header>;
        locale: string;
        manageBooleanChange(el : Element, value : boolean, column : string, row: string);
        matrix: any;
        rows: Row[];
        updatePivot(column : number|string, row: number|string, value : any);
        urlPrefix: string;
    }

    export {
        Column,
        Row
    }

    export default {
        created(){
            this.headers.push({
                text: '',
                sortable: false,
                value: '',
            });
            _.each(this.columns, (el) => {
                this.headers.push({
                    text: el.name,
                    sortable: false,
                    value: el.name,
                    align: 'center'
                });
            })
            this.init();
        },
        props: {
            axios:{
                type: Object
            },
            columns: {
                type: Array,
                required: true
            },
            defaultType: {
                type: String,
                default: 'boolean'
            },
            locale: {
                type: String
            },
            rows: {
                type: Array,
                required: true
            },
            urlPrefix: {
                type: String,
                required: true
            }
        },
        data(){
            return {
                axiosChoosen: this.axios || axios,
                headers: [],
                matrix: {}
            }
        },
        methods:{
            init(){
                let initMatrix = {};
                for(let index1 in this.rows){
                    let element1 = this.rows[index1];
                    let realIndex1 = element1.key;
                    let initMatrixInside  = {};
                    for(let index2 in this.columns){
                        let element2 = this.columns[index2];
                        let realIndex3 = element2.key;
                        initMatrixInside[realIndex3] = {
                            type: this.defaultType,
                            loading: true,
                            value: null
                        };
                    }
                    initMatrix[realIndex1] = initMatrixInside;
                };
                this.matrix = initMatrix;
                this.axiosChoosen.get(`${this.urlPrefix}/init`, {params:{locale: this.locale}}).then((response) => {
                    let matrix = response.data;
                    _.each(matrix, (el1) => {
                        _.each(el1, (el2) => {
                            el2.loading = false;
                            el2.type = el2.type || this.defaultType;
                        })
                    })
                    this.matrix = matrix;
                })
            },
            manageBooleanChange(el : Element, value : boolean, column : string, row: string){
                el.loading = true;
                this.axiosChoosen.put(`${this.urlPrefix}`, {
                    column: column,
                    row: row,
                    value: !value,
                    type: el.type
                }).then(() => {
                    el.value = !value
                    this.$nextTick(() => {
                        this.$forceUpdate();
                    });
                }).catch((err) => {
                    console.trace();
                    console.log(err);
                }).then(() => {
                    el.loading = false;
                });
            },
            updatePivot(column : string, row: string, value : any){
                let el : Element = this.matrix[row][column];
                let type = el.type || 'boolean';

                switch (type){
                    case 'boolean':
                        this.manageBooleanChange(el, value.value as boolean, column, row);
                        break;
                    default:
                        throw `DynamicMatrix::updatePivot type:${type} is unknown`;
                }
            }
        }
    } as ComponentOptions<DynamicMatrixComponent>;
</script>


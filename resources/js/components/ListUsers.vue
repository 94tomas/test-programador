<template>
    <v-container>
        <v-row align="center">
            <v-col cols="8">
                <v-select
                    :items="items"
                    item-text="text"
                    item-value="value"
                    label="Filtro por estado"
                    outlined
                    v-model="filter"
                    @change="filterByStatus"
                ></v-select>
            </v-col>
        </v-row>
        <v-data-table
            :headers="headers"
            :items="data"
            class="elevation-1"
            hide-default-footer
        >
            <template v-slot:item.status="{ item }">
                <v-chip dark :color="(item.status) ? 'success' : 'error'">{{ (item.status) ? 'Activo' : 'inactivo' }}</v-chip>
            </template>
            <template v-slot:item.actions=" item ">
                <v-btn small @click="ghangeState(item)">Cambiar estado</v-btn>
            </template>
        </v-data-table>
        <div class="text-center">
            <v-pagination
                v-model="page"
                :length="totalPage"
                :total-visible="7"
                dark
                @input="changePage"
                class="py-5"
            ></v-pagination>
        </div>
    </v-container>
</template>
<script>
import axios from 'axios'
export default {
    data() {
        return {
            headers: [
                { text: 'Nombre', align: 'start', value: 'name', },
                { text: 'Email', value: 'email' },
                { text: 'Estado', value: 'status', sortable: false },
                { text: '', value: 'actions', align: 'end', sortable: false }
            ],
            data: [],
            page: 1,
            totalPage: null,
            items: [
                { text: 'Todos', value: 'all' },
                { text: 'Activos', value: 'active' },
                { text: 'Inactivos', value: 'inactive' }
            ],
            filter: 'all'
        }
    },
    created() {
        this.listUsers()
    },
    methods: {
        changePage(event) {
            this.page = event;
            this.listUsers();
        },
        filterByStatus(event) {
            this.filter = event;
            this.page = 1;
            this.listUsers();
            console.log(this.filter);
        },
        async listUsers() {
            await axios.get(`/list?page=${this.page}&status=${this.filter}`)
            .then(response => {
                console.log(response);
                this.data = response.data.data
                this.totalPage = response.data.last_page;
            })
            .catch(error => {
                console.log(error);
            })
        },
        // 
        ghangeState(item) {
            console.log(item.item);
            const formData = new FormData();
            formData.append('status', !item.item.status)
            formData.append('_method', 'PUT');
            console.log(formData);

            axios.post(`/list/${item.item.id}`, formData)
            .then(response => {
                console.log(response);
                if (response.data.msg == 'ok') {
                    this.listUsers()
                }
            })
            .catch(error => {
                console.log(error);
            })
        }
    },
}
</script>
<template>
    <div class="user-admin">
        <b-form class="mb-3">
            <input id="user-id" type="hidden" v-model="user.id">
            <b-row>
                <b-col md="6" sm="12">
                    <b-form-group label="Nome:" label-for="user-name">
                        <b-form-input id="user-name" type="text" v-model="user.name" 
                            required :readonly="mode === 'remove'"
                            placeholder="Informe o nome do usuário"/>
                    </b-form-group>
                </b-col>
                <b-col md="6" sm="12">
                    <b-form-group label="E-mail:" label-for="user-email">
                        <b-form-input id="user-email" type="email" v-model="user.email" 
                            required :readonly="mode === 'remove'"
                            placeholder="Informe o e-mail do usuário"/>
                    </b-form-group>
                </b-col>
            </b-row>

            <b-form-checkbox v-if="mode === 'save'" id="user-admin" v-model="user.admin" class="mt-3 mb-3">
                Administrador
            </b-form-checkbox>
            
            <b-row v-if="mode === 'save'">
                <b-col md="6" sm="12">
                    <b-form-group label="Senha:" label-for="user-password">
                        <b-form-input id="user-password" type="password" v-model="user.password" 
                            required placeholder="Informe a senha do usuário"/>
                    </b-form-group>
                </b-col>
                <b-col md="6" sm="12">
                    <b-form-group label="Confirme a senha:" label-for="user-confirm-password">
                        <b-form-input id="user-confirm-password" type="password" v-model="user.confirmPassword" 
                            required placeholder="Confirme a senha do usuário"/>
                    </b-form-group>
                </b-col>
            </b-row>

            <b-row>
                <b-col cols="12">
                    <b-button variant="primary" v-if="mode === 'save'" @click="save">
                        Salvar
                    </b-button>
                    <b-button variant="danger" v-if="mode === 'remove'" @click="remove">
                        Excluir
                    </b-button>
                    <b-button  @click="reset" class="ml-2">
                        Cancelar
                    </b-button>
                </b-col>
            </b-row>
        </b-form>

        <b-table hover responsive striped :items="users" :fields="fields">
            <template slot="actions" slot-scope="data">
                <b-button variant="warning" @click="loadUser(data.item)" class="mr-2">
                    <i class="fa fa-pencil"></i>
                </b-button>
                <b-button variant="danger" @click="loadUser(data.item, 'remove')">
                    <i class="fa fa-trash"></i>
                </b-button>
            </template>
        </b-table>
    </div>
</template>

<script>
import { baseApiUrl, showError } from '@/global'
import axios from 'axios'

export default {
    name: 'UserAdmin',
    data() {
        return {
            mode: 'save',
            user: {},
            users: [],
            fields: [
                { key: 'id', label: 'Códigos', sortable: true },
                { key: 'name', label: 'Nome', sortable: true },
                { key: 'email', label: 'E-mail' },
                { key: 'admin', label: 'Administrador', sortable: true , 
                    formatter: value => value ? 'Sim': 'Não' 
                },
                { key: 'actions', label: 'Ações' }
            ]
        }
    },
    methods: {
        loadUsers() {
            const url = `${baseApiUrl}/users`
            axios.get(url)
                .then(res => { 
                    this.users = res.data
                })
        },
        loadUser(user, mode = 'save') {
            this.mode = mode
            this.user = { ...user }
            window.scrollTo({
                top: 0,
                left: 0,
                behavior: "smooth"
            });
        },
        reset() {
            this.mode = 'save'
            this.user = {}
            this.loadUsers()
        },
        save() {
            const method = this.user.id ? 'put' : 'post'
            const id = this.user.id ? `${this.user.id}` : ''

            axios[method](`${baseApiUrl}/users/${id}`, this.user)
                .then(() => {
                    this.$toasted.global.defaultSuccess()
                    this.reset()
                })
                .catch(showError)
        },
        remove() {
            const id = this.user.id 

            axios.delete(`${baseApiUrl}/users/${id}`)
                .then(() => {
                    this.$toasted.global.defaultSuccess()
                    this.reset()
                })
                .catch(showError)
        },
    },
    mounted() {
        this.loadUsers()
    }
}
</script>

<style>

</style>
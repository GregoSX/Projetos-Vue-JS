<template>
    <div class="article-admin">
        <b-form class="mb-3">
            <input id="article-id" type="hidden" v-model="article.id">
            <b-row>
                <b-col cols="12">
                    <b-form-group label="Nome:" label-for="article-name">
                        <b-form-input id="article-name" type="text" v-model="article.name" 
                            required :readonly="mode === 'remove'"
                            placeholder="Informe o nome do artigo"/>
                    </b-form-group>
                </b-col>
            </b-row>

            <b-row>
                <b-col cols="12">
                    <b-form-group label="Descrição:" label-for="article-description">
                        <b-form-input id="article-description" type="text" v-model="article.description" 
                            required :readonly="mode === 'remove'"
                            placeholder="Informe a descrição do artigo"/>
                    </b-form-group>
                </b-col>
            </b-row>

            <b-row>
                <b-col cols="12">
                    <b-form-group v-if="mode === 'save'" label="Imagem (URL):" label-for="article-imageUrl">
                        <b-form-input id="article-imageUrl" type="text" v-model="article.imageUrl" 
                            required :readonly="mode === 'remove'"
                            placeholder="Informe a URL da imagem do artigo"/>
                    </b-form-group>
                </b-col>
            </b-row>

            <b-row >
                <b-col cols="12">
                    <b-form-group v-if="mode === 'save'" label="Categoria:" label-for="article-categoryId">
                        <b-form-select
                            id="article-categoryId" :options="categories"
                            v-model="article.categoryId"
                        />
                    </b-form-group>
                </b-col>
            </b-row>

            <b-row >
                <b-col cols="12">
                    <b-form-group v-if="mode === 'save'" label="Autor:" label-for="article-userId">
                        <b-form-select
                            id="article-userId" :options="users"
                            v-model="article.userId"
                        />
                    </b-form-group>
                </b-col>
            </b-row>

            <b-row>
                <b-col cols="12">
                    <b-form-group v-if="mode === 'save'" label="Conteúdo:" label-for="article-content">
                        <VueEditor v-model="article.content"/>
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

        <b-table hover responsive striped :items="articles" :fields="fields">
            <template slot="actions" slot-scope="data">
                <b-button variant="warning" @click="loadArticle(data.item)" class="mr-2">
                    <i class="fa fa-pencil"></i>
                </b-button>
                <b-button variant="danger" @click="loadArticle(data.item, 'remove')">
                    <i class="fa fa-trash"></i>
                </b-button>
            </template>
        </b-table>

        <b-pagination size="md" v-model="page" :total-row="count" :per-page="limit"/>
    </div>
</template>

<script>
import { VueEditor } from 'vue2-editor'
import { baseApiUrl, showError } from '@/global'
import axios from 'axios'

export default {
    name: 'ArticleAdmin',
    components: { VueEditor },
    data() {
        return {
            mode: 'save',
            article: {},
            articles: [],
            categories: [],
            users: [],
            page: 1,
            limit: 0,
            count: 0,
            fields: [
                { key: 'id', label: 'Códigos', sortable: true },
                { key: 'name', label: 'Nome', sortable: true },
                { key: 'description', label: 'Descrição' },
                { key: 'actions', label: 'Ações' }
            ]
        }
    },
    methods: {
        loadArticles() {
            const url = `${baseApiUrl}/articles?page=${this.page}`
            axios.get(url)
                .then(res => { 
                    this.articles = res.data.data
                    this.count = res.data.count
                    this.limit = res.data.limit
                })
        },
        loadArticle(article, mode = 'save') {
            this.mode = mode
            const url = `${baseApiUrl}/articles/${article.id}`
            axios.get(url)
                .then(res => { 
                    this.article = res.data
                })
            window.scrollTo({
                top: 0,
                left: 0,
                behavior: "smooth"
            });
        },
        loadCategories() {
            const url = `${baseApiUrl}/categories`
            axios.get(url)
                .then(res => { 
                   this.categories = res.data.map(category => {
                        return { value: category.id, text: category.path }
                   })
                })
        },
        loadUsers() {
            const url = `${baseApiUrl}/users`
            axios.get(url)
                .then(res => { 
                   this.users = res.data.map(user => {
                        return { value: user.id, text: `${user.name} - ${user.email}`}
                   })
                })
        },  
        reset() {
            this.mode = 'save'
            this.article = {}
            this.loadArticles()
        },
        save() {
            const method = this.article.id ? 'put' : 'post'
            const id = this.article.id ? `${this.article.id}` : ''

            axios[method](`${baseApiUrl}/articles/${id}`, this.article)
                .then(() => {
                    this.$toasted.global.defaultSuccess()
                    this.reset()
                })
                .catch(showError)
        },
        remove() {
            const id = this.article.id 

            axios.delete(`${baseApiUrl}/articles/${id}`)
                .then(() => {
                    this.$toasted.global.defaultSuccess()
                    this.reset()
                })
                .catch(showError)
        },
    },
    watch: {
        page() {
            this.loadArticles()
        }
    },
    mounted() {
        this.loadArticles()
        this.loadCategories()
        this.loadUsers()
    }
}
</script>

<style>

</style>
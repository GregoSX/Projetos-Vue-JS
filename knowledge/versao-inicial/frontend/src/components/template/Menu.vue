<template>
    <aside class="menu" v-if="isMenuVisible">
        <div class="menu-filter">
            <i class="fa fa-search fa-lg"></i>
            <input type="text" placeholder="Buscar" v-model="treeFilter" class="filter-field">
        </div>
        <Tree :data="treeData" :options="treeOptions" 
            :filter="treeFilter" ref="tree"/>
    </aside>
</template>

<script>
import { mapState } from 'vuex';
import Tree from 'liquor-tree'
import { baseApiUrl } from '../../global';
import axios from 'axios';

export default {
    name: 'Menu',
    components: { Tree }, 
    computed: mapState(['isMenuVisible']),
    data() {
        return {
            treeFilter: '',
            treeData: this.getTreeData(),
            treeOptions: {
                propertyNames: { 'text': 'name' },
                filter: { emptyText: 'Categoria não encontrada' }
            }
        }
    },
    methods: {
        getTreeData() {
            const url = `${baseApiUrl}/categories/tree`
            return axios.get(url).then(res => res.data)
        },
        onNodeSelected(node) {
            this.$router.push({
                name: 'articlesByCategory',
                params: { id: node.id }
            })
        }
    },
    mounted() {
        this.$refs.tree.$on('node:selected', this.onNodeSelected)
    }
}
</script>

<style>
    .menu {
        grid-area: menu;
        background: linear-gradient(to right, #232526, #414345);

        display: flex;
        flex-direction: column;
        flex-wrap: wrap;
    }

    .menu a, .menu a:hover {
        color: #fff;
        text-decoration: none;
    }

    .menu .tree-node.selected > .tree-content,
    .menu .tree-node .tree-content:hover {
        background-color: rgba(255, 255, 255, 0.2);
    }

    .tree-arrow.has-child {
        filter: brightness(2);
    }

    .menu .menu-filter {
        display: flex;
        justify-content: center;
        align-items: center;
        margin: 20px;
        padding-bottom: 20px;
        border-bottom: 1px solid #AAA;
    }

    .menu .menu-filter i {
        color: #aaa;
        margin-right: 10px;
    }

    .filter-field {
        color: #ccc;
        font-size: 1.3rem;
        border: none;
        outline: none;
        width: 100%;
        background: transparent;
    }

    .tree-filter-empty {
        color: #ccc;
        font-size: 1rem;
        display: flex;
        justify-content: center;
    }
</style>
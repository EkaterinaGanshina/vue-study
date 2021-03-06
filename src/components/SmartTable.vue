<template>
    <div class="smart-table">
        <div v-if="isLoadingShown" class="alert alert-warning">Загрузка...</div>
        <template v-else>
            <div class="table-header">
                <div class="str-count">{{ totalItemsStr }}</div>
                <TableSearch v-model="searchQuery"/>
            </div>

            <table class="table table-striped users">
                <thead>
                <tr>
                    <slot name="table-header">
                        <th>ID</th>
                        <th>Имя</th>
                        <th>Фамилия</th>
                        <th>День рождения</th>
                        <th>E-mail</th>
                        <th>Телефон</th>
                        <th>Компания</th>
                        <th>Действия</th>
                    </slot>
                </tr>
                </thead>
                <tbody>
                <tr v-if="isNotFoundShown">
                    <td colspan="8">Ничего не найдено</td>
                </tr>
                <tr v-else v-for="user in list" :key="user.id" class="user">
                    <slot name="table-row" v-bind="user">
                        <td>{{ user.id }}</td>
                        <td>{{ user.firstName }}</td>
                        <td>{{ user.lastName }}</td>
                        <td>{{ user.birthday }}</td>
                        <td>{{ user.email }}</td>
                        <td>{{ user.phone }}</td>
                        <td>{{ user.company }}</td>
                        <td>
                            <router-link :to="`/view/${user.id}`" tag="button"
                                         class="btn btn-default btn-sm"
                                         title="Полный профиль">
                                <span class="glyphicon glyphicon-list-alt"></span>
                            </router-link>
                            <router-link :to="`/edit/${user.id}`" tag="button"
                                         class="btn btn-default btn-sm"
                                         title="Редактировать">
                                <span class="glyphicon glyphicon-pencil"></span>
                            </router-link>
                        </td>
                    </slot>
                </tr>
                </tbody>
            </table>

            <Pagination :pagesCount="pagesCount" :itemsPerPage="itemsPerPage" v-model="currentPage"/>
            <ItemsPerPageSelect v-model="itemsPerPage"/>
        </template>
    </div>
</template>

<script>
import axios from '@/axios.js'
import proschet from 'proschet'

export default {
  name: 'smart-table',
  components: {
    TableSearch: () => import('@/components/TableSearch.vue'),
    Pagination: () => import('@/components/Pagination.vue'),
    ItemsPerPageSelect: () => import('@/components/ItemsPerPageSelect.vue')
  },
  props: {
    url: {
      type: String,
      required: true
    }
  },
  data: function() {
    return {
      list: [],
      total: 0,
      searchTotal: 0,
      userDeclensions: ['пользователь', 'пользователя', 'пользователей'],
      searchQuery: '',
      itemsPerPage: 10,
      currentPage: 1
    }
  },
  computed: {
    totalItemsStr() {
      const getUsers = proschet(this.userDeclensions)
      const found = this.searchQuery
        ? `, найдено ${this.searchTotal} ${getUsers(this.searchTotal)}`
        : ''
      return `Всего ${this.total} ${getUsers(this.total)}${found}`
    },

    // показывать ли плашку "Загрузка"
    isLoadingShown() {
      return !this.searchQuery && !this.list.length
    },

    // показывать ли строку таблицы "Ничего не найдено"
    isNotFoundShown() {
      return this.searchQuery && this.searchTotal === 0
    },

    itemsCount() {
      return this.searchQuery ? this.searchTotal : this.total
    },

    pagesCount() {
      return Math.ceil(this.itemsCount / this.itemsPerPage)
    },

    requestConfig() {
      let config = {
        params: {
          _page: this.currentPage,
          _limit: this.itemsPerPage
        }
      }

      if (this.searchQuery) {
        config.params.q = this.searchQuery
      }

      return config
    }
  },
  watch: {
    currentPage: 'loadItems', // переключение страницы (пагинация)
    itemsPerPage: 'refreshTable', // изменение количества эл-тов в таблице
    searchQuery: 'refreshTable' // изменение поискового запроса (поиск)
  },
  mounted() {
    this.loadItems()
  },
  methods: {
    loadItems() {
      axios
        .get(this.url, this.requestConfig)
        .then(response => {
          const total = Number(response.headers['x-total-count'])

          this.list = response.data

          if (this.searchQuery) {
            this.searchTotal = total
          } else {
            this.total = total
          }
        })
        .catch(error => {
          console.error(error)
        })
    },

    refreshTable() {
      this.currentPage = 1
      this.loadItems()
    }
  }
}
</script>

<style scoped>
.table-header {
  margin: 20px 0;
  padding: 0 5px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.table-header .str-count {
  font-weight: 700;
  font-size: 14px;
}
</style>

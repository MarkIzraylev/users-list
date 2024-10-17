<script setup>
  'use strict'
  import { computed, ref, watch } from 'vue'

  import InputField from './components/InputField.vue'
  import MediaButton from './components/MediaButton.vue'
  import YesNoModal from './components/YesNoModal.vue'
  import DataTable from './components/DataTable.vue'

  const usersEndpoint = 'https://5ebbb8e5f2cfeb001697d05c.mockapi.io/users'
  const messages = {
    notSorted: 'Сортировка по данному полю отсутствует.',
    sortedInAscendingOrder: 'Строки отсортированы по возрастанию значения этого поля.',
    sortedInDescendingOrder: 'Строки отсортированы по убыванию значения этого поля.',
    userLoadError: 'Сервер выдал ошибку в ответ на запрос списка пользователей.',
    usersNotFound: 'Пользователи, соответствующие фильтру, не найдены. Попробуйте изменить поисковый запрос.'
  }
  const columns = [
    { name: 'username', title: 'Имя пользователя' },
    { name: 'email', title: 'E-mail' },
    { name: 'registration_date', title: 'Дата регистрации' },
    { name: 'rating', title: 'Рейтинг' },
  ]

  const showRemoveUserModal = ref(false)
  const showResetFiltersButton = computed(() => {
    return searchText.value != ''||
      sorting.value.registrationDate != 0 ||
      sorting.value.rating != 0
  })
  const infoMessage = ref('')
  const searchText = ref('')
  
  // массивы пользователей
  const fetchedUsers = ref() // получает пользователей по API
  const filteredUsers = ref() // строится на основе fetchedUsers в соответствии с фильтрами
  const paginatedUsers = ref() // строится на основе filteredUsers в соответствии с currentPage и usersPerPage
  // выбранный пользователь (записывается при клике на крестик)
  const selectedUser = ref()
  
  const currentPage = ref(0)
  const amountOfPages = ref(0)
  const usersPerPage = 5 // макс. число пользователей на странице

  const sorting = ref({
    'registrationDate': 0,
    'rating': 0
  })

  async function loadUsers() {
    try {
      const response = await fetch(usersEndpoint)
      if (!response.ok) {
        throw new Error(messages.userLoadError)
      }
      fetchedUsers.value = await response.json()
      filteredUsers.value = fetchedUsers.value
      resetPages()
      sortUsers()
    } catch (error) {
      infoMessage.value = messages.userLoadError
      console.error(messages.userLoadError, error)
    }
  }

  function filterUsers(newSearchText) {
    filteredUsers.value = fetchedUsers.value.filter(user => {
      const [usernameLower, emailLower, newSearchTextLower] = arrToLower([user.username, user.email, newSearchText])
      return anyHasElement([usernameLower, emailLower], newSearchTextLower)
    })
  }

  function sortUsers() {
    filteredUsers.value.sort((a, b) => {
      const aRegTime = truncateDate(a.registration_date).getTime()
      const bRegTime = truncateDate(b.registration_date).getTime()

      const sortByRegDate = sortByParameter(aRegTime, bRegTime, sorting.value.registrationDate)
      const sortByRating = sortByParameter(a.rating, b.rating, sorting.value.rating)

      return sortByRegDate || sortByRating
    })
  }

  function removeUser() {
    fetchedUsers.value = fetchedUsers.value.filter(user => user.id != selectedUser.value.id)
    filterUsers(searchText.value)
    recalculateAmountOfPages()
    sortUsers()
    closeRemoveUserModal()
  }

  function clearFilters() {
    searchText.value = ''
    filteredUsers.value = fetchedUsers.value
    for (const sortingParam of Object.keys(sorting.value)) {
      sorting.value[sortingParam] = 0
    }
  }

  function recalculateAmountOfPages() {
    amountOfPages.value = Math.ceil(filteredUsers.value.length / usersPerPage)
    if (currentPage.value > amountOfPages.value) {
      currentPage.value = amountOfPages.value
    }
  }

  function changeSorting(parameter) {
    if (!filteredUsers.value) return false
    sorting.value[parameter] = sorting.value[parameter] === -1 ? 1 : -1;
    filterUsers(searchText.value)
    sortUsers()
  }

  function resetPages() {
    recalculateAmountOfPages()
    currentPage.value = amountOfPages.value > 0 ? 1 : 0;
  }

  function thereIsAnotherPage() {
    if (!filteredUsers.value) return false
    return currentPage.value * usersPerPage < filteredUsers.value.length
  }

  function thereIsPageBefore() {
    return currentPage.value > 1
  }

  function nextPage() {
    if (thereIsAnotherPage()) {
      currentPage.value++
    }
  }

  function previousPage() {
    if (thereIsPageBefore()) {
      currentPage.value--
    }
  }

  function closeRemoveUserModal() {
    showRemoveUserModal.value = false
  }

  function selectUser(chosenUser) {
    selectedUser.value = chosenUser
  }

  function openRemoveUserModal(user) {
    selectUser(user)
    showRemoveUserModal.value = true
  }

  function getSortingTitle(parameter) {
    switch (sorting.value[parameter]) {
      case 0:
        return messages.notSorted
      case 1:
        return messages.sortedInAscendingOrder
      case -1:
        return messages.sortedInDescendingOrder
    }
  }

  function arrToLower(arr) {
    return arr.map(el => el.toLowerCase())
  }

  function anyHasElement(arr, searchValue) {
    return arr.some(source => source.includes(searchValue))
  }

  // приводит дату к строке формата дд.мм.гг
  function formatDate(date) {
    return `${String(date.getDate()).padStart(2, '0')}.${String(date.getMonth() + 1).padStart(2, '0')}.${String(date.getFullYear()).slice(-2)}`;
  }

  // оставляет от даты и времени только дату
  function truncateDate(stringDate) {
    return new Date(new Date(stringDate).toLocaleDateString())
  }

  function sortByParameter(a, b, sortingValue) {
    if (sortingValue !== 0 && a !== b) {
      return sortingValue * (a - b)
    }
  }

  // оставляет в paginatedUsers только тех, кто должен отображаться на текущей странице
  watch([filteredUsers, currentPage], ([newFilteredUsers, newCurrentPage]) => {
    const lastExludingIndex = usersPerPage * newCurrentPage
    paginatedUsers.value = newFilteredUsers.slice(lastExludingIndex - usersPerPage, lastExludingIndex)
    if (currentPage.value == 0) {
      infoMessage.value = messages.usersNotFound
    } else {
      infoMessage.value = ''
    }
  })

  watch(searchText, newSearchText => {
    filterUsers(newSearchText)
    resetPages()
    sortUsers()
  })

  watch(filteredUsers, () => {
    sortUsers()
  })

  loadUsers()
</script>
<template>
  <yes-no-modal v-if="showRemoveUserModal" :actionOnYes="removeUser" :closeModal="closeRemoveUserModal">
    Вы уверены, что хотите удалить пользователя?
  </yes-no-modal>
  
  <section class="main">
    <h1 class="main__header">Список пользователей</h1>
    <div class="main__search-panel">
      <InputField placeholder="Поиск по имени или e-mail" v-model="searchText" :disabled="fetchedUsers == undefined" />
      <media-button
        v-if="showResetFiltersButton"
        iconName="clean.svg"
        iconAlt="Очистить"
        text="Очистить фильтр"
        mt="large"
        @click="clearFilters"
      />
    </div>
    
    <div class="main__data">
      <div class="main__data__sorting">
        <span class="main__data__sorting__item">Сортировка:</span>
        <media-button
          text="Дата регистрации"
          class="main__data__sorting__item--dashed"
          :class="{'active': sorting.registrationDate !== 0}"
          @click="changeSorting('registrationDate')"
          :title="getSortingTitle('registrationDate')"
        />
        <media-button
          text="Рейтинг"
          class="main__data__sorting__item--dashed"
          :class="{'active': sorting.rating !== 0}"
          @click="changeSorting('rating')"
          :title="getSortingTitle('rating')"
        />
      </div>

      <data-table
        v-if="paginatedUsers"
        :onRemove="openRemoveUserModal"
        :columns="columns.map(column => column.title)"
        :dataKeys="columns.map(column => column.name)"
        :paginatedEntries="paginatedUsers" :formatDate="formatDate"
      />
      
      <p v-if="infoMessage" class="information">{{ infoMessage }}</p>
      
      <div class="main__data__sorting" style="margin-top: 15px;">
        <media-button iconName="left.png" iconAlt="Предыдущая" size="small" @click="previousPage" :clickable="thereIsPageBefore" />
        <span class="main__data__sorting__item" style="min-width: 35px; text-align: center">{{ currentPage }} / {{ amountOfPages }}</span>
        <media-button iconName="right.png" iconAlt="Следующая" size="small" @click="nextPage" :clickable="thereIsAnotherPage" />
      </div>
    </div>
    
  </section>
</template>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:ital,opsz,wght@0,14..32,100..900;1,14..32,100..900&display=swap');
  html, body {
    padding: 0;
    margin: 0;
    box-sizing: border-box;
    background-color: #F7F7F7;
  }
  *,
  *:before,
  *::after {
    box-sizing: border-box;
    font-family: Inter;
  }
  .main {
    background-color: #F7F7F7;
    padding-top: 12px;
    padding-inline: 27px 12px;
    width: 1000px;
    margin: 0 auto;
  }
  .main__header {
    font-size: 24px;
    margin: 0;
    font-weight: 700;
    line-height: 28px;
    color: #333333;
  }
  .main__search-panel {
    box-shadow: 0px 18px 15px 0px #94949426;
    background-color: #FFFFFF;
    padding: 12px 44px 16px 16px;
    border-radius: 7px;
    margin-top: 20px;
  }
  .main__data {
    margin-top: 72px;
  }
  .main__data__sorting {
    display: flex;
    gap: 8px
  }
  .main__data__sorting__item {
    font-size: 10px;
    font-weight: 500;
    line-height: 14px;
    padding: 0;
    border: none;
    outline: none;
    background-color: transparent;
    color: #9EAAB4;
  }
  .main__data__sorting__item--dashed {
    font-size: 10px;
    font-weight: 500;
    line-height: 14px;
    background-color: transparent;
    color: #9EAAB4;
    /* Чёрточки заданы через background-image, чтобы иметь возможность изменять их длину и расстояние между ними */
    background-image: linear-gradient(to right, transparent 0%, transparent 37.5%, #9EAAB4 37.5%, #9EAAB4 62.5%);
    background-size: 8px 1px;
    background-repeat: repeat-x;
    background-position: bottom;
  }
  .main__data__sorting__item--dashed:hover {
    cursor: pointer;
  }
  .active {
    color: #333333;
    background-image: linear-gradient(to right, transparent 0%, transparent 37.5%, #333333 37.5%, #333333 62.5%);
  }
  .information {
    font-size: 12px;
    font-weight: 500;
    line-height: 16px;
    color: #ACACAC;
  }
</style>

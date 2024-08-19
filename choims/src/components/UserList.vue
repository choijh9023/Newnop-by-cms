<template>
    <div class="min-h-screen flex flex-col items-center justify-center bg-gray-100 p-4">
      <div class="w-full max-w-4xl bg-white rounded-lg shadow-lg p-6">
        <h1 class="text-3xl font-bold mb-6 text-center">유저 목록(User List)</h1>
        
       
        <div class="mb-6 w-full md:w-1/2 mx-auto">
          <input 
            v-model="searchQuery" 
            type="text" 
            placeholder="유저 검색창(Search users by name)"
            class="p-3 border border-gray-300 rounded w-full shadow"
          />
        </div>
        
        <!-- Sort Button -->
        <div class="mb-6 flex justify-center gap-4">
          <button 
            @click="sortUsersAsc" 
            class="bg-blue-500 text-white px-4 py-2 rounded shadow"
          >
            오름차순
          </button>
          <button 
            @click="sortUsersDesc" 
            class="bg-blue-500 text-white px-4 py-2 rounded shadow"
          >
            내림차순
          </button>
        </div>
        
        <!-- Error Handling-->
        <div v-if="error" class="text-red-500 mb-4 text-center">
          {{ error }}
        </div>
    
        <!-- Loading Message -->
        <div v-if="loading" class="text-center">
          <p>Loading...</p>
        </div>
    
        <!-- User List -->
        <ul v-else class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8 w-full md:w-4/5 mx-auto">
          <li 
            v-for="user in paginatedUsers" 
            :key="user.login.uuid" 
            class="bg-white p-6 rounded-lg shadow-xl text-center cursor-pointer"
            @click="selectUser(user)"
          >
            <img :src="user.picture.large" alt="User picture" class="rounded-full mb-4 w-24 h-24 mx-auto border">
            <h2 class="text-xl font-semibold mb-2">{{ user.name.first }} {{ user.name.last }}</h2>
            <p class="text-gray-500 text-sm mb-1 max-w-xs truncate">{{ user.email }}</p>




            <p class="text-gray-600">{{ user.phone }}</p>
          </li>
        </ul>
  
        <!-- 페이지네이션 버튼 -->
        <div class="mt-6 flex justify-center gap-4">
          <button 
            @click="prevPage" 
            :disabled="currentPage === 1"
            class="bg-blue-500 text-white px-4 py-2 rounded shadow disabled:bg-gray-400"
          >
            이전
          </button>
          <button 
            @click="nextPage" 
            :disabled="currentPage === totalPages"
            class="bg-blue-500 text-white px-4 py-2 rounded shadow disabled:bg-gray-400"
          >
            다음
          </button>
        </div>
      </div>
  
      <!-- 모달 -->
      <div v-if="selectedUser" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
        <div class="bg-white p-6 rounded-lg shadow-lg max-w-md w-full relative">
          <button @click="selectedUser = null" class="absolute top-0 right-0 mt-2 mr-5 text-gray-600 text-2xl">&times;</button>
          <img :src="selectedUser.picture.large" alt="User picture" class="rounded-full mb-4 w-32 h-32 mx-auto border">
          <h2 class="text-2xl font-semibold mb-2 text-center">{{ selectedUser.name.first }} {{ selectedUser.name.last }}</h2>
          <p class="text-gray-500 mb-1 text-center">{{ selectedUser.email }}</p>
          <p class="text-gray-600 mb-1 text-center">{{ selectedUser.phone }}</p>
          <p class="text-gray-600 mb-1 text-center">{{ selectedUser.location.city }}, {{ selectedUser.location.state }}, {{ selectedUser.location.country }}</p>
          <p class="text-gray-600 mb-1 text-center">Age: {{ selectedUser.dob.age }}</p>
          <p class="text-gray-600 mb-1 text-center">Username: {{ selectedUser.login.username }}</p>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    data() {
      return {
        users: [],
        searchQuery: '',
        loading: false,
        error: null,
        currentPage: 1,
        usersPerPage: 3,
        selectedUser: null, // 선택된 사용자 정보
      };
    },
    computed: {
      filteredUsers() {
        return this.users.filter(user =>
          `${user.name.first} ${user.name.last}`
            .toLowerCase()
            .includes(this.searchQuery.toLowerCase())
        );
      },
      paginatedUsers() {
        const start = (this.currentPage - 1) * this.usersPerPage;
        const end = start + this.usersPerPage;
        return this.filteredUsers.slice(start, end);
      },
      totalPages() {
        return Math.ceil(this.filteredUsers.length / this.usersPerPage);
      },
    },
    methods: {
      async fetchUsers() {
        this.loading = true;
        this.error = null;
        try {
          const response = await axios.get('https://randomuser.me/api/?results=12');
          this.users = response.data.results;
        } catch (err) {
          this.error = 'Failed to fetch users';
        } finally {
          this.loading = false;
        }
      },
      sortUsersAsc() {
        this.users.sort((a, b) =>
          a.name.first.localeCompare(b.name.first)
        );
      },
      sortUsersDesc() {
        this.users.sort((a, b) =>
          b.name.first.localeCompare(a.name.first)
        );
      },
      nextPage() {
        if (this.currentPage < this.totalPages) {
          this.currentPage++;
        }
      },
      prevPage() {
        if (this.currentPage > 1) {
          this.currentPage--;
        }
      },
      selectUser(user) {
        this.selectedUser = user; // 사용자 선택
      },
    },
    mounted() {
      this.fetchUsers();
    },
  };
  </script>
  
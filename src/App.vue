<template>
  <div class="app">
    <h2>Добавить пользователя</h2>
    <div class="form">
      <input v-model="newUser.name" placeholder="name" />
      <select v-model="newUser.gender">
        <option value="male">male</option>
        <option value="female">female</option>
      </select>
      <input v-model="newUser.email" placeholder="email" />
      <input v-model="newUser.age" type="number" placeholder="age" />
      <button @click="addUser">добавить</button>
      <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
    </div>

    <div class="ag-theme-alpine" style="height: 400px; width: 600px">
      <ag-grid-vue
        v-if="!isLoading"
        class="ag-theme-alpine"
        :columnDefs="columnDefs"
        :rowData="rowData"
        :defaultColDef="defaultColDef"
        @grid-ready="onGridReady"
      />
      <div v-else class="loading-message">Загрузка данных...</div>
    </div>

    <h3>Лог редактирования</h3>
    <ul>
      <li v-for="log in logs" :key="log">{{ log }}</li>
    </ul>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import axios from 'axios';
import "ag-grid-community/styles/ag-grid.css";
import "ag-grid-community/styles/ag-theme-alpine.css";
import { AgGridVue } from "ag-grid-vue3";

export default {
  name: "App",
  components: {
    AgGridVue,
  },
  setup() {
    const rowData = ref([]);
    const logs = ref([]);
    const newUser = ref({ name: '', gender: 'male', email: '', age: '' });
    const gridApi = ref(null);
    const isLoading = ref(true);
    const errorMessage = ref('');

    const columnDefs = ref([
      { field: 'name', headerName: 'Name' },
      { field: 'gender', headerName: 'Gender' },
      { field: 'email', headerName: 'Email' },
      { field: 'age', headerName: 'Age' },
      {
        field: 'action',
        headerName: 'Action',
        cellRenderer: (params) => {
          const button = document.createElement('button');
          button.textContent = 'Удалить';
          button.addEventListener('click', () => {
            removeUser(params.node);
          });
          return button;
        }
      }
    ]);

    const defaultColDef = ref({
      sortable: true,
      filter: true,
      resizable: true
    });

    const removeUser = (node) => {
      const removedUser = node.data;
      gridApi.value.applyTransaction({ remove: [removedUser] });
      logs.value.push(`Удален пользователь "${removedUser.name}"`);
    };

    const addUser = () => {
      if (!newUser.value.name || !newUser.value.email || !newUser.value.age) {
        errorMessage.value = 'Все поля должны быть заполнены.';
        return;
      }
      errorMessage.value = '';
      const user = { ...newUser.value };
      gridApi.value.applyTransaction({ add: [user] });
      logs.value.push(`Добавлен пользователь "${user.name}"`);
      newUser.value = { name: '', gender: 'male', email: '', age: '' };
    };

    const fetchUsers = async () => {
      try {
        const response = await axios.get('https://randomuser.me/api/', {
          params: { results: 5 }
        });
        const users = response.data.results.map(user => ({
          name: `${user.name.first} ${user.name.last}`,
          gender: user.gender,
          email: user.email,
          age: user.dob.age
        }));
        rowData.value = users;
        isLoading.value = false;
      } catch (error) {
        console.error("Ошибка при получении данных пользователей:", error);
        isLoading.value = false;
      }
    };

    const onGridReady = (params) => {
      gridApi.value = params.api;
      params.api.sizeColumnsToFit();
    };

    onMounted(() => {
      fetchUsers();
    });

    return {
      newUser,
      addUser,
      logs,
      columnDefs,
      defaultColDef,
      onGridReady,
      rowData,
      isLoading,
      errorMessage
    };
  },
};
</script>

<style>
.ag-root-wrapper-body.ag-layout-normal {
  height: auto;
}

.loading-message {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  font-size: 1.5rem;
}

.error-message {
  color: red;
}
</style>

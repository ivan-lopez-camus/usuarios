<template>
  <div class="container user-list mt-5">
    <h2 class="text-center mb-4">Listado de Usuarios</h2>

    <!-- Buscador -->
    <div class="mb-3">
      <input
        type="text"
        class="form-control"
        placeholder="Buscar por nombre..."
        v-model="searchQuery"
        @input="filterUsers"
      />
    </div>

    <!-- Mensaje de carga -->
    <div
      v-if="loading"
      class="d-flex justify-content-center align-items-center"
    >
      <div class="spinner-border text-primary" role="status">
        <span class="visually-hidden">Cargando...</span>
      </div>
      <span class="ms-2">Cargando...</span>
    </div>

    <!-- Tabla de usuarios -->
    <div v-if="!loading && !error" class="table-responsive">
      <table class="table table-striped table-bordered">
        <thead class="table-dark">
          <tr>
            <th scope="col">
              <input type="checkbox" v-model="selectAll" @change="toggleAll" />
            </th>
            <th scope="col">ID</th>
            <th scope="col">Nombre</th>
            <th scope="col">Email</th>
            <th scope="col">Dirección</th>
            <th scope="col">Compañía</th>
            <th scope="col">Teléfono</th>
            <th scope="col">Sitio Web</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="user in paginatedUsers" :key="user.id">
            <td>
              <input type="checkbox" v-model="selectedUsers" :value="user.id" />
            </td>
            <td>{{ user.id }}</td>
            <td>{{ user.name }}</td>
            <td>{{ user.email }}</td>
            <td>
              {{
                `${user.address.street}, ${user.address.suite}, ${user.address.city}`
              }}
            </td>
            <td>{{ user.company.name }}</td>
            <td>{{ user.phone }}</td>
            <td>
              <a
                :href="'https://' + user.website"
                target="_blank"
                class="link-primary"
              >
                {{ user.website }}
              </a>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Paginador -->
    <nav v-if="!loading && !error && filteredUsers.length > 0">
      <ul class="pagination justify-content-center">
        <li class="page-item" :class="{ disabled: currentPage === 1 }">
          <button class="page-link" @click="prevPage">Anterior</button>
        </li>
        <li class="page-item" :class="{ disabled: currentPage === totalPages }">
          <button class="page-link" @click="nextPage">Siguiente</button>
        </li>
      </ul>
    </nav>

    <!-- Modal de error -->
    <div
      class="modal fade"
      id="errorModal"
      tabindex="-1"
      aria-labelledby="errorModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="errorModalLabel">Error</h5>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
            ></button>
          </div>
          <div class="modal-body">
            {{ error }}
          </div>
          <div class="modal-footer">
            <button
              type="button"
              class="btn btn-secondary"
              data-bs-dismiss="modal"
            >
              Cerrar
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { Modal } from "bootstrap";

export default {
  data() {
    return {
      users: [],
      filteredUsers: [],
      paginatedUsers: [],
      selectedUsers: [],
      searchQuery: "",
      loading: true,
      error: null,
      currentPage: 1,
      itemsPerPage: 5,
      selectAll: false,
    };
  },
  computed: {
    totalPages() {
      return Math.ceil(this.filteredUsers.length / this.itemsPerPage);
    },
  },
  mounted() {
    axios
      .get("https://jsonplaceholder.typicode.com/users")
      .then((response) => {
        this.users = response.data;
        this.filteredUsers = this.users;
        this.updatePaginatedUsers();
        this.loading = false;
      })
      .catch(() => {
        this.error = "Hubo un error al cargar los datos";
        this.loading = false;
        this.showErrorModal();
      });
  },
  methods: {
    showErrorModal() {
      const modal = new Modal(document.getElementById("errorModal"));
      modal.show();
    },
    filterUsers() {
      this.filteredUsers = this.users.filter((user) =>
        user.name.toLowerCase().includes(this.searchQuery.toLowerCase())
      );
      this.currentPage = 1;
      this.updatePaginatedUsers();
    },
    updatePaginatedUsers() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      this.paginatedUsers = this.filteredUsers.slice(start, end);
    },
    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
        this.updatePaginatedUsers();
      }
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
        this.updatePaginatedUsers();
      }
    },
    toggleAll() {
      if (this.selectAll) {
        this.selectedUsers = this.paginatedUsers.map((user) => user.id);
      } else {
        this.selectedUsers = [];
      }
    },
  },
  watch: {
    selectedUsers() {
      this.selectAll =
        this.paginatedUsers.length > 0 &&
        this.paginatedUsers.every((user) =>
          this.selectedUsers.includes(user.id)
        );
    },
  },
};
</script>

<style scoped>
.table-responsive {
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  overflow: hidden;
}

h2 {
  font-weight: bold;
  color: #333;
}

.spinner-border {
  width: 3rem;
  height: 3rem;
}

.pagination {
  margin-top: 15px;
}
</style>

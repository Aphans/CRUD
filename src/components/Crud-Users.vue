<template>
    <!-- Formulario para filtrar por nivel de usuario -->
    <form @submit.prevent="filterUsers">
      <label>
        Filtrar por nivel:
        <select v-model="filterLevel">
          <option value="">Todos</option>
          <option value="1">Administrador</option>
          <option value="0">Usuario</option>
        </select>
      </label>
      <button type="submit">Filtrar</button>
    </form>

    <!-- Sección para mostrar todos los usuarios -->
    <table>
      <thead>
        <tr>
          <th>ID</th>
          <th>Nombre</th>
          <th>Apellidos</th>
          <th>Nivel de usuario</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="user in filteredUsers" :key="user.id">
          <td>{{ user.id }}</td>
          <td>{{ user.name }}</td>
          <td>{{ user.surname }}</td>
          <td>{{ user.level==1? 'Administrador' : 'Usuario' }}</td>
          <td>
            <!-- Botones para editar y eliminar usuarios -->
            <button @click="editUser(user)">Editar</button>
            <button @click="deleteUser(user)">Eliminar</button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Formulario para crear un nuevo usuario -->
    <form @submit.prevent="createUser">
      <label>
        Nombre:
        <input v-model="newUser.name" />
      </label>
      <label>
        Apellidos:
        <input v-model="newUser.surname" />
      </label>
      <label>
        Nivel de usuario:
        <select v-model="newUser.level">
          <option value="1">Administrador</option>
          <option value="0">Usuario</option>
        </select>
      </label>
      <button type="submit">Crear usuario</button>
    </form>
    <!-- Sección para editar un usuario existente -->
<form v-if="editingUser" @submit.prevent="updateUser">
  <label>
    Nombre:
    <input v-model="editingUser.name" />
  </label>
  <label>
    Apellidos:
    <input v-model="editingUser.surname" />
  </label>
  <label>
    Nivel de usuario:
    <select v-model="editingUser.level">
      <option value="1">Administrador</option>
      <option value="0">Usuario</option>
    </select>
  </label>
  <button type="submit">Guardar cambios</button>
  <button @click="cancelEditing">Cancelar</button>
</form>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      users: [],
      editingUser: null,
      filteredUsers: [],
      filterLevel: '',
      // Propiedad para almacenar los datos del nuevo usuario
      newUser: {
        name: '',
        surname: '',
        level: ''
      },
    }
  },
  mounted(){
    this.showUsers();
  },
  updated(){
    this.showUsers();
  },
  methods: {
    async filterUsers() {
  if (this.filterLevel) {
    this.filteredUsers = this.users.filter(user => user.level === this.filterLevel)
  } else {
    this.filteredUsers = this.users
  }
},
    editUser(user) {
      this.editingUser = user
      // Muestre el formulario de edición aquí
    },
    async showUsers(){
      try{
        const response = await axios.get(`http://localhost:4000/usuarios/`);
        this.users = response.data;
      }
      catch(error){
        console.error(error);
      }
    },
    async updateUser() {
      // Envíe una solicitud HTTP PUT para actualizar el usuario en el servidor
      try {
        const response = await axios.put(`http://localhost:4000/usuarios/${this.editingUser.id}`, this.editingUser)
        const updatedUser = response.data;
        // Actualice la lista de usuarios en el componente de Vue
        this.users = this.users.map(user => user.id === updatedUser.id ? updatedUser : user)
      } catch (error) {
        console.error(error)
      }
    },
    async deleteUser(user) {
      // Envíe una solicitud HTTP DELETE para eliminar el usuario del servidor
      try {
        await axios.delete(`http://localhost:4000/usuarios/${user.id}`)
        // Elimine el usuario de la lista de usuarios en el componente de Vue
        this.users = this.users.filter(u => u.id !== user.id)
      } catch (error) {
        console.error(error)
      }
    },
    async createUser() {
  const newUser = {
    name: this.newUser.name,
  surname: this.newUser.surname,
  level: this.newUser.level
  }
  // Envíe una solicitud HTTP POST para crear el nuevo usuario en el servidor
  try {
    const response = await axios.post(`http://localhost:4000/usuarios`, newUser)
    this.users = response.data;
  } catch (error) {
    console.log(error)
  }
  //Clean 
  this.newUser = {
  name: '',
  surname: '',
  level: ''
  }
}
  }
}
</script>
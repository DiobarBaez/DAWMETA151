<template>
  <v-app>
    <v-container>
      <v-row>
        <v-col cols="12">
          <h2>Meta CRUD App</h2>
          <v-btn @click="isAddRegistroModalActive = true" color="primary">Agregar Registro</v-btn>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12">
          <table>
            <thead>
              <tr>
                <th>ID</th>
                <th>Título</th>
                <th>Contenido</th>
                <th>Acción</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="registro in registros" :key="registro.id">
                <td>{{ registro.id }}</td>
                <td>{{ registro.title }}</td>
                <td>{{ registro.contenido }}</td>
                <td>
                  <v-btn @click="deleteRegistroConfirmation(registro.id)" color="error">Eliminar Registro</v-btn>
                  <v-btn @click="openUpdateModal(registro)">Actualizar Registro</v-btn>
                </td>
              </tr>
            </tbody>
          </table>
        </v-col>
      </v-row>

      <!-- Modal para agregar un nuevo registro -->
      <v-dialog v-model="isAddRegistroModalActive">
        <v-card>
          <v-card-title>
            <span class="headline">Agregar Registro</span>
          </v-card-title>
          <v-card-text>
            <v-text-field v-model="newRegistroID" label="ID"></v-text-field>
            <v-text-field v-model="newRegistroTitulo" label="Título"></v-text-field>
            <v-text-field v-model="newRegistroContenido" label="Contenido"></v-text-field>
          </v-card-text>
          <v-card-actions>
            <v-btn @click="addNewRegistro" color="primary">Agregar</v-btn>
            <v-btn @click="closeAddRegistroModal">Cancelar</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <!-- Modal para actualizar el registro -->
      <v-dialog v-model="isUpdateModalActive">
        <v-card>
          <v-card-title>
            <span class="headline">Actualizar Registro</span>
          </v-card-title>
          <v-card-text>
            <v-text-field v-model="updatedRegistroTitulo" label="Título Actualizado"></v-text-field>
            <v-text-field v-model="updatedRegistroContenido" label="Contenido Actualizado"></v-text-field>
          </v-card-text>
          <v-card-actions>
            <v-btn @click="updateRegistro" color="primary">Actualizar</v-btn>
            <v-btn @click="closeUpdateModal">Cancelar</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <!-- Modal de confirmación de eliminación -->
      <v-dialog v-model="isDeleteConfirmationActive">
        <v-card>
          <v-card-title>
            <span class="headline">Confirmar Eliminación</span>
          </v-card-title>
          <v-card-text>
            ¿Estás seguro de que deseas eliminar este registro?
          </v-card-text>
          <v-card-actions>
            <v-btn @click="deleteRegistroConfirmed" color="error">Eliminar</v-btn>
            <v-btn @click="cancelDeleteRegistro">Cancelar</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <!-- Mensaje de retroalimentación -->
      <v-snackbar v-model="isSnackbarVisible" :timeout="1000" color="primary">
        {{ feedbackMessage }}
      </v-snackbar>
    </v-container>
  </v-app>
</template>

<script>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const API_URL = "http://localhost:5038/api/meta_crud/";

export default {
  setup() {
    const newRegistroID = ref("");
    const newRegistroTitulo = ref("");
    const newRegistroContenido = ref("");
    const registros = ref([]);
    const isUpdateModalActive = ref(false);
    const isDeleteConfirmationActive = ref(false);
    const selectedRegistro = ref(null);
    const updatedRegistroTitulo = ref("");
    const updatedRegistroContenido = ref("");
    const feedbackMessage = ref("");
    const isAddRegistroModalActive = ref(false);
    const isSnackbarVisible = ref(false);

    const refreshData = async () => {
      try {
        const response = await axios.get(API_URL + "GetRegistros");
        registros.value = response.data;
      } catch (error) {
        console.error("Error al obtener registros:", error);
      }
    };

    const addNewRegistro = async () => {
      if (!newRegistroID.value || !newRegistroTitulo.value || !newRegistroContenido.value) {
        feedbackMessage.value = "Por favor, completa todos los campos.";
        return;
      }

      try {
        const response = await axios.post(API_URL + "AddRegistro", {
          id: newRegistroID.value,
          title: newRegistroTitulo.value,
          contenido: newRegistroContenido.value,
        });
        feedbackMessage.value = response.data.message;
        newRegistroID.value = "";
        newRegistroTitulo.value = "";
        newRegistroContenido.value = "";
        isAddRegistroModalActive.value = false;
        refreshData();
      } catch (error) {
        console.error("Error al agregar registro:", error);
        if (error.response) {
          console.error("Respuesta del servidor:", error.response.data);
          feedbackMessage.value = "Error en la solicitud al servidor: " + error.response.data.message;
        } else {
          feedbackMessage.value = "Error al agregar registro.";
        }
      }
    };

    const deleteRegistroConfirmation = (id) => {
      selectedRegistro.value = id;
      isDeleteConfirmationActive.value = true;
    };

    const deleteRegistroConfirmed = async () => {
      if (selectedRegistro.value !== null) {
        try {
          const response = await axios.delete(API_URL + `DeleteRegistro/${selectedRegistro.value}`);
          refreshData();
          feedbackMessage.value = response.data.message;
        } catch (error) {
          console.error("Error al eliminar registro:", error);
          feedbackMessage.value = "Error al eliminar registro.";
        } finally {
          selectedRegistro.value = null;
          isDeleteConfirmationActive.value = false;
        }
      }
    };

    const cancelDeleteRegistro = () => {
      selectedRegistro.value = null;
      isDeleteConfirmationActive.value = false;
    };

    const openUpdateModal = (registro) => {
      selectedRegistro.value = registro;
      updatedRegistroTitulo.value = registro.title;
      updatedRegistroContenido.value = registro.contenido;
      isUpdateModalActive.value = true;
    };

    const closeUpdateModal = () => {
      selectedRegistro.value = null;
      updatedRegistroTitulo.value = "";
      updatedRegistroContenido.value = "";
      isUpdateModalActive.value = false;
    };

    const updateRegistro = async () => {
      try {
        const response = await axios.put(API_URL + `UpdateRegistro/${selectedRegistro.value.id}`, {
          title: updatedRegistroTitulo.value,
          contenido: updatedRegistroContenido.value,
        });
        refreshData();
        feedbackMessage.value = response.data.message;
        closeUpdateModal();
      } catch (error) {
        console.error("Error al actualizar registro:", error);
        feedbackMessage.value = "Error al actualizar registro.";
      }
    };

    const closeAddRegistroModal = () => {
      isAddRegistroModalActive.value = false;
    };

    onMounted(() => {
      refreshData();
    });

    return {
      newRegistroID,
      newRegistroTitulo,
      newRegistroContenido,
      registros,
      isUpdateModalActive,
      isDeleteConfirmationActive,
      selectedRegistro,
      updatedRegistroTitulo,
      updatedRegistroContenido,
      feedbackMessage,
      isAddRegistroModalActive,
      isSnackbarVisible,
      refreshData,
      addNewRegistro,
      deleteRegistroConfirmation,
      deleteRegistroConfirmed,
      cancelDeleteRegistro,
      openUpdateModal,
      closeUpdateModal,
      updateRegistro,
      closeAddRegistroModal,
    };
  },
};
</script>

<style>
/* Estilos para la tabla */
table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

thead {
  background-color: #f2f2f2;
}

th, td {
  padding: 10px;
  border: 1px solid #ccc;
  text-align: left;
}

/* Estilos para los botones */
.v-btn {
  margin-right: 10px;
}

/* Estilos para los diálogos */
.v-dialog {
  max-width: 400px;
  margin: 0 auto;
}
</style>

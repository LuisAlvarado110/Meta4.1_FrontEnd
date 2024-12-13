<template>
  <v-container>
    <v-card>
      <v-card-title>
        <div class="text-center">Lista de Estudiantes</div>
        <v-spacer />
        <v-row align="center" justify="space-between" class="w-100">
          <!-- Barra de búsqueda -->
          <v-col cols="12" md="4">
            <v-text-field
              v-model="filtro"
              label="Buscar por matrícula"
              class="mr-4"
              prepend-icon="mdi-account-search"
            ></v-text-field>
          </v-col>
          <!-- Botón de agregar estudiante -->
          <v-col cols="auto">
            <v-btn prepend-icon="mdi-account-plus" color="green" @click="abrirDialogoCrear">Agregar Estudiante</v-btn>
          </v-col>
        </v-row>
      </v-card-title>
      <v-card-text>
        <v-alert v-if="error" type="error" dismissible>
          {{ error }}
        </v-alert>

        <v-data-table
          v-if="filtrarEstudiantes.length"
          :headers="headers"
          :items="filtrarEstudiantes"
          :items-per-page="5"
          class="elevation-1"
        >
          <template #item.acciones="{ item }">
            <v-icon small class="mr-2" @click="editarEstudiante(item)">mdi-pencil</v-icon>
            <v-icon small class="mr-2" color="blue" @click="verCursos(item)">mdi-book-variant</v-icon>
            <v-icon small class="mr-2" color="green" @click="verProfesores(item)">mdi-human-male-board</v-icon>
            <v-icon small class="mr-2" color="red" @click="abrirDialogoEliminar(item)">mdi-delete</v-icon>
          </template>
        </v-data-table>

        <v-alert v-else type="info" dismissible>
          No hay estudiantes disponibles para mostrar.
        </v-alert>
      </v-card-text>
    </v-card>

    <!-- Diálogo para Crear/Editar -->
    <v-dialog v-model="dialog" max-width="500px">
      <v-card>
        <v-card-title>
          {{ dialogModo === 'crear' ? 'Crear Estudiante' : 'Editar Estudiante' }}
        </v-card-title>
        <v-card-text>
          <v-form ref="form">
            <v-text-field
              v-model="formEstudiante.matricula"
              label="Matrícula"
              required
              :disabled="dialogModo === 'editar'"
            ></v-text-field>
            <v-text-field
              v-model="formEstudiante.nombre"
              label="Nombre"
              required
            ></v-text-field>
            <v-text-field
              v-model="formEstudiante.creditos"
              label="Créditos"
              required
            ></v-text-field>
            <v-text-field
              v-model="formEstudiante.semestre"
              label="Semestre"
              required
            ></v-text-field>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-btn text @click="dialog = false">Cancelar</v-btn>
          <v-btn color="primary" @click="guardarEstudiante">Guardar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Diálogo de eliminación -->
    <v-dialog v-model="deleteDialog" max-width="400">
      <v-card>
        <v-card-title class="text-h6">¿Borrar?</v-card-title>
        <v-card-text>
          ¿Estás seguro de que deseas borrar los datos del estudiante: <strong>{{ estudianteAEliminar?.nombre }}</strong>?
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn text @click="cerrarDialogoEliminar">Cancelar</v-btn>
          <v-btn color="red" text @click="confirmarEliminarEstudiante">Borrar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Diálogo de cursos -->
    <v-dialog v-model="cursosDialog" max-width="600px">
      <v-card>
        <v-card-title>
          Cursos del estudiante {{ estudianteSeleccionado?.nombre }}
        </v-card-title>
        <v-card-text>
          <v-data-table
            :headers="cursoHeaders"
            :items="cursos"
            :items-per-page="5"
            class="elevation-1"
          >
          <template #item.acciones="{ item }">
            <v-icon small color="red" @click="abrirDialogoEliminarCurso(item)">mdi-book-remove</v-icon>
          </template>
        </v-data-table>
          <v-btn
            prepend-icon="mdi-plus"
            color="green"
            class="mt-3"
            @click="abrirDialogoAltaCurso"
          >
            Dar de alta en curso
          </v-btn>
        </v-card-text>
        <v-card-actions>
          <v-btn text @click="cursosDialog = false">Cerrar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Diálogo para dar de alta en curso -->
    <v-dialog v-model="altaCursoDialog" max-width="400px">
      <v-card>
        <v-card-title>Dar de alta en curso</v-card-title>
        <v-card-text>
          <v-form ref="formAltaCurso">
            <v-text-field
              v-model="cursoClave"
              label="Clave del Curso"
              required
            ></v-text-field>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-btn text @click="altaCursoDialog = false">Cancelar</v-btn>
          <v-btn color="primary" @click="darAltaEnCurso">Aceptar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Diálogo para confirmar baja de curso -->
    <v-dialog v-model="eliminarCursoDialog" max-width="400px">
      <v-card>
        <v-card-title class="text-h6">Confirmar Baja de Curso</v-card-title>
        <v-card-text>
          ¿Estás seguro de que deseas dar de baja el curso: <strong>{{ cursoAEliminar?.nombreCurso }}</strong>?
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn text @click="cerrarDialogoEliminarCurso">Cancelar</v-btn>
          <v-btn color="red" text @click="confirmarEliminarCurso">Confirmar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!--Diálogo de profesores asociados-->
    <v-dialog v-model="profesoresDialog" max-width="600px">
      <v-card>
        <v-card-title>
          Profesores de los cursos del estudiante {{ estudianteSeleccionado?.nombre }}
        </v-card-title>
        <v-card-text>
          <v-data-table
            :headers="profesorHeaders"
            :items="profesores"
            :items-per-page="5"
            class="elevation-1"
          ></v-data-table>
        </v-card-text>
        <v-card-actions>
          <v-btn text @click="profesoresDialog = false">Cerrar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  name: "TablaEstudiantes",
  data() {
    return {
      estudiantes: [],
      filtro: "",
      headers: [
        { title: "Matrícula", key: "matricula", sortable: false },
        { title: "Nombre", key: "nombre", sortable: false },
        { title: "Créditos", key: "creditos", sortable: false },
        { title: "Semestre", key: "semestre", sortable: false },
        { title: "Acciones", key: "acciones", sortable: false },
      ],
      cursos: [],
      cursoHeaders: [
        { title: "Clave del Curso", key: "claveCurso", sortable: false },
        { title: "Nombre del Curso", key: "nombreCurso", sortable: false },
        { title: "Acciones", key: "acciones", sortable: false },
      ],
      profesores: [],
      profesorHeaders: [
        { title: "Número de Empleado", key: "numEmpleado", sortable: false },
        { title: "Nombre del Profesor", key: "nombre", sortable: false },
      ],
      profesoresDialog: false,
      dialog: false,
      deleteDialog: false,
      cursosDialog: false,
      altaCursoDialog: false,
      eliminarCursoDialog: false,
      dialogModo: "crear",
      formEstudiante: {
        matricula: "",
        nombre: "",
        creditos: "",
        semestre: ""
      },
      cursoClave: "",
      estudianteAEliminar: null,
      estudianteSeleccionado: null,
      cursoAEliminar: null,
      error: null,
    };
  },
  computed: {
    filtrarEstudiantes() {
      const filtroLower = this.filtro.toLowerCase();
      return this.estudiantes.filter(estudiante =>
        estudiante.matricula.toLowerCase().includes(filtroLower)
      );
    }
  },
  methods: {
    async cargarEstudiantes() {
      try {
        const response = await axios.get("https://localhost:5000/getAllEstudiantes");
        this.estudiantes = response.data;
        this.error = null;
      } catch (error) {
        console.error("Error al cargar estudiantes:", error);
      }
    },
    abrirDialogoCrear() {
      this.dialogModo = "crear";
      this.formEstudiante = { nombre: "", matricula: "", creditos: "", semestre: "" };
      this.dialog = true;
    },
    editarEstudiante(estudiante) {
      this.dialogModo = "editar";
      this.formEstudiante = { ...estudiante };
      this.dialog = true;
    },
    async guardarEstudiante() {
      try {
        if (this.dialogModo === "crear") {
          await axios.post("https://localhost:5000/createEstudiante", this.formEstudiante);
        } else {
          await axios.put(
            `https://localhost:5000/updateEstudiante/${this.formEstudiante.matricula}`,
            this.formEstudiante
          );
        }
        this.dialog = false;
        this.cargarEstudiantes();
      } catch (error) {
        console.error("Error al guardar estudiante:", error);
      }
    },
    abrirDialogoEliminar(estudiante) {
      this.estudianteAEliminar = estudiante;
      this.deleteDialog = true;
    },
    cerrarDialogoEliminar() {
      this.estudianteAEliminar = null;
      this.deleteDialog = false;
    },
    async confirmarEliminarEstudiante() {
      try {
        await axios.delete(`https://localhost:5000/deleteEstudiante/${this.estudianteAEliminar.matricula}`);
        this.cerrarDialogoEliminar();
        this.cargarEstudiantes();
      } catch (error) {
        console.error("Error al eliminar estudiante:", error);
      }
    },
    async verCursos(estudiante) {
      this.estudianteSeleccionado = estudiante;
      try {
        const response = await axios.get(`https://localhost:5000/getCursosEstudiante/${estudiante.matricula}`);
        if (response.data && response.data.cursosInscritos) {
          this.cursos = response.data.cursosInscritos; // Ajustar a la estructura esperada
        } else {
          this.cursos = []; // Manejar caso donde no haya cursos inscritos
        }
        this.cursosDialog = true; // Mostrar el diálogo de cursos
      } catch (error) {
        console.error("Error al cargar cursos:", error);
        this.$toast.error("No se pudieron cargar los cursos del estudiante."); // Notificación opcional
      }
    },
    abrirDialogoAltaCurso() {
      this.cursoClave = "";
      this.altaCursoDialog = true;
    },
    async darAltaEnCurso() {
      try {
        await axios.patch(`https://localhost:5000/darAltaEstudiante/${this.estudianteSeleccionado.matricula}`, {
          claveCurso: this.cursoClave
        });
        this.altaCursoDialog = false;
        this.verCursos(this.estudianteSeleccionado); // Actualizar lista de cursos
      } catch (error) {
        console.error("Error al inscribir en curso:", error);
        this.$toast.error("No se pudo inscribir al curso."); // Notificación opcional
      }
    },
    abrirDialogoEliminarCurso(curso) {
      this.cursoAEliminar = curso;
      this.eliminarCursoDialog = true;
    },
    cerrarDialogoEliminarCurso() {
      this.cursoAEliminar = null;
      this.eliminarCursoDialog = false;
    },
    async confirmarEliminarCurso() {
      try {
        await axios.patch(`https://localhost:5000/darBajaEstudiante/${this.estudianteSeleccionado.matricula}`, {
          claveCurso: this.cursoAEliminar.claveCurso
        });
        this.eliminarCursoDialog = false;
        this.verCursos(this.estudianteSeleccionado); // Actualizar lista de cursos
      } catch (error) {
        console.error("Error al desinscribir del curso:", error);
        this.$toast.error("No se pudo desinscribir del curso."); // Notificación opcional
      }
    },
    async verProfesores(estudiante) {
      this.estudianteSeleccionado = estudiante;
      try {
        const response = await axios.get(`https://localhost:5000/getProfesoresEstudiante/${estudiante.matricula}`);

        if (response.data && response.data.profesores) {
          this.profesores = response.data.profesores;
        } else {
          this.profesores = [];
        }
        this.profesoresDialog = true;
      } catch (error) {
        console.error("Error al cargar los profesores:", error);
        this.$toast.error("No se pudieron cargar los profesores del estudiante.");
      }
    }
  },
  mounted() {
    this.cargarEstudiantes();
  },
};
</script>

<style scoped>
.elevation-1 {
  margin-top: 20px;
}
.mb-4 {
  margin-bottom: 16px;
}
</style>

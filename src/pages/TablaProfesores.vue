<template>
  <v-container>
    <v-card>
      <v-card-title>
        <div class="text-center">Lista de Profesores</div>
        <v-spacer />
        <v-row align="center" justify="space-between" class="w-100">
          <!-- Barra de búsqueda -->
          <v-col cols="12" md="4">
            <v-text-field
              v-model="filtro"
              label="Buscar por número de empleado"
              class="mr-4"
              prepend-icon="mdi-account-search"
            ></v-text-field>
          </v-col>
          <!-- Botón de agregar curso -->
          <v-col cols="auto">
            <v-btn prepend-icon="mdi-account-plus" color="green" @click="abrirDialogoCrear">Agregar Profesor</v-btn>
          </v-col>
        </v-row>
      </v-card-title>
      <v-card-text>
        <v-alert v-if="error" type="error" dismissible>
          {{ error }}
        </v-alert>

        <v-data-table
          v-if="filtrarProfesores.length"
          :headers="headers"
          :items="filtrarProfesores"
          :items-per-page="5"
          class="elevation-1"
        >
          <template #item.acciones="{ item }">
            <v-icon small class="mr-2" @click="editarProfesor(item)">mdi-pencil</v-icon>
            <v-icon small class="mr-2" color="blue" @click="gestionarCursos(item)">mdi-book-variant</v-icon>
            <v-icon small class="mr-2" color="green" @click="verEstudiantes(item)">mdi-account-school</v-icon>
            <v-icon small class="mr-2" color="red" @click="abrirDialogoEliminar(item)">mdi-delete</v-icon>
          </template>
        </v-data-table>
        <v-alert v-else type="info" dismissible>
          No hay profesores disponibles para mostrar.
        </v-alert>
      </v-card-text>
    </v-card>

    <!-- Diálogo para Crear/Editar -->
    <v-dialog v-model="dialog" max-width="500px">
      <v-card>
        <v-card-title>
          {{ dialogModo === 'crear' ? 'Crear Profesor' : 'Editar Profesor' }}
        </v-card-title>
        <v-card-text>
          <v-form ref="form">
            <v-text-field
              v-model="formProfesor.nombre"
              label="Nombre"
              required
            ></v-text-field>
            <v-text-field
              v-model="formProfesor.numEmpleado"
              label="Número de Empleado"
              required
              :disabled="dialogModo === 'editar'"
            ></v-text-field>
            <v-text-field
              v-model="formProfesor.departamento"
              label="Área de estudio"
              required
            ></v-text-field>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-btn text @click="dialog = false">Cancelar</v-btn>
          <v-btn color="primary" @click="guardarProfesor">Guardar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Diálogo de Gestión de Cursos -->
    <v-dialog v-model="cursosDialog" max-width="600px">
      <v-card>
        <v-card-title>
          Cursos de {{ profesorSeleccionado?.nombre }}
        </v-card-title>
        <v-card-text>
          <v-alert v-if="errorCursos" type="error" dismissible>
            {{ errorCursos }}
          </v-alert>
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

    <!-- Diálogo de Eliminación -->
    <v-dialog v-model="deleteDialog" max-width="400">
      <v-card>
        <v-card-title class="text-h6">¿Borrar?</v-card-title>
        <v-card-text>
          ¿Estás seguro de que deseas borrar los datos del profesor
          <strong>{{ profesorSeleccionado?.nombre }}</strong>?
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn text @click="closeDeleteDialog">Cancelar</v-btn>
          <v-btn color="red" text @click="confirmarEliminarProfesor">Borrar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!--Diálogo de estudiantes asociados-->
    <v-dialog v-model="estudiantesDialog" max-width="600px">
      <v-card>
        <v-card-title>
          Estudiantes de los cursos del profesor {{ profesorSeleccionado?.nombre }}
        </v-card-title>
        <v-card-text>
          <v-data-table
            :headers="estudiantesHeaders"
            :items="estudiantes"
            :items-per-page="5"
            class="elevation-1"
          ></v-data-table>
        </v-card-text>
        <v-card-actions>
          <v-btn text @click="estudiantesDialog = false">Cerrar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  name: "TablaProfesores",
  data() {
    return {
      profesores: [],
      filtro: "",
      headers: [
        { title: "Número de Empleado", key: "numEmpleado", sortable: false },
        { title: "Nombre", key: "nombre", sortable: false },
        { title: "Área de estudio", key: "departamento", sortable: false },
        { title: "Acciones", key: "acciones", sortable: false },
      ],
      cursos: [],
      cursoHeaders: [
        { title: "Clave del Curso", key: "claveCurso", sortable: false },
        { title: "Nombre del Curso", key: "nombreCurso", sortable: false },
        { title: "Acciones", key: "acciones", sortable: false },
      ],
      estudiantes: [],
      estudiantesHeaders: [
        { title: "Matrícula", key: "matricula", sortable: false },
        { title: "Nombre", key: "nombre", sortable: false }
      ],
      dialog: false,
      cursosDialog: false,
      altaCursoDialog: false,
      eliminarCursoDialog: false,
      estudiantesDialog: false,
      dialogModo: "crear", // 'crear' o 'editar'
      formProfesor: {
        nombre: "",
        numEmpleado: "",
        departamento: "",
      },
      deleteDialog: false,
      profesorSeleccionado: null,
      cursoClave: "",
      cursoAEliminar: null,
      error: null,
      errorCursos: null,
    };
  },
  computed: {
    filtrarProfesores() {
      const filtroLower = this.filtro.toLowerCase();
      return this.filtro
        ? this.profesores.filter((profesor) =>
            String(profesor.numEmpleado).toLowerCase().includes(filtroLower)
          )
        : this.profesores;
    },
  },

  methods: {
    async cargarProfesores() {
      try {
        const response = await axios.get("https://localhost:5000/getAllProfesores");
        this.profesores = response.data.data;
      } catch (error) {
        console.error("Error al cargar profesores:", error);
      }
    },
    abrirDialogoCrear() {
      this.dialogModo = "crear";
      this.formProfesor = { nombre: "", numEmpleado: "", departamento: "" };
      this.dialog = true;
    },
    editarProfesor(profesor) {
      this.dialogModo = "editar";
      this.formProfesor = { ...profesor };
      this.dialog = true;
    },
    async guardarProfesor() {
      try {
        if (this.dialogModo === "crear") {
          await axios.post("https://localhost:5000/createProfesor", this.formProfesor);
        } else {
          await axios.put(
            `https://localhost:5000/updateProfesor/${this.formProfesor.numEmpleado}`,
            this.formProfesor
          );
        }
        this.dialog = false;
        this.cargarProfesores();
      } catch (error) {
        console.error("Error al guardar profesor:", error);
      }
    },
    abrirDialogoEliminar(profesor) {
      this.profesorSeleccionado = profesor;
      this.deleteDialog = true;
    },
    closeDeleteDialog() {
      this.profesorSeleccionado = null;
      this.deleteDialog = false;
    },
    async confirmarEliminarProfesor() {
      try {
        await axios.delete(
          `https://localhost:5000/deleteProfesor/${this.profesorSeleccionado.numEmpleado}`
        );
        this.closeDeleteDialog();
        this.cargarProfesores();
      } catch (error) {
        console.error("Error al eliminar profesor:", error);
      }
    },
    gestionarCursos(profesor) {
      this.profesorSeleccionado = profesor;
      this.verCursos(profesor);
    },
    async verCursos(profesor) {
      try {
        const response = await axios.get(
          `https://localhost:5000/getCursosProfesor/${profesor.numEmpleado}`
        );
        this.cursos = response.data.cursosImpartidos || [];
        this.cursosDialog = true;
      } catch (error) {
        console.error("Error al cargar cursos:", error);
        this.errorCursos = "No se pudieron cargar los cursos del profesor.";
      }
    },
    abrirDialogoAltaCurso() {
      this.cursoClave = "";
      this.altaCursoDialog = true;
    },
    async darAltaEnCurso() {
      try {
        await axios.patch(`https://localhost:5000/darAltaProfesor/${this.profesorSeleccionado.numEmpleado}`,
          { claveCurso: this.cursoClave }
        );
        this.altaCursoDialog = false;
        this.verCursos(this.profesorSeleccionado);
      } catch (error) {
        console.error("Error al inscribir en curso:", error);
        this.$toast.error("No se pudo inscribir al curso.");
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
        await axios.patch(
          `https://localhost:5000/darBajaProfesor/${this.profesorSeleccionado.numEmpleado}`,
          { claveCurso: this.cursoAEliminar.claveCurso }
        );
        this.eliminarCursoDialog = false;
        this.verCursos(this.profesorSeleccionado);
      } catch (error) {
        console.error("Error al desinscribir del curso:", error);
        this.$toast.error("No se pudo desinscribir del curso.");
      }
    },
    async verEstudiantes(profesor) {
      this.profesorSeleccionado = profesor;
      try {
        const response = await axios.get(`https://localhost:5000/getEstudiantesProfesor/${profesor.numEmpleado}`);

        if (response.data && response.data.estudiantes) {
          this.estudiantes = response.data.estudiantes;
        } else {
          this.estudiantes = [];
        }
        this.estudiantesDialog = true;
      } catch (error) {
        console.error("Error al cargar los estudiantes:", error);
        this.$toast.error("No se pudieron cargar los estudiantes del profesor.");
      }
    }
  },
  mounted() {
    this.cargarProfesores();
  },
};
</script>

<style scoped>
.elevation-1 {
  margin-top: 20px;
}
</style>

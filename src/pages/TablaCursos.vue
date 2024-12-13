<template>
  <v-container>
    <v-card>
      <v-card-title>
        <div class="text-center">Lista de Cursos</div>
        <v-spacer />
        <v-spacer />
        <v-row align="center" justify="space-between" class="w-100">
          <!-- Barra de búsqueda -->
          <v-col cols="12" md="4">
            <v-text-field
              v-model="filtroClaveCurso"
              label="Buscar por clave del curso"
              class="mr-4"
              prepend-icon="mdi-book-search"
            ></v-text-field>
          </v-col>
          <!-- Botón de agregar curso -->
          <v-col cols="auto">
            <v-btn prepend-icon="mdi-book-plus" color="green" @click="abrirDialogoCrear">Agregar Curso</v-btn>
          </v-col>
        </v-row>
      </v-card-title>
      <v-card-text>


        <!-- Tabla de cursos -->
        <v-data-table
          :headers="headers"
          :items="filtrarCursos"
          :items-per-page="5"
          class="elevation-1"
        >
          <template #item.acciones="{ item }">
            <v-icon small class="mr-2" @click="editarCurso(item)">mdi-pencil</v-icon>
            <v-icon small class="mr-2" color="green" @click="verEstudiantes(item)">mdi-account-school</v-icon>
            <v-icon small class="mr-2" color="green" @click="verProfesores(item)">mdi-human-male-board</v-icon>
            <v-icon small color="red" @click="abrirDialogoEliminar(item)">mdi-delete</v-icon>
          </template>
        </v-data-table>
      </v-card-text>
    </v-card>

    <!-- Diálogo para Crear/Editar -->
    <v-dialog v-model="dialog" max-width="500px">
      <v-card>
        <v-card-title>
          {{ dialogModo === 'crear' ? 'Crear Curso' : 'Editar Curso' }}
        </v-card-title>
        <v-card-text>
          <v-form ref="form">
            <v-text-field
              v-model="formCurso.claveCurso"
              label="Clave del Curso"
              required
              type="number"
              :disabled="dialogModo === 'editar'"
            ></v-text-field>
            <v-text-field
              v-model="formCurso.nombreCurso"
              label="Nombre del Curso"
              required
            ></v-text-field>
            <v-text-field
              v-model="formCurso.creditos"
              label="Créditos"
              required
              type="number"
              :disabled="dialogModo === 'editar'"
            ></v-text-field>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-btn text @click="dialog = false">Cancelar</v-btn>
          <v-btn color="primary" @click="guardarCurso">Guardar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Diálogo de Confirmación para Eliminar -->
    <v-dialog v-model="deleteDialog" max-width="400">
      <v-card>
        <v-card-title class="text-h6">¿Borrar?</v-card-title>
        <v-card-text>
          ¿Está seguro de que desea eliminar el curso: <strong>{{ cursoAEliminar?.nombreCurso }}</strong>?
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn text @click="cerrarDialogoEliminar">Cancelar</v-btn>
          <v-btn color="red" text @click="confirmarEliminarCurso">Borrar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!--Diálogo de estudiantes asociados-->
    <v-dialog v-model="estudiantesDialog" max-width="600px">
      <v-card>
        <v-card-title>
          Estudiantes del curso {{ cursoSeleccionado?.claveCurso }}
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

        <!--Diálogo de profesores asociados-->
        <v-dialog v-model="profesoresDialog" max-width="600px">
      <v-card>
        <v-card-title>
          Profesores del curso {{ estudianteSeleccionado?.claveCurso }}
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
  name: "TablaCursos",
  data() {
    return {
      cursos: [],
      filtroClaveCurso: "", // Campo para la búsqueda por clave del curso
      headers: [
        { title: "Clave del Curso", key: "claveCurso", sortable: false },
        { title: "Nombre del Curso", key: "nombreCurso", sortable: false },
        { title: "Créditos", key: "creditos", sortable: false },
        { title: "Acciones", key: "acciones", sortable: false },
      ],
      dialog: false,
      dialogModo: "crear", // 'crear' o 'editar'
      formCurso: {
        nombreCurso: "",
        claveCurso: "",
        creditos: "",
      },
      estudiantes: [],
      estudiantesHeaders: [
        { title: "Matrícula", key: "matricula", sortable: false },
        { title: "Nombre", key: "nombre", sortable: false }
      ],
      profesores: [],
      profesorHeaders: [
        { title: "Número de Empleado", key: "numEmpleado", sortable: false },
        { title: "Nombre del Profesor", key: "nombre", sortable: false },
      ],
      profesoresDialog: false,
      estudiantesDialog: false,
      deleteDialog: false,
      cursoAEliminar: null, // Para almacenar el curso a eliminar
    };
  },
  computed: {
    filtrarCursos() {
      const filtro = this.filtroClaveCurso.toString().toLowerCase();
      return this.filtroClaveCurso
        ? this.cursos.filter((curso) =>
            String(curso.claveCurso).toLowerCase().includes(filtro)
          )
        : this.cursos;
    },
  },
  methods: {
    async cargarCursos() {
      try {
        const response = await axios.get("https://localhost:5000/getAllCursos");
        this.cursos = response.data.data;
      } catch (error) {
        console.error("Error al cargar cursos:", error);
      }
    },
    abrirDialogoCrear() {
      this.dialogModo = "crear";
      this.formCurso = { nombreCurso: "", claveCurso: "", creditos: "" };
      this.dialog = true;
    },
    editarCurso(curso) {
      this.dialogModo = "editar";
      this.formCurso = { ...curso };
      this.dialog = true;
    },
    async guardarCurso() {
      try {
        if (this.dialogModo === "crear") {
          await axios.post("https://localhost:5000/createCurso", this.formCurso);
        } else {
          await axios.put(
            `https://localhost:5000/updateCurso/${this.formCurso.claveCurso}`,
            this.formCurso
          );
        }
        this.dialog = false;
        this.cargarCursos();
      } catch (error) {
        console.error("Error al guardar curso:", error);
      }
    },
    abrirDialogoEliminar(curso) {
      this.cursoAEliminar = curso;
      this.deleteDialog = true;
    },
    cerrarDialogoEliminar() {
      this.deleteDialog = false;
      this.cursoAEliminar = null;
    },
    async confirmarEliminarCurso() {
      try {
        if (this.cursoAEliminar) {
          await axios.delete(`https://localhost:5000/deleteCurso/${this.cursoAEliminar.claveCurso}`);
          this.cargarCursos();
          this.cerrarDialogoEliminar();
        }
      } catch (error) {
        console.error("Error al eliminar curso:", error);
      } finally {
        this.cerrarDialogoEliminar();
      }
    },
    async verEstudiantes(curso) {
      this.cursoSeleccionado = curso;
      try {
        const response = await axios.get(`https://localhost:5000/getEstudiantesCurso/${curso.claveCurso}`);

        if (response.data && response.data.estudiantes) {
          this.estudiantes = response.data.estudiantes;
        } else {
          this.estudiantes = [];
        }
        this.estudiantesDialog = true;
      } catch (error) {
        console.error("Error al cargar los estudiantes:", error);
        this.$toast.error("No se pudieron cargar los estudiantes del curso.");
      }
    },
    async verProfesores(curso) {
      this.cursoSeleccionado = curso;
      try {
        const response = await axios.get(`https://localhost:5000/getProfesoresCurso/${curso.claveCurso}`);

        if (response.data && response.data.profesores) {
          this.profesores = response.data.profesores;
        } else {
          this.profesores = [];
        }
        this.profesoresDialog = true;
      } catch (error) {
        console.error("Error al cargar los profesores:", error);
        this.$toast.error("No se pudieron cargar los profesores del curso.");
      }
    }
  },
  mounted() {
    this.cargarCursos();
  },
};
</script>

<style scoped>
.elevation-1 {
  margin-top: 20px;
}
</style>

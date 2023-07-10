<template>
  <div class="home m-5">
    <div class="card">
      <div class="card-body">
        <div class="mb-4 d-flex justify-content-end">
          <a class="btn btn-sm btn-primary" @click="nuevo()">Nuevo Registro</a>
        </div>
        <div class="table-responsive">
          <table class="table table-hover table-bordered">
            <thead class="table-dark">
              <tr>
                <th>#</th>
                <th>Nombre</th>
                <th>Apellidos</th>
                <th>Dirección</th>
                <th>Telefono</th>
                <th>Acciones</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="empleado in lista_empleados" :key="empleado.id">
                <td>{{ empleado.id }}</td>
                <td>{{ empleado.nombre }}</td>
                <td>{{ empleado.apellidos }}</td>
                <td>{{ empleado.direccion }}</td>
                <td>{{ empleado.telefono }}</td>
                <td>
                  <div class="btn-group" role="group" aria-label="Button group">
                    <a href="#" class="btn btn-sm btn-outline-success" @click.prevent="editar(empleado.id)">Editar</a>
                    <a href="#" class="btn btn-sm btn-outline-danger" @click.prevent="eliminar(empleado.id)">Eliminar</a>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
        <CrearEmpleado v-if="modal_create" @close="registrar()" @cerrarModal="modal_create = false" />
        <EditarEmpleado v-if="modal_edit" @cerrarModalEdit="modal_edit = false" :idEmpleado="id_empleado"
          @close="actualizar()" />
      </div>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import axios from "axios";
import CrearEmpleado from "@/components/crear.vue";
import EditarEmpleado from "@/components/editar.vue";
export default {
  name: 'HomeView',
  components: {
    CrearEmpleado,
    EditarEmpleado,
  },
  data: function () {
    return {
      modal_create: false,
      modal_edit: false,
      lista_empleados: null,
      id_empleado: null,
    }
  },
  mounted: function () {
    this.listar();
  },
  methods: {
    nuevo() {
      this.modal_create = true;
    },

    registrar() {
      this.modal_create = false;
      this.listar();
    },

    listar() {
      axios.get('http://127.0.0.1:8000/api/empleados').then(data => {
        this.lista_empleados = data.data;
      })
    },

    editar(id) {
      this.id_empleado = id;
      this.modal_edit = true;
    },

    actualizar() {
      this.modal_edit = false;
      this.listar();
    },
    async eliminar(id) {
      if (confirm('¿Estás seguro de que deseas realizar esta acción?')) {
        try {
          const { status, data } = await axios.delete(`http://127.0.0.1:8000/api/empleados/delete/${id}`);
          if (status === 200) {
            alert(data.message);
            this.listar();
          }
        } catch (error) {
          console.log(error);
        }
      }
    }
  }

}
</script>

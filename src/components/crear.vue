<template>
    <b-modal v-model="showModal" @hidden="$emit('cerrarModal')" id="modal-1" title="REGISTRAR EMPLEADO" no-close-on-esc
        no-close-on-backdrop>
        <form>
            <div class="form-group">
                <label for="name">Nombre</label>
                <b-form-input v-model="empleados.nombre" placeholder="Ingrese su nombre"></b-form-input>
            </div>
            <div class="form-group mt-2">
                <label class="mb-1" for="name">Apellidos</label>
                <b-form-input v-model="empleados.apellidos" placeholder="Ingrese sus apellidos"></b-form-input>
            </div>
            <div class="form-group mt-2">
                <label class="mb-1" for="name">Direccion</label>
                <b-form-input v-model="empleados.direccion" placeholder="Ingrese su dirección"></b-form-input>
            </div>
            <div class="form-group mt-2">
                <label class="mb-1" for="name">Telefono</label>
                <b-form-input type="number" v-model="empleados.telefono" placeholder="Ingrese su telefono"></b-form-input>
            </div>
            <hr>
            <div class="mt-4">
                REGISTRAR PAGOS
                <div>
                    <div class="form-group mt-2">
                        <label class="mb-1" for="name">Fecha</label>
                        <b-form-input v-model="newPago.fecha" type="date" placeholder="Ingrese su telefono"></b-form-input>
                    </div>
                    <div class="form-group mt-2">
                        <label class="mb-1" for="name">Nro. Transacción</label>
                        <b-form-input v-model="newPago.transaccion" placeholder="Ingrese su telefono"></b-form-input>
                    </div>
                    <div class="form-group mt-2">
                        <label class="mb-1" for="name">Importe</label>
                        <b-form-input v-model="newPago.importe" type="number" step="0.01"
                            placeholder="Ingrese su telefono"></b-form-input>
                    </div>
                </div>
                <div class="mt-2 mb-2 d-flex justify-content-end">
                    <button class="btn btn-primary"
                        :disabled="Object.values(newPago).some(value => value.length === 0) || newPago.importe <= 0"
                        @click="agregarPago()">ADD</button>
                </div>
                <div v-if="empleados.json.length > 0" class="table-responsive">
                    <table class="table table-hover">
                        <thead class="table-dark">
                            <tr>
                                <th>#</th>
                                <th>Fecha</th>
                                <th>Nro. Transacción</th>
                                <th>Importe</th>
                                <th></th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(item, index) in empleados.json" :key="index">
                                <td>{{ index + 1 }}</td>
                                <td>{{ item.fecha }}</td>
                                <td>{{ item.transaccion }}</td>
                                <td>{{ item.importe }}</td>
                                <td>
                                    <div class="btn-group" role="group" aria-label="Button group">
                                        <a class="btn btn-sm btn-danger" @click="eliminar_pago(index)">Eliminar</a>
                                    </div>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
                <p v-else>No hay ningun pago registrado</p>
            </div>
        </form>
        <template #modal-footer="{ cancel }">
            <!-- Emulate built in modal footer ok and cancel button actions -->
            <b-button size="sm" variant="danger" @click="cancel()">
                Cancel
            </b-button>
            <b-button size="sm" :disabled="Object.values(empleados).some(value => value.length === 0)" variant="success"
                @click="registrar()">
                Registrar
            </b-button>
        </template>
    </b-modal>
</template>
<script>
import axios from 'axios';
export default {
    name: "crearComponet",
    data: function () {
        return {
            showModal: true,
            empleados: {
                nombre: '',
                apellidos: '',
                direccion: '',
                telefono: '',
                json: [],
            },
            newPago: {
                fecha: '',
                transaccion: '',
                importe: ''
            }
        }
    },
    methods: {
        agregarPago() {
            this.empleados.json.push(this.newPago);
            this.newPago = {
                fecha: '',
                transaccion: '',
                importe: ''
            }
        },
        eliminar_pago(id) {
            this.empleados.json.splice(id, 1);
        },

        async registrar() {
            try {
                const { status, data } = await axios.post('http://127.0.0.1:8000/api/empleados/store', this.empleados);
                if (status === 200) {
                    this.$emit('close');
                    alert(data.message);
                }
            } catch (error) {
                console.log(error.response.data.message);
            }
        }
    }
}
</script>
<style scoped></style>
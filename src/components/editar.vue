<template>
    <b-modal v-model="showModal" @hidden="$emit('cerrarModalEdit')" id="modal-2" title="EDITAR EMPLEADO"
        dialog-class="modal-xl" no-close-on-esc no-close-on-backdrop>
        <form>
            <div class="row">
                <div class="form-group col-md-4">
                    <label for="name">Nombre</label>
                    <b-form-input v-model="empleados.nombre" placeholder="Ingrese su nombre"></b-form-input>
                </div>
                <div class="form-group col-md-4">
                    <label class="mb-1" for="name">Apellidos</label>
                    <b-form-input v-model="empleados.apellidos" placeholder="Ingrese sus apellidos"></b-form-input>
                </div>
                <div class="form-group col-md-4">
                    <label class="mb-1" for="name">Direccion</label>
                    <b-form-input v-model="empleados.direccion" placeholder="Ingrese su dirección"></b-form-input>
                </div>
                <div class="form-group col-md-4">
                    <label class="mb-1" for="name">Telefono</label>
                    <b-form-input type="number" v-model="empleados.telefono"
                        placeholder="Ingrese su telefono"></b-form-input>
                </div>
            </div>
            <hr>
            <div class="mt-4">
                <p v-if="edit_json === false">REGISTRAR PAGOS</p>
                <p v-else>ACTUALIZAR PAGO</p>
                <div class="row">
                    <div class="form-group col-md-4">
                        <label class="mb-1" for="name">Fecha</label>
                        <b-form-input v-model="newPago.fecha" type="date" placeholder="Ingrese la fecha"></b-form-input>
                    </div>
                    <div class="form-group col-md-4">
                        <label class="mb-1" for="name">Nro. Transacción</label>
                        <b-form-input v-model="newPago.transaccion" placeholder="Ingrese Nro. Transacción"></b-form-input>
                    </div>
                    <div class="form-group col-md-4">
                        <label class="mb-1" for="name">Importe</label>
                        <b-form-input v-model="newPago.importe" type="number" step="0.01"
                            placeholder="Ingrese Importe"></b-form-input>
                    </div>
                </div>
                <div v-if="edit_json === false" class="mt-3 mb-3 d-flex justify-content-end">
                    <button class="btn btn-primary btn-sm"
                        :disabled="Object.values(newPago).some(value => value.length === 0) || newPago.importe <= 0"
                        @click="agregarPago()">AGREGAR</button>
                </div>
                <div v-else class="mt-3 mb-3 d-flex justify-content-end gap-3">
                    <button class="btn btn-danger btn-sm" @click="cancelar_pago()">CANCELAR
                    </button>
                    <button class="btn btn-success btn-sm"
                        :disabled="Object.values(newPago).some(value => value.length === 0) || newPago.importe <= 0"
                        @click="actualizar_pago()">
                        ACTUALIZAR
                    </button>
                </div>

                <div v-if="empleados.json.length > 0" class="table-responsive">
                    <table class="table table-hover">
                        <thead class="table-dark">
                            <tr>
                                <th>#</th>
                                <th>Estado</th>
                                <th>Fecha</th>
                                <th>Nro. Transacción</th>
                                <th>Importe</th>
                                <th></th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(pago, index) in empleados.json" :key="index">
                                <td>{{ index + 1 }}</td>
                                <td>
                                    <span v-if="pago.id > 0">Existente</span>
                                    <span v-else>New</span>
                                </td>
                                <td>
                                    {{ pago.fecha }}
                                </td>
                                <td>
                                    {{ pago.transaccion }}
                                </td>
                                <td>
                                    {{ pago.importe }}
                                </td>
                                <td>
                                    <div class="btn-group" role="group" aria-label="Button group">
                                        <a class="btn btn-sm btn-success" @click="show_pago(index)">Editar</a>
                                        <a class="btn btn-sm btn-danger" @click="eliminar_pago(index, pago.id)">Eliminar</a>
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
            <b-button size="sm" :disabled="validar_datos_vacios()" variant="success" @click="actualizar()">
                Actualizar
            </b-button>
        </template>
    </b-modal>
</template>
<script>
import axios from 'axios';
export default {
    name: "UpdateComponent",
    props: {
        idEmpleado: {
            type: Number,
            required: true,
        }
    },
    data: function () {
        return {
            showModal: true,
            empleados: {
                nombre: '',
                apellidos: '',
                direccion: '',
                telefono: '',
                json: [],
                jsonDelete: [],
            },
            newPago: {
                id: 0,
                fecha: '',
                transaccion: '',
                importe: ''
            },
            edit_json: false,
            indice_json: null,
        }
    },
    mounted: function () {
        this.showData();
    },
    methods: {
        async showData() {
            try {
                const { status, data } = await axios.get('http://127.0.0.1:8000/api/empleados/' + this.idEmpleado);
                if (status === 200) {
                    this.empleados.nombre = data[0].nombre;
                    this.empleados.apellidos = data[0].apellidos;
                    this.empleados.direccion = data[0].direccion;
                    this.empleados.telefono = data[0].telefono;

                    const objetos = JSON.parse(data[0].pagos);
                    this.empleados.json.push(...objetos);
                }
            } catch (error) {
                this.showModal = false;
                alert(error.message);
            }
        },
        agregarPago() {
            this.empleados.json.push(this.newPago);
            this.newPago = {
                id: 0,
                fecha: '',
                transaccion: '',
                importe: ''
            };
        },
        show_pago(index) {
            const data = this.empleados.json[index];
            this.newPago = {
                id: data.id,
                fecha: data.fecha,
                transaccion: data.transaccion,
                importe: data.importe
            };
            this.indice_json = index;
            this.edit_json = true;
        },
        cancelar_pago() {
            this.newPago = {
                id: 0,
                fecha: '',
                transaccion: '',
                importe: ''
            };
            this.edit_json = false;
        },
        actualizar_pago() {
            const objeto = this.empleados.json[this.indice_json];
            objeto.fecha = this.newPago.fecha;
            objeto.transaccion = this.newPago.transaccion;
            objeto.importe = this.newPago.importe;

            this.indice_json = null;
            this.edit_json = false;
            this.newPago = {
                id: 0,
                fecha: '',
                transaccion: '',
                importe: ''
            };
        },
        eliminar_pago(index, id) {
            if (id > 0) {
                this.empleados.jsonDelete.push({ 'id': id });
            }
            this.empleados.json.splice(index, 1);
        },
        validar_datos_vacios() {
            const copiaEmpleados = { ...this.empleados };
            delete copiaEmpleados.jsonDelete;
            return Object.values(copiaEmpleados).some(value => value.length === 0)
        },
        async actualizar() {
            try {
                const { status, data } = await axios.put(`http://127.0.0.1:8000/api/empleados/edit/${this.idEmpleado}`, this.empleados);
                if (status === 200) {
                    alert(data.message);
                    this.$emit('close');
                }
            } catch (error) {
                console.log(error);
            }
        }
    }
}
</script>
<style scoped></style>
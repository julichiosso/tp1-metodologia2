<template>
  <div class="registro-container">
    <h1 class="titulo">Registro de Clientes</h1>

    <form @submit.prevent="registrarCliente" class="formulario">
      <div class="input-group">
        <label for="nombre">Nombre y Apellido:</label>
        <input
          type="text"
          id="nombre"
          v-model="datosCliente.nombre"
          required
          placeholder="Ej: Juan Pérez"
          :disabled="loading"
        />
      </div>

      <div class="input-group">
        <label for="email">Email:</label>
        <input
          type="email"
          id="email"
          v-model="datosCliente.email"
          required
          placeholder="ejemplo@correo.com"
          :disabled="loading"
        />
      </div>

      <button type="submit" class="btn-registrar" :disabled="loading">
        {{ loading ? 'Registrando...' : 'Registrar' }}
      </button>
    </form>

    <!-- Mensajes -->
     <div v-if="mensaje" class="mensaje-exito">{{ mensaje }}</div>
     <div v-if="error" class="mensaje-error">{{ error }}</div>

    <div class="tabla-container">
      <div class="tabla-header">
        <h2>Clientes</h2>
        <button @click="obtenerClientes" class="btn-listar" :disabled="loading">
          Actualizar
        </button>
      </div>

      <table class="tabla">
        <thead>
          <tr>
            <th>ID</th>
            <th>Nombre</th>
            <th>Email</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="cliente in listaClientes"
            :key="cliente.id"
            @click="clienteSeleccionado = cliente"
            :class="{ seleccionado: clienteSeleccionado?.id === cliente.id }"
            tabindex="0"
            @keydown.enter.prevent="modificarCliente(cliente)"
          >
            <td>{{ cliente.id }}</td>
            <td>{{ cliente.nombre }}</td>
            <td>{{ cliente.email }}</td>
            <td class="acciones">
              <button @click.stop="modificarCliente(cliente)" class="btn-accion azul" title="Editar">
                <svg xmlns="http://www.w3.org/2000/svg" class="icon" viewBox="0 0 24 24" fill="none">
                  <path d="M4 21h17M12.5 6.5l5 5-9 9H4v-4.5l8.5-8.5zM17 4a1.5 1.5 0 0 1 2 2l-1.5 1.5-2-2L17 4Z" stroke="white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
              </button>
                <button @click.stop="eliminarCliente(cliente)" class="btn-accion rojo" title="Eliminar">
                <svg xmlns="http://www.w3.org/2000/svg" class="icon" viewBox="0 0 24 24" fill="none">
                  <path d="M6 7h12M10 11v6M14 11v6M5 7l1 12a2 2 0 0 0 2 2h8a2 2 0 0 0 2-2l1-12M9 7V5a1 1 0 0 1 1-1h4a1 1 0 0 1 1 1v2" stroke="white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
              </button>
            </td>
          </tr>

          <tr v-if="listaClientes.length === 0">
            <td colspan="4" class="sin-datos">No hay clientes registrados.</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Popup Modificar Cliente -->
    <div v-if="mostrarPopup" class="overlay" @click.self="cerrarPopup">
      <div class="popup">
        <h3>Modificar Cliente</h3>
        <form @submit.prevent="guardarCambios" class="formulario-popup">
          <label for="edit-nombre">Nombre y Apellido:</label>
          <input
            id="edit-nombre"
            v-model="clienteEditado.nombre"
            type="text"
            required
            placeholder="Nombre y Apellido"
            :disabled="loading"
          />

          <label for="edit-email">Email:</label>
          <input
            id="edit-email"
            v-model="clienteEditado.email"
            type="email"
            required
            placeholder="ejemplo@correo.com"
            :disabled="loading"
          />

          <div class="botones-popup">
            <button type="submit" :disabled="loading" class="btn-registrar">
              Guardar
            </button>
            <button type="button" @click="cerrarPopup" class="btn-cancelar">
              Cancelar
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Confirmación Eliminación -->
    <!-- Modal de Confirmación -->
<div v-if="mostrarConfirmacion" class="overlay">
  <div class="popup">
    <h3>¿Eliminar Cliente?</h3>
    <p>Estás por eliminar <strong>{{ clienteAEliminar?.nombre }}</strong>. Esta acción no se puede deshacer.</p>
    <div class="botones-popup">
      <button @click="mostrarConfirmacion = false" class="btn-cancelar">Cancelar</button>
      <button @click="confirmarEliminacion" class="btn-accion rojo">Eliminar</button>
    </div>
  </div>
</div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue'
import axios from 'axios'

const mensaje = ref('')
const error = ref('')
const loading = ref(false)

const mostrarPopup = ref(false)
const mostrarConfirmacion = ref(false)

const clienteAEliminar = ref(null)
const clienteEditado = ref({ nombre: '', email: '', id: null })
const clienteSeleccionado = ref(null)
const listaClientes = ref([])

const datosCliente = ref({
  nombre: '',
  email: ''
})

const mostrarMensajeTemporal = async (tipo, texto) => {
  if (tipo === 'exito') mensaje.value = texto
  if (tipo === 'error') error.value = texto

  await nextTick()

  setTimeout(() => {
    if (tipo === 'exito') mensaje.value = ''
    if (tipo === 'error') error.value = ''
  }, 3500)
}

const obtenerClientes = async () => {
  try {
    loading.value = true
    const res = await axios.get('http://localhost:5272/api/Cliente')
    listaClientes.value = res.data
  } catch (err) {
    console.error('Error al obtener clientes:', err)
    mostrarMensajeTemporal('error', 'No se pudieron cargar los clientes.')
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  obtenerClientes()
})

const registrarCliente = async () => {
  mensaje.value = ''
  error.value = ''

  if (!datosCliente.value.nombre.trim() || !datosCliente.value.email.trim()) {
    mostrarMensajeTemporal('error', 'Todos los campos son obligatorios.')
    return
  }

  if (!datosCliente.value.email.includes('@')) {
    mostrarMensajeTemporal('error', 'El email que ingresó es inválido.')
    return
  }

  try {
    loading.value = true

    const cliente = {
      nombre: datosCliente.value.nombre.trim(),
      email: datosCliente.value.email.trim(),
      fechaRegistro: new Date().toISOString()
    }

    const response = await axios.post('http://localhost:5272/api/Cliente', cliente)

    const clienteNuevo = {
      ...response.data,
      esNuevo: true,
      fechaRegistro: new Date(response.data.fechaRegistro).toISOString()
    }

    listaClientes.value.push(clienteNuevo)

    setTimeout(() => {
      clienteNuevo.esNuevo = false
      listaClientes.value.sort((a, b) => new Date(b.fechaRegistro) - new Date(a.fechaRegistro))
    }, 5000)

    mostrarMensajeTemporal('exito', '✅ Cliente registrado correctamente')

    datosCliente.value = {
      nombre: '',
      email: ''
    }
  } catch (err) {
    console.error(err)
    mostrarMensajeTemporal('error', 'Error al guardar en SQL: ' + err.message)
  } finally {
    loading.value = false
  }
}

const eliminarCliente = (cliente) => {
      if (!cliente) {
        mostrarMensajeTemporal('error', 'Seleccioná un cliente primero.')
      return
  }
  clienteAEliminar.value = cliente
  mostrarConfirmacion.value = true
}

const confirmarEliminacion = async () => {
  try {
    loading.value = true
    await axios.delete(`http://localhost:5272/api/Cliente/${clienteAEliminar.value.id}`)
    listaClientes.value = listaClientes.value.filter(c => c.id !== clienteAEliminar.value.id)
    if (clienteSeleccionado.value?.id === clienteAEliminar.value.id) clienteSeleccionado.value = null
    mostrarMensajeTemporal('exito', 'Cliente eliminado con éxito.')
  } catch (err) {
    mostrarMensajeTemporal('error', 'Error al eliminar cliente.')
  } finally {
    loading.value = false
    mostrarConfirmacion.value = false
  }
}

const modificarCliente = (cliente) => {
  if (!cliente) return mostrarMensajeTemporal('error', 'Seleccioná un cliente para modificar.')
  clienteEditado.value = { ...cliente }
  mostrarPopup.value = true
}

const cerrarPopup = () => {
  mostrarPopup.value = false
  clienteEditado.value = { nombre: '', email: '', id: null }
}

const guardarCambios = async () => {
  if (!clienteEditado.value.nombre.trim() || !clienteEditado.value.email.trim()) {
    mostrarMensajeTemporal('error', 'Todos los campos son obligatorios.')
    return
  }
  if (!clienteEditado.value.email.includes('@')) {
    mostrarMensajeTemporal('error', 'El email que ingresó es inválido.')
    return
  }
  try {
    loading.value = true
    await axios.put(`http://localhost:5272/api/Cliente/${clienteEditado.value.id}`, clienteEditado.value)
    const index = listaClientes.value.findIndex(c => c.id === clienteEditado.value.id)
    if (index !== -1) listaClientes.value[index] = { ...clienteEditado.value }
    mostrarMensajeTemporal('exito', 'Cliente modificado con éxito.')
    cerrarPopup()
  } catch (err) {
    mostrarMensajeTemporal('error', 'Error al guardar cambios.')
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
/* Contenedor principal */
.registro-container {
  background: linear-gradient(145deg, #0d2c45, #092536);
  padding: 2.5rem 3rem;
  border-radius: 24px;
  color: #f0f4f8;
  max-width: 950px;
  margin: 40px auto;
  box-shadow: 0 25px 45px rgba(0, 0, 0, 0.3);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  animation: fadeIn 0.6s ease;
  user-select: none;
}

/* Título */
.titulo {
  text-align: center;
  font-size: 2.4rem;
  margin-bottom: 1.5rem;
  font-weight: 700;
  letter-spacing: 1px;
  text-shadow: 0 0 6px #0f2133;
}

/* Formulario */
.formulario {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
  background: #123a5a;
  padding: 4%;

  border-radius: 16px;
  margin-bottom: 2.5rem;
  box-shadow: inset 0 0 15px #0f2844;
  transition: box-shadow 0.3s ease;
}

.formulario:hover {
  box-shadow: inset 0 0 22px #161616;
}

/* Labels */
.input-group label {
  display: block;
  margin-bottom: 0.6rem;
  font-weight: 600;
  color: #a8c0ff;
  user-select: text;
}

/* Inputs */
.input-group input {
  padding: 0.65rem 1rem;
  width: 96%;
  border: none;
  border-radius: 12px;
  outline: none;
  font-size: 1.1rem;
  color: #0d2c45;
  background-color: #c4d8ff;
  transition: box-shadow 0.3s ease;
  user-select: text;
}

.input-group input:focus {
  box-shadow: 0 0 10px #090d11;
}

/* Botón registrar */
.btn-registrar {
  background: #1e90ff;
  color: white;
  border: none;
  padding: 0.9rem;
  border-radius: 14px;
  cursor: pointer;
  font-weight: 700;
  font-size: 1.15rem;
  box-shadow: 0 0 5px #1e90ffcc;
  transition: background-color 0.3s ease, box-shadow 0.3s ease;
  user-select: none;
}

.btn-registrar:hover:not(:disabled) {
  background: #0077dd;
  box-shadow: 0 0 30px #0077ddcc;
}

.btn-registrar:disabled {
  cursor: not-allowed;
  background: #95c2ff;
  box-shadow: none;
}

/* Mensajes */
.mensaje-exito,
.mensaje-error {
  text-align: center;
  margin-bottom: 1rem;
  font-weight: 600;
  padding: 0.7rem 1rem;
  border-radius: 12px;
  user-select: none;
  max-width: 700px;
  margin-left: auto;
  margin-right: auto;
  animation: fadeIn 0.6s ease;
}

/*.mensaje-exito {
  color: #1a472a;
  background-color: #a4d4a4;
  border: 2px solid #7eaf80;
  box-shadow: 0 0 8px #4caf50aa;
}*/

.mensaje-exito{
  background-color: #21ae60d3;
  /*border: 2px solid #383838;*/
  color: #fafafa;
  text-align: center;
  margin-top: 1rem;
  padding: 0.7rem 1rem;
  border-radius: 12px;
  animation: fadeIn 0.6s ease;
}
.mensaje-error {
  background-color: #d00020;
  color: #fafafa;
  text-align: center;
  margin-top: 1rem;
  padding: 0.7rem 1rem;
  border-radius: 12px;
  animation: fadeIn 0.6s ease;
}

/*.mensaje-error {
  color: #6f1212;
  background-color: #f9b4b4;
  border: 2px solid #e74c3c;
  box-shadow: 0 0 8px #e74c3caa;
}*/

/* Tabla container */
.tabla-container {
  background: #ffffff;
  border-radius: 20px;
  padding: 2rem;
  color: #1a1a1a;
  animation: slideIn 0.5s ease;
  box-shadow: 0 12px 28px rgba(0, 0, 0, 0.12);
}

/* Header tabla */
.tabla-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.25rem;
}

/* Botón actualizar lista */
.btn-listar {
  background: #27ae60;
  color: white;
  padding: 0.7rem 1.3rem;
  border: none;
  border-radius: 14px;
  cursor: pointer;
  font-weight: 700;
  font-size: 1rem;
  
  transition: background-color 0.3s ease, box-shadow 0.3s ease;
  user-select: none;
}

.btn-listar:hover:not(:disabled) {
  background: #1e8449;
  box-shadow: 0 0 10px #1e844999;
}

.btn-listar:disabled {
  cursor: not-allowed;
  background: #94d18b;
  box-shadow: none;
}

/* Tabla */
.tabla {
  width: 100%;
  border-collapse: collapse;
  font-size: 1rem;
  user-select: none;
}

.tabla th,
.tabla td {
  padding: 1rem 1.2rem;
  text-align: left;
  border-bottom: 1px solid #ddd;
  vertical-align: middle;
}

.tabla th {
  background-color: #f0f4f8;
  color: #34495e;
  font-weight: 700;
  letter-spacing: 0.05em;
}
tr:hover{
  background-color: #d0eaff;
  cursor: pointer;
}
/* Fila seleccionada */
.seleccionado {
  background-color: #d0eaff;
  font-weight: 600;
  box-shadow: inset 4px 0 0 0 #1e90ff;
}

/* Botones acciones */
.acciones {
  display: flex;
  gap: 0.7rem;
}

.btn-accion {
  padding: 0.5rem 0.75rem;
  border: none;
  border-radius: 10px;
  color: white;
  font-size: 1.2rem;
  cursor: pointer;
  transition: background-color 0.3s ease, box-shadow 0.3s ease;
  user-select: none;
}

.btn-accion.azul {
  background-color: #3498db;
}

.btn-accion.azul:hover:not(:disabled) {
  background-color: #2980b9;
  box-shadow: 0 0 20px #2980b9cc;
}

.btn-accion.rojo {
  background-color: #e74c3c;
}

.btn-accion.rojo:hover:not(:disabled) {
  background-color: #c0392b;
  box-shadow: 0 0 20px #c0392bcc;
}

.btn-accion:disabled {
  cursor: not-allowed;
  opacity: 0.65;
  box-shadow: none;
}

/* Sin datos */
.sin-datos {
  text-align: center;
  font-style: italic;
  color: #666;
  padding: 1.2rem 0;
}

/* Popup (overlay y contenido) */
.overlay {
  position: fixed;
  inset: 0;
  background-color: rgba(0, 0, 0, 0.45);
  backdrop-filter: blur(2px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1500;
  animation: fadeIn 0.25s ease;
}




/* Formulario popup */
.formulario-popup {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  text-align: left;
}

.formulario-popup label {
  font-weight: 600;
  color: #34495e;
}

.formulario-popup input {
  padding: 0.6rem 1rem;
  border-radius: 10px;
  border: 1.5px solid #a8c0ff;
  outline: none;
  font-size: 1.05rem;
  transition: border-color 0.3s ease;
  color: #123a5a;
}

.formulario-popup input:focus {
  border-color: #1e90ff;
  box-shadow: 0 0 8px #1e90ffaa;
}

/* Botones en popup */
.botones-popup {
  display: flex;
  justify-content: flex-end;
  gap: 1rem;
  margin-top: 1.2rem;
}

.btn-cancelar {
  background: #ccc;
  color: #555;
  border: none;
  padding: 0.7rem 1.3rem;
  border-radius: 12px;
  cursor: pointer;
  font-weight: 600;
  transition: background-color 0.3s ease;
  user-select: none;
}

.btn-cancelar:hover:not(:disabled) {
  background: #aaa;
}

.btn-cancelar:disabled {
  cursor: not-allowed;
  opacity: 0.6;
}

/* Animaciones */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(50px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.icon {
  width: 20px;
  height: 20px;
  pointer-events: none;
}

.popup p {
  margin-bottom: 1rem;
  font-size: 1rem;
  color: #555;
  line-height: 1.4;
}
.popup {
  background-color: #fefefe;
  padding: 2rem;
  border-radius: 20px;
  max-width: 400px;
  width: 90%;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
  font-family: 'Segoe UI', sans-serif;
  text-align: center;
  animation: aparecer 0.25s ease-out;
}

.popup h3 {
  margin-bottom: 1.5rem;
  font-size: 1.6rem;
  font-weight: 600;
  color: #2c3e50;
}

.popup label {
  display: block;
  text-align: left;
  margin-bottom: 0.3rem;
  color: #34495e;
  font-weight: 500;
}

.popup input {
  width: 90%;
  padding: 0.6rem 1rem;
  border: 1.5px solid #d0d7de;
  border-radius: 10px;
  margin-bottom: 1.2rem;
  font-size: 1rem;
  outline: none;
  transition: 0.2s;
}

.popup input:focus {
  border-color: #3498db;
  box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.2);
}

.botones-popup {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
  margin-top: 1rem;
}

.botones-popup button {
  flex: 1;
  padding: 0.6rem 0;
  border-radius: 12px;
  font-weight: 600;
  font-size: 1rem;
  transition: 0.2s;
  cursor: pointer;
}

.btn-accion {
  background-color: #3498db;
  color: white;
  border: none;
}

.btn-accion:hover {
  background-color: #2980b9;
}

.btn-cancelar {
  background-color: #e0e0e0;
  color: #555;
  border: none;
}

.btn-cancelar:hover {
  background-color: #cacaca;
}

@keyframes aparecer {
  from {
    opacity: 0;
    transform: scale(0.96);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}


</style>
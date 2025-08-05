<template>
  <div class="historial-container">
    <h1 class="titulo">Historial de Movimientos</h1>
    <p style="font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;">Compras y ventas realizadas por los clientes</p>

    <div class="formulario">
      <label style="font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;" for="cliente" class="label">Seleccione el cliente a buscar</label>
      <select id="cliente" v-model="clienteSeleccionado" class="select">
        <option disabled value=""></option>
        <option v-for="cliente in clientes" :key="cliente.id" :value="cliente.id">
          {{ cliente.nombre }}
        </option>
      </select>

      <p style="font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;">Clientes cargados: {{ clientes.length }}</p>
      <p style="font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;">Movimientos cargados: {{ movimientos.length }}</p>

      <button style="font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;" @click="obtenerMovimientos" class="select">Buscar</button>
    </div>

    <div v-if="movimientos.length > 0" class="tarjetas-contenedor">
      <div
        v-for="mov in movimientos"
        :key="mov.id"
        :class="['tarjeta', mov.accion.toLowerCase()]"
        @click="mostrarDetalle(mov)"
      >
        <div class="contenido-tarjeta">
          <div class="cripto">{{ mov.cryptoCode }}</div>
          <div class="detalle">Criptomoneda: {{ getCryptoName(mov.cryptoCode) }}</div>
          <div class="detalle">Cantidad: {{ mov.cantidad }}</div>
          <div class="detalle">Monto: {{ formatCurrency(mov.montoARS) }}</div>
          <div class="detalle">Fecha: {{ formatFecha(mov.fechaHora) }}</div>
          <div class="detalle">Tipo: {{ mov.accion }}</div>
        </div>
      </div>
    </div>

    <p v-else class="mensaje-no-hay">No hay movimientos registrados todavía.</p>

    <pre style="color: white; background: #011; padding: 1rem; border-radius: 10px;">
    {{ JSON.stringify(movimientos, null, 2) }}
    </pre>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const clientes = ref([])
const clienteSeleccionado = ref('')
const movimientos = ref([])

const datosformulario = ref({
  id: 0,
  cryptoCode: '',
  accion: '',
  cantidad: 0,
  monto: 0,
  fechayhora: ''
})

const cryptoNames = {
  BTC: 'Bitcoin',
  ETH: 'Ethereum',
  USDT: 'Tether',
  BNB: 'Bnb',
  // Agrega más según necesites
}

function getCryptoName(code) {
  return cryptoNames[code] || code
}

const ultimaTransaccion = ref(null)

function guardarTransaccion() {
  ultimaTransaccion.value = { ...datosformulario.value }
}

onMounted(async () => {
  try {
    const response = await axios.get('http://localhost:5272/api/Cliente')
    clientes.value = Array.isArray(response.data) ? response.data : response.data.data
  } catch (error) {
    console.error('Error al obtener clientes:', error)
  }
})

const obtenerMovimientos = async () => {
  if (!clienteSeleccionado.value) {
    alert('Por favor seleccione un cliente.')
    return
  }

  try {
    const response = await axios.get(
      `http://localhost:5272/api/Transactions/byClient/${clienteSeleccionado.value}`
    )
    console.log("Raw movimientos:", response.data)  

    if (Array.isArray(response.data)) {
      movimientos.value = response.data.map(mov => ({
      id: mov.id,
      accion: mov.action === 'purchase' ? 'Compra' : 'Venta',
      cryptoCode: mov.crypto_code,
      cantidad: mov.crypto_amount,                // bien
      montoARS: isNaN(parseFloat(mov.money)) ? 0 : parseFloat(mov.money),
      fechaHora: mov.datetime ? mov.datetime : null
}))
    } else {
      movimientos.value = []
    }
  } catch (error) {
    console.error("Error al obtener las transacciones: ", error)
    movimientos.value = []
  }
}

function mostrarDetalle(mov) {
  datosformulario.value = {
    id: mov.id,
    cryptoCode: mov.cryptoCode,
    accion: mov.accion,
    cantidad: mov.cantidad,
    monto: mov.montoARS,
    fechayhora: mov.fechaHora
  }
}

function formatCurrency(value) {
  return value.toLocaleString('es-AR', {
    style: 'currency',
    currency: 'ARS'
  })
}

function formatFecha(fechaISO) {
  const date = new Date(fechaISO)
  return isNaN(date.getTime())
    ? 'Fecha inválida'
    : date.toLocaleDateString('es-AR', {
        day: '2-digit',
        month: 'short',
        year: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      })
}
</script>

  <style scoped>

  .historial-container {
  background: linear-gradient(145deg, #0d2c45, #092536);
  padding: 2.5rem 3rem;
  border-radius: 24px;
  color: #f0f4f8;
  max-width: 1000px;
  margin: 40px auto;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.4);
  animation: fadeIn 0.5s ease-in-out;
}

.titulo {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  font-size: 2.5rem;
  text-align: center;
  margin-bottom: 2rem;
  font-weight: bold;
  color: #ffffff;
  text-shadow: 0 0 5px rgba(0, 255, 255, 0.3);
}

.formulario {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
  margin-bottom: 2.5rem;
}

.label {
  font-weight: 500;
  font-size: 1rem;
}

.select {
  padding: 0.8rem 1.2rem;
  border-radius: 14px;
  border: 1px solid #1b3e5e;
  background-color: #123652;
  color: #eaf6ff;
  font-size: 1rem;
  outline: none;
  transition: border 0.3s, box-shadow 0.3s;
}

.select:focus {
  border: 1px solid #00c2ff;
  box-shadow: 0 0 6px rgba(0, 255, 255, 0.3);
}

.tarjetas-contenedor {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 1.5rem;
}

.tarjeta {
  background: radial-gradient(circle at top left, #123652, #0d2c45);
  border-radius: 20px;
  padding: 1.5rem;
  box-shadow: 0 6px 16px rgba(0, 255, 255, 0.05), 0 0 10px rgba(0, 255, 255, 0.05);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  animation: aparecer 0.3s ease-in-out;
  border-left: 6px solid transparent;
}

.tarjeta.compra {
  border-left-color: #3dd68c;
}

.tarjeta.venta {
  border-left-color: #ff7070;
}

.tarjeta:hover {
  transform: translateY(-5px);
}

.contenido-tarjeta {

  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.cripto {
  font-size: 1.4rem;
  font-weight: bold;
  color: #00e6ff;
  margin-bottom: 0.5rem;
  text-shadow: 0 0 4px rgba(0, 255, 255, 0.3);
}

.detalle {
  font-size: 0.95rem;
  color: #e0f0ff;
}

.mensaje-no-hay {
  text-align: center;
  font-size: 1.2rem;
  margin-top: 2rem;
  color: #a0bfcf;
}

@keyframes aparecer {
  from {
    opacity: 0;
    transform: translateY(15px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: scale(0.98);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}
  </style>
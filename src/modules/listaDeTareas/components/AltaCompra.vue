<template>
  <div class="registro-container">
    <h1 class="titulo">Comprá acá</h1>

    <form @submit.prevent="registro" class="formulario">
      <div class="input-group">
        <label for="clienteId">ID de Cliente</label>
        <input
          type="number"
          id="clienteId"
          v-model="datosformulario.clienteId"
          required
          placeholder="Ej: 1"
        />
      </div>

      <div class="input-group">
        <label for="crypto">Criptomoneda</label>
        <select v-model="datosformulario.cryptoCode" id="crypto" required>
          <option disabled value="">Elegí una criptomoneda</option>
          <option
            v-for="crypto in criptosDisponibles"
            :key="crypto"
            :value="crypto"
          >
            {{ crypto.toUpperCase() }}
          </option>
        </select>
      </div>

      <div class="input-group">
        <label for="accion">Acción</label>
        <select v-model="datosformulario.accion" id="accion" required>
          <option disabled value="">¿Qué desea realizar?</option>
          <option value="buy">Compra</option>
          <option value="sale">Venta</option>
        </select>
      </div>

      <div class="input-group">
        <label for="cantidad">Cantidad</label>
        <input
          type="number"
          step="0.00000001"
          min="0.00000001"
          :value="datosformulario.cantidad.toFixed(8)"
          @input="e => datosformulario.cantidad = parseFloat(e.target.value)"
          placeholder="Ej: 0.00000001"
          required
        />
      </div>

      <div class="input-group">
        <label for="monto">Monto en ARS</label>
        <input
          type="text"
          :value="datosformulario.monto.toLocaleString('es-AR')"
          readonly
          placeholder="Precio en pesos"
        />
        <input type="hidden" v-model="datosformulario.monto" />
      </div>

      <div class="input-group">
        <label for="datetime">Fecha y hora actual:</label>
        <div class="fechaHora">{{ fechaHoraActual }}</div>
      </div>

      <button class="btn-registrar" type="submit" :disabled="loading">
        Confirmar
      </button>

      <p v-if="loading" class="loading-mensaje">Cargando...</p>
      <div v-if="mensaje" class="mensaje-exito">{{ mensaje }}</div>
      <div v-if="error" class="mensaje-error">{{ error }}</div>
    </form>
  </div>
</template>

<script setup>

import { ref, watch, onMounted, onBeforeUnmount } from 'vue'
import axios from 'axios'

const mensaje = ref('')
const error = ref('')
const loading = ref(false)

const datosformulario = ref({
  cryptoCode: '',
  accion: 'buy',
  clienteId: 1,
  cantidad: 0.00000001,
  monto: 0,
  fechayhora: ''
})

const criptosDisponibles = ['btc', 'eth', 'usdc', 'bnb']

const preciosCryptos = async (cripto) => {
  try {
    const url = `https://criptoya.com/api/bybit/${cripto.toLowerCase()}/ars`
    const response = await axios.get(url)
    if (response.data && response.data.ask) {
      return response.data.ask
    } else {
      throw new Error('Formato inesperado en la respuesta de CriptoYa')
    }
  } catch (err) {

    throw new Error('✖️ No se pudo obtener el precio de la cripto')
  }
}

const calcularMonto = async () => {
  const cripto = datosformulario.value.cryptoCode
  const cantidad = datosformulario.value.cantidad

  if (!cripto || cantidad <= 0) {
    datosformulario.value.monto = 0
    return
  }
 
  try {
    const precio = await preciosCryptos(cripto)
    datosformulario.value.monto = parseFloat((cantidad * precio).toFixed(2))
  } catch (err) {
    error.value = err.message
    datosformulario.value.monto = 0
  }
}


watch(() => [datosformulario.value.cryptoCode, datosformulario.value.cantidad], calcularMonto)

const registro = async () => {
  mensaje.value = ''
  error.value = ''

  if (!datosformulario.value.cryptoCode || datosformulario.value.cantidad <= 0) {
    error.value = 'Complete todos los campos correctamente'
    return
  }

  try {
    loading.value = true
    await calcularMonto()

    const compra = {
      cryptoCode: datosformulario.value.cryptoCode,
      clienteId: datosformulario.value.clienteId,
      accion: datosformulario.value.accion,

      cantidad: parseFloat(datosformulario.value.cantidad),
      monto: parseFloat(datosformulario.value.monto),
      fechaHora: new Date().toISOString()
    }


    await axios.post('http://localhost:5272/api/Transactions', compra)

    mensaje.value = '✅ Transacción realizada con éxito'
    datosformulario.value = {
      cryptoCode: '',
      accion: 'buy',
      clienteId: 1,
      cantidad: 0.00000001,
      monto: 0,
      fechayhora: ''
    }
  } catch (err) {
    error.value = err?.response?.data || 'Error al enviar los datos'
  } finally {
    loading.value = false
  }
}

const fechaHoraActual = ref('')

function formatearFechaHora(date) {
  const yyyy = date.getFullYear()
  const mm = String(date.getMonth() + 1).padStart(2, '0')
  const dd = String(date.getDate()).padStart(2, '0')
  const hh = String(date.getHours()).padStart(2, '0')
  const min = String(date.getMinutes()).padStart(2, '0')
  return `${yyyy}-${mm}-${dd} ${hh}:${min}`
}

function actualizarFechaHora() {
  fechaHoraActual.value = formatearFechaHora(new Date())
}

let intervalo

onMounted(() => {
  actualizarFechaHora()
  intervalo = setInterval(actualizarFechaHora, 60000)
})

onBeforeUnmount(() => {
  clearInterval(intervalo)
})
</script>

<style scoped>
.registro-container {
  
  background: linear-gradient(145deg, #0d2c45, #092536);
  padding: 2.5rem 3rem;
  border-radius: 24px;
  color: #f0f4f8;
  max-width: 600px;
  margin: 40px auto;
  box-shadow: 0 25px 45px rgba(0, 0, 0, 0.3);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  animation: fadeIn 0.6s ease;
  user-select: none;
}

.titulo {
  text-align: center;
  font-size: 2.4rem;
  margin-bottom: 1.5rem;


  font-weight: 700;
  letter-spacing: 1px;
  text-shadow: 0 0 6px #0f2133;
}

.formulario {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
  background: #123a5a;
  padding: 4%;
  border-radius: 16px;
  margin-bottom: 2.5rem;
  box-shadow: inset 0 0 5px #0f2844;
  transition: box-shadow 0.3s ease;
}

.formulario:hover {
  box-shadow: inset 0 0 22px #161616;
}

.input-group label {
  display: block;
  margin-bottom: 0.6rem;
  font-weight: 600;
  color: #a8c0ff;
  user-select: text;
}

.input-group input,
.input-group select {
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

.input-group input:focus,
.input-group select:focus {
  box-shadow: 0 0 10px #090d11;
}

.fechaHora {
  padding: 0.65rem 1rem;
  background-color: #c4d8ff;
  color: #0d2c45;
  border-radius: 12px;
  font-weight: 600;
  box-shadow: inset 0 0 8px #0f2133;
}

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

.mensaje-exito {
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

.loading-mensaje {
  text-align: center;
  color: #f0f4f8;
  font-weight: 600;
  margin-top: 1rem;

}

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
</style>
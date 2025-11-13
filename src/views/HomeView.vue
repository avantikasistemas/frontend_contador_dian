<template>
    <div class="container mt-2">
        <div class="general-header">
            <div class="header-app">
                <img :src="logotipo" :alt="logotipo" class="logo-app" />
                <h4 class="titulo-app">VERIFICACIÓN FACTURACIÓN ELECTRÓNICA ERP - DIAN</h4>
            </div>
        </div>

        <!-- Sección para cargar archivo Excel -->
        <div class="seccion-carga-archivo mt-4">
            <div class="d-flex align-items-center gap-3">
                <div class="input-file-wrapper">
                    <label for="archivoExcel" class="form-label fw-bold mb-2">Cargar Archivo Excel - DIAN:</label>
                    <input 
                        type="file" 
                        id="archivoExcel" 
                        class="form-control" 
                        accept=".xlsx,.xls"
                        @change="handleFileChange"
                        ref="fileInput"
                    />
                </div>
                <button 
                    type="button" 
                    class="btn btn-primary btn-procesar"
                    @click="procesarArchivo"
                    :disabled="!archivoSeleccionado"
                >
                    <span>📤</span> Procesar Archivo DIAN
                </button>
            </div>
            <small v-if="nombreArchivo" class="text-muted mt-2 d-block">
                Archivo seleccionado: <strong>{{ nombreArchivo }}</strong>
            </small>
        </div>

        <!-- Sección para cargar archivo DMS -->
        <div class="seccion-carga-archivo mt-4">
            <div class="d-flex align-items-center gap-3">
                <div class="input-file-wrapper">
                    <label for="archivoDMS" class="form-label fw-bold mb-2">Cargar Archivo DMS:</label>
                    <input 
                        type="file" 
                        id="archivoDMS" 
                        class="form-control" 
                        accept=".xlsx,.xls"
                        @change="handleFileChangeDMS"
                        ref="fileInputDMS"
                    />
                </div>
                <button 
                    type="button" 
                    class="btn btn-success btn-procesar"
                    @click="procesarArchivoDMS"
                    :disabled="!archivoSeleccionadoDMS"
                >
                    <span>📤</span> Procesar Archivo DMS
                </button>
            </div>
            <small v-if="nombreArchivoDMS" class="text-muted mt-2 d-block">
                Archivo seleccionado: <strong>{{ nombreArchivoDMS }}</strong>
            </small>
        </div>

        <!-- Sección para enviar correo -->
        <div class="seccion-enviar-correo mt-4 text-center">
            <button 
                type="button" 
                class="btn btn-warning btn-enviar-correo"
                @click="enviarCorreo"
                :disabled="loading"
            >
                <span>📧</span> Enviar Correo con Resumen
            </button>
            <!-- <p class="text-muted mt-2 small">
                Se enviará un resumen de los últimos datos procesados de DIAN y DMS
            </p> -->
        </div>

    </div>


    <!-- Modal de éxito -->
    <div class="modal fade" id="exitoModal" tabindex="-1" aria-labelledby="exitoModalLabel" aria-hidden="true" data-bs-backdrop="static" ref="exitoModal">
        <div class="modal-dialog modal-dialog-centered">
            <div class="modal-content">
                <div class="modal-header bg-success text-white">
                    <div class="d-flex align-items-center w-100">
                        <span class="me-2" style="font-size:2.5rem;line-height:1;">&#10004;</span>
                        <h5 class="modal-title flex-grow-1" id="exitoModalLabel">{{ modalTitle }}</h5>
                        <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal" aria-label="Close"></button>
                    </div>
                </div>
                <div class="modal-body text-success text-center">
                    <p>{{ msg }}</p>                    
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cerrar</button>
                </div>
            </div>
        </div>
    </div>

    <!-- Modal de error -->
    <div class="modal fade" id="errorModal" tabindex="-1" aria-labelledby="errorModalLabel" aria-hidden="true" data-bs-backdrop="static" ref="errorModal">
        <div class="modal-dialog modal-dialog-centered">
            <div class="modal-content">
                <div class="modal-header bg-danger text-white">
                    <div class="d-flex align-items-center w-100">
                        <span class="me-2" style="font-size:2.5rem;line-height:1;">&#10006;</span>
                        <h5 class="modal-title flex-grow-1" id="errorModalLabel">Error</h5>
                        <button type="button" class="btn-close btn-close-white" data-bs-dismiss="modal" aria-label="Close"></button>
                    </div>
                </div>
                <div class="modal-body text-danger text-center">
                    <strong>{{ errorMsg }}</strong>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cerrar</button>
                </div>
            </div>
        </div>
    </div>

    <!-- Modal de Edición de Registro -->
    <div class="modal fade" id="editarModal" tabindex="-1" aria-labelledby="editarModalLabel" aria-hidden="true" data-bs-backdrop="static" ref="editarModal">
        <div class="modal-dialog modal-xl modal-dialog-scrollable">
            <div class="modal-content">
                <div class="modal-header bg-primary text-white">
                    <div class="d-flex align-items-center w-100">
                        <span class="me-2" style="font-size:2rem;line-height:1;">✏️</span>
                        <h5 class="modal-title flex-grow-1" id="editarModalLabel">Editar Registro de Mercadeo</h5>
                        <button type="button" class="btn-close btn-close-white" @click="cerrarModalEdicion" aria-label="Close"></button>
                    </div>
                </div>
                <div class="modal-body">
                    <div class="container-fluid">
                        <!-- Información de Solo Lectura -->
                        <div class="seccion-readonly mb-4">
                            <h6 class="seccion-titulo mb-3">📋 Información del Registro (Solo Lectura)</h6>
                            <div class="row g-3">
                                <div class="col-md-2">
                                    <label class="form-label fw-bold">ID:</label>
                                    <input type="text" class="form-control" :value="registroEdicion.id" readonly>
                                </div>
                                <div class="col-md-5">
                                    <label class="form-label fw-bold">Evento:</label>
                                    <input type="text" class="form-control" :value="registroEdicion.evento" readonly>
                                </div>
                                <div class="col-md-5">
                                    <label class="form-label fw-bold">Nombre:</label>
                                    <input type="text" class="form-control" :value="registroEdicion.nombre" readonly>
                                </div>
                                <div class="col-md-4">
                                    <label class="form-label fw-bold">Celular:</label>
                                    <input type="text" class="form-control" :value="registroEdicion.celular" readonly>
                                </div>
                                <div class="col-md-4">
                                    <label class="form-label fw-bold">Correo:</label>
                                    <input type="email" class="form-control" :value="registroEdicion.correo" readonly>
                                </div>
                                <div class="col-md-4">
                                    <label class="form-label fw-bold">Ciudad:</label>
                                    <input type="text" class="form-control" :value="registroEdicion.ciudad" readonly>
                                </div>
                                <div class="col-md-6">
                                    <label class="form-label fw-bold">Empresa:</label>
                                    <input type="text" class="form-control" :value="registroEdicion.empresa" readonly>
                                </div>
                                <div class="col-md-6">
                                    <label class="form-label fw-bold">Cargo:</label>
                                    <input type="text" class="form-control" :value="registroEdicion.cargo" readonly>
                                </div>
                            </div>
                        </div>

                        <hr class="my-4">

                        <!-- Campos Editables -->
                        <div class="seccion-editable">
                            <h6 class="seccion-titulo mb-3">✏️ Campos Editables</h6>
                            <div class="row g-3">
                                <div class="col-md-12">
                                    <label class="form-label fw-bold">Coordinación: <span class="text-danger">*</span></label>
                                    <input 
                                        type="text" 
                                        class="form-control form-control-editable" 
                                        v-model="registroEdicion.coordinacion"
                                        placeholder="Ej: Zona Norte, Zona Sur, etc."
                                    >
                                </div>
                                <div class="col-md-4">
                                    <label class="form-label fw-bold">Oportunidad:</label>
                                    <div class="input-group">
                                        <span class="input-group-text">$</span>
                                        <input 
                                            type="number" 
                                            class="form-control form-control-editable" 
                                            v-model="registroEdicion.oportunidad"
                                            step="0.01"
                                            min="0"
                                            placeholder="0.00"
                                        >
                                    </div>
                                </div>
                                <div class="col-md-4">
                                    <label class="form-label fw-bold">Monto Cotizado:</label>
                                    <div class="input-group">
                                        <span class="input-group-text">$</span>
                                        <input 
                                            type="number" 
                                            class="form-control form-control-editable" 
                                            v-model="registroEdicion.monto_cotizado"
                                            step="0.01"
                                            min="0"
                                            placeholder="0.00"
                                        >
                                    </div>
                                </div>
                                <div class="col-md-4">
                                    <label class="form-label fw-bold">Facturado:</label>
                                    <div class="input-group">
                                        <span class="input-group-text">$</span>
                                        <input 
                                            type="number" 
                                            class="form-control form-control-editable" 
                                            v-model="registroEdicion.facturado"
                                            step="0.01"
                                            min="0"
                                            placeholder="0.00"
                                        >
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" @click="cerrarModalEdicion">
                        ❌ Cancelar
                    </button>
                    <button type="button" class="btn btn-primary" @click="guardarCambios">
                        💾 Guardar Cambios
                    </button>
                </div>
            </div>
        </div>
    </div>
    
    <!-- Overlay de carga -->
    <div v-if="loading" class="loading-overlay">
        <div class="spinner-border text-light" role="status">
            <span class="visually-hidden"></span>
        </div>
        <p class="mt-2 text-light">{{ loading_msg }}</p>
    </div>
</template>

<script setup>

import { ref, onMounted, watch, computed } from 'vue';
import { useRouter } from 'vue-router';
import axios from 'axios';
import { Modal } from 'bootstrap';
import logotipo from '@/assets/logotipo.png';
import apiUrl from "../../config.js";

const cargo = ref(null);
const lugarInspeccionId = ref(null);
const lugaresInspeccion = ref([]);
const responsables = ref([]);
const fechaDesde = ref(null);
const fechaHasta = ref(null);
const fechaDesdeFormateada = ref(null);
const fechaHastaFormateada = ref(null);

const novedades = ref(null);
const registros = ref([]);

// Variables para el manejo del archivo Excel
const archivoSeleccionado = ref(null);
const nombreArchivo = ref('');
const fileInput = ref(null);
const tipoArchivo = ref('ventas'); // Tipo de archivo por defecto: 'ventas', 'compras', etc.

// Variables para el manejo del archivo DMS
const archivoSeleccionadoDMS = ref(null);
const nombreArchivoDMS = ref('');
const fileInputDMS = ref(null);

// Variables para filtros y paginación
const filtros = ref({
    evento: '',
    nombre: '',
    empresa: '',
    ciudad: '',
    coordinacion: ''
});
const mostrarTabla = ref(false);
const totalRegistros = ref(0);
const totalPaginas = ref(0);
const paginaActual = ref(1);
const registrosPorPagina = ref(15);

const msg = ref('');
const errorMsg = ref('');
const modalTitle = ref('');

const modalInstance = ref(null);
const modalErrorInstance = ref(null);
const modalEditarInstance = ref(null);

// Variables para edición de registro
const registroEdicion = ref({
    id: null,
    evento: '',
    nombre: '',
    celular: '',
    correo: '',
    empresa: '',
    ciudad: '',
    cargo: '',
    coordinacion: '',
    oportunidad: 0,
    monto_cotizado: 0,
    facturado: 0
});

const loading = ref(false);
const loading_msg = ref('');

const total_paginas = ref(0);
const total_registros = ref(0);
const limit = ref(10);
const position = ref(1);

const router = useRouter();

// Función para validar extensión del archivo
const validarExtensionArchivo = (nombreArchivo) => {
    const extensionesPermitidas = ['xls', 'xlsx'];
    const extension = nombreArchivo.split('.').pop().toLowerCase();
    return extensionesPermitidas.includes(extension);
};

// Función para manejar el cambio de archivo
const handleFileChange = (event) => {
    const file = event.target.files[0];
    if (file) {
        // Validar extensión
        if (!validarExtensionArchivo(file.name)) {
            errorMsg.value = 'Solo se permiten archivos Excel (.xls o .xlsx)';
            modalErrorInstance.value.show();
            // Limpiar el input
            event.target.value = '';
            archivoSeleccionado.value = null;
            nombreArchivo.value = '';
            return;
        }
        archivoSeleccionado.value = file;
        nombreArchivo.value = file.name;
    } else {
        archivoSeleccionado.value = null;
        nombreArchivo.value = '';
    }
};

// Función para manejar el cambio de archivo DMS
const handleFileChangeDMS = (event) => {
    const file = event.target.files[0];
    if (file) {
        if (!validarExtensionArchivo(file.name)) {
            errorMsg.value = 'Por favor, seleccione un archivo válido (solo se permiten archivos .xls o .xlsx).';
            modalErrorInstance.value.show();
            // Limpiar el input
            event.target.value = '';
            archivoSeleccionadoDMS.value = null;
            nombreArchivoDMS.value = '';
            return;
        }
        archivoSeleccionadoDMS.value = file;
        nombreArchivoDMS.value = file.name;
    } else {
        archivoSeleccionadoDMS.value = null;
        nombreArchivoDMS.value = '';
    }
};

// Función para convertir archivo a Base64
const convertirArchivoABase64 = (file) => {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => {
            // Extraer solo la parte base64 (eliminar el prefijo "data:...;base64,")
            const base64String = reader.result.split(',')[1];
            resolve(base64String);
        };
        reader.onerror = (error) => reject(error);
    });
};

// Función para procesar y enviar el archivo al backend
const procesarArchivo = async () => {
    if (!archivoSeleccionado.value) {
        errorMsg.value = 'Por favor, seleccione un archivo Excel.';
        modalErrorInstance.value.show();
        return;
    }

    try {
        loading.value = true;
        loading_msg.value = 'Procesando archivo...';

        // Convertir el archivo a Base64
        const archivoBase64 = await convertirArchivoABase64(archivoSeleccionado.value);

        // Enviar al backend
        const response = await axios.post(
            `${apiUrl}/contador/procesar-archivo`,
            { 
                archivo: archivoBase64,
                nombre_archivo: nombreArchivo.value,
                tipo_archivo: tipoArchivo.value
            },
            {
                headers: {
                    Accept: "application/json",
                }
            }
        );

        if (response.status === 200) {
            msg.value = response.data.message || 'Archivo procesado exitosamente';
            modalTitle.value = 'Operación Exitosa';
            modalInstance.value.show();

            // Descargar el archivo procesado
            if (response.data.data && response.data.data.archivo_procesado) {
                descargarArchivoProcesado(
                    response.data.data.archivo_procesado,
                    response.data.data.nombre_archivo_procesado
                );
            }

            // Limpiar el input de archivo
            archivoSeleccionado.value = null;
            nombreArchivo.value = '';
            if (fileInput.value) {
                fileInput.value.value = '';
            }

            // Recargar datos si es necesario
            // await cargarDatos();
        }
    } catch (error) {
        console.error(error);
        errorMsg.value = error.response?.data?.message || 'Error al procesar el archivo.';
        modalErrorInstance.value.show();
    } finally {
        loading.value = false;
        loading_msg.value = '';
    }
};

// Función para procesar y enviar el archivo DMS al backend
const procesarArchivoDMS = async () => {
    if (!archivoSeleccionadoDMS.value) {
        errorMsg.value = 'Por favor, seleccione un archivo DMS.';
        modalErrorInstance.value.show();
        return;
    }

    try {
        loading.value = true;
        loading_msg.value = 'Procesando archivo DMS...';

        // Convertir el archivo a Base64
        const archivoBase64 = await convertirArchivoABase64(archivoSeleccionadoDMS.value);

        // Enviar al backend
        const response = await axios.post(
            `${apiUrl}/contador/procesar-archivo-dms`,
            { 
                archivo: archivoBase64,
                nombre_archivo: nombreArchivoDMS.value,
                tipo_archivo: 'dms'
            },
            {
                headers: {
                    Accept: "application/json",
                }
            }
        );

        if (response.status === 200) {
            msg.value = response.data.message || 'Archivo DMS procesado exitosamente';
            modalTitle.value = 'Operación Exitosa';
            modalInstance.value.show();

            // Descargar el archivo procesado
            if (response.data.data && response.data.data.archivo_procesado) {
                descargarArchivoProcesado(
                    response.data.data.archivo_procesado,
                    response.data.data.nombre_archivo_procesado
                );
            }

            // Limpiar el input de archivo
            archivoSeleccionadoDMS.value = null;
            nombreArchivoDMS.value = '';
            if (fileInputDMS.value) {
                fileInputDMS.value.value = '';
            }
        }
    } catch (error) {
        console.error(error);
        errorMsg.value = error.response?.data?.message || 'Error al procesar el archivo DMS.';
        modalErrorInstance.value.show();
    } finally {
        loading.value = false;
        loading_msg.value = '';
    }
};

// Función para descargar el archivo procesado
const descargarArchivoProcesado = (archivoBase64, nombreArchivo) => {
    try {
        // Convertir Base64 a Blob
        const byteCharacters = atob(archivoBase64);
        const byteNumbers = new Array(byteCharacters.length);
        for (let i = 0; i < byteCharacters.length; i++) {
            byteNumbers[i] = byteCharacters.charCodeAt(i);
        }
        const byteArray = new Uint8Array(byteNumbers);
        const blob = new Blob([byteArray], { 
            type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' 
        });

        // Crear enlace de descarga
        const url = window.URL.createObjectURL(blob);
        const link = document.createElement('a');
        link.href = url;
        link.download = nombreArchivo;
        document.body.appendChild(link);
        link.click();
        
        // Limpiar
        document.body.removeChild(link);
        window.URL.revokeObjectURL(url);
    } catch (error) {
        console.error('Error al descargar archivo:', error);
        errorMsg.value = 'Error al descargar el archivo procesado.';
        modalErrorInstance.value.show();
    }
};


// Función para enviar correo con resumen
const enviarCorreo = async () => {
    try {
        loading.value = true;
        loading_msg.value = 'Enviando correo...';

        // Enviar al backend
        const response = await axios.post(
            `${apiUrl}/graph/enviar-correo`,
            {},
            {
                headers: {
                    Accept: "application/json",
                }
            }
        );

        if (response.status === 200) {
            msg.value = response.data.message || 'Correo enviado exitosamente';
            modalTitle.value = 'Operación Exitosa';
            modalInstance.value.show();
        }
    } catch (error) {
        console.error(error);
        errorMsg.value = error.response?.data?.message || 'Error al enviar el correo.';
        modalErrorInstance.value.show();
    } finally {
        loading.value = false;
        loading_msg.value = '';
    }
};


// Código que se ejecuta al montar el componente
onMounted(async () => {
    modalInstance.value = new Modal(exitoModal);
    modalErrorInstance.value = new Modal(errorModal);
    modalEditarInstance.value = new Modal(editarModal);

});

</script>

<style scoped>

.general-header {
    display: flex;
    justify-content: flex-start;
}

.header-app {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 1rem;
    margin-top: 2rem;
    margin-bottom: 1rem;
}

.logo-app {
    height: 48px;
    width: auto;
    object-fit: contain;
}

.titulo-app {
    margin-bottom: 0;
    font-weight: bold;
}

.container {
    max-width: 95vw;
    width: 98vw;
    margin: 0 auto;
}

/* Estilos para la sección de carga de archivo */
.seccion-carga-archivo {
    background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0,0,40,0.1);
}

.input-file-wrapper {
    flex: 1;
    max-width: 400px;
}

.input-file-wrapper .form-label {
    color: #2c3e50;
    font-size: 0.95rem;
}

.input-file-wrapper .form-control {
    border: 2px solid #3498db;
    border-radius: 6px;
    padding: 8px 12px;
    font-size: 0.9rem;
}

.input-file-wrapper .form-control:focus {
    border-color: #2874a6;
    box-shadow: 0 0 0 0.2rem rgba(52, 152, 219, 0.25);
}

.btn-procesar {
    margin-top: 28px;
    background: linear-gradient(90deg, #3498db 0%, #2874a6 100%);
    border: none;
    padding: 10px 20px;
    font-weight: 600;
    font-size: 0.95rem;
    border-radius: 6px;
    display: flex;
    align-items: center;
    gap: 8px;
    transition: all 0.3s ease;
    box-shadow: 0 2px 6px rgba(52, 152, 219, 0.3);
}

.btn-procesar:hover:not(:disabled) {
    background: linear-gradient(90deg, #2874a6 0%, #1f618d 100%);
    transform: translateY(-2px);
    box-shadow: 0 4px 10px rgba(52, 152, 219, 0.4);
}

.btn-procesar:disabled {
    background: #95a5a6;
    cursor: not-allowed;
    opacity: 0.6;
}

.btn-procesar span {
    font-size: 1.2rem;
}

/* Estilos para la sección de enviar correo */
.seccion-enviar-correo {
    background: linear-gradient(135deg, #fff3e0 0%, #ffe0b2 100%);
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0,0,40,0.1);
}

.btn-enviar-correo {
    background: linear-gradient(90deg, #ff9800 0%, #f57c00 100%);
    border: none;
    padding: 12px 30px;
    font-weight: 600;
    font-size: 1rem;
    border-radius: 6px;
    display: inline-flex;
    align-items: center;
    gap: 10px;
    transition: all 0.3s ease;
    box-shadow: 0 2px 6px rgba(255, 152, 0, 0.3);
    color: white;
}

.btn-enviar-correo:hover:not(:disabled) {
    background: linear-gradient(90deg, #f57c00 0%, #e65100 100%);
    transform: translateY(-2px);
    box-shadow: 0 4px 10px rgba(255, 152, 0, 0.4);
    color: white;
}

.btn-enviar-correo:disabled {
    background: #95a5a6;
    cursor: not-allowed;
    opacity: 0.6;
}

.btn-enviar-correo span {
    font-size: 1.3rem;
}

/* Estilos para la sección de filtros */
.seccion-filtros {
    background: linear-gradient(135deg, #e8f5e9 0%, #c8e6c9 100%);
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0,0,40,0.1);
}

.seccion-filtros h5 {
    color: #2e7d32;
    font-weight: 700;
    margin-bottom: 15px;
}

.seccion-filtros .form-label {
    font-weight: 600;
    color: #1b5e20;
    font-size: 0.9rem;
    margin-bottom: 5px;
}

.seccion-filtros .form-control {
    border: 2px solid #66bb6a;
    border-radius: 6px;
    font-size: 0.9rem;
}

.seccion-filtros .form-control:focus {
    border-color: #43a047;
    box-shadow: 0 0 0 0.2rem rgba(76, 175, 80, 0.25);
}

.seccion-filtros .btn {
    font-weight: 600;
    padding: 8px 20px;
    border-radius: 6px;
    display: inline-flex;
    align-items: center;
    gap: 8px;
}

/* Estilos para la tabla de registros */
.seccion-tabla {
    background: white;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0,0,40,0.1);
}

.tabla-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.tabla-header h5 {
    color: #1976d2;
    font-weight: 700;
    margin: 0;
}

.tabla-header .badge {
    font-size: 0.95rem;
    padding: 8px 15px;
}

.detalle-title{
    display: flex;
    justify-content: space-between;
}

.btn-excel {
    background-color: #d6f5e3;
    color: #217346;
    border: 1px solid #217346;
    border-radius: 5px;
    padding: 8px 18px 8px 14px;
    font-size: 1em;
    font-weight: 600;
    display: flex;
    align-items: center;
    gap: 6px;
    cursor: pointer;
    transition: background 0.2s, color 0.2s, border 0.2s;
    box-shadow: 0 2px 6px rgba(33,115,70,0.08);
}
.btn-excel:hover {
    background-color: #b6e6c9;
    color: #145c2c;
    border-color: #145c2c;
}

/* Tabla de registros - estilos modernos */
.tabla-scroll {
    max-height: 500px;
    overflow-x: auto;
    overflow-y: auto;
    border-radius: 10px;
    box-shadow: 0 2px 8px rgba(0,0,40,0.08);
}

.tabla-registros {
    width: 100%;
    border-collapse: separate;
    border-spacing: 0;
    background: white;
    border-radius: 10px;
    overflow: hidden;
    font-size: 0.8rem;
}

.tabla-registros th {
    background: linear-gradient(90deg, #1976d2 0%, #1565c0 100%);
    color: #fff;
    font-weight: 600;
    text-align: center;
    padding: 12px 10px;
    font-size: 0.8rem;
    border-bottom: 2px solid #0d47a1;
    position: sticky;
    top: 0;
    z-index: 10;
}

.tabla-registros td {
    text-align: center;
    padding: 10px 8px;
    font-size: 0.75rem;
    color: #424242;
    background: #fafafa;
    border-bottom: 1px solid #e0e0e0;
    white-space: nowrap;
}

.tabla-registros tbody tr:nth-child(even) td {
    background: #f5f5f5;
}

.tabla-registros tbody tr:hover td {
    background: #e3f2fd;
    transition: background 0.2s;
}

/* Botón de editar */
.btn-editar {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border: none;
    padding: 6px 12px;
    font-size: 1.1rem;
    border-radius: 5px;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 2px 4px rgba(102, 126, 234, 0.3);
}

.btn-editar:hover {
    background: linear-gradient(135deg, #764ba2 0%, #667eea 100%);
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(102, 126, 234, 0.4);
}

/* Estilos del modal de edición */
.modal-xl {
    max-width: 95%;
}

/* Z-index para modales de éxito y error por encima del modal de edición */
#exitoModal.modal,
#errorModal.modal {
    z-index: 1070 !important;
}

#exitoModal ~ .modal-backdrop,
#errorModal ~ .modal-backdrop {
    z-index: 1065 !important;
}

.seccion-titulo {
    color: #1976d2;
    font-weight: 700;
    border-bottom: 2px solid #1976d2;
    padding-bottom: 8px;
}

.seccion-readonly {
    background: #f5f5f5;
    padding: 20px;
    border-radius: 8px;
    border-left: 4px solid #9e9e9e;
}

.seccion-readonly .form-control {
    background-color: #e0e0e0;
    border: 1px solid #bdbdbd;
    cursor: not-allowed;
    font-weight: 500;
    color: #424242;
}

.seccion-editable {
    background: #e8f5e9;
    padding: 20px;
    border-radius: 8px;
    border-left: 4px solid #4caf50;
}

.form-control-editable {
    border: 2px solid #66bb6a;
    font-weight: 600;
}

.form-control-editable:focus {
    border-color: #43a047;
    box-shadow: 0 0 0 0.2rem rgba(76, 175, 80, 0.25);
}

/* Estilos para la paginación */
.paginacion {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 15px 0;
    flex-wrap: wrap;
    gap: 15px;
}

.paginacion-info {
    color: #616161;
    font-size: 0.9rem;
    font-weight: 500;
}

.paginacion-controles {
    display: flex;
    align-items: center;
    gap: 8px;
    flex-wrap: wrap;
}

.paginacion-controles .btn {
    padding: 6px 12px;
    font-size: 0.85rem;
    font-weight: 600;
}

.paginacion-numeros {
    font-weight: 600;
    color: #1976d2;
    padding: 0 10px;
    font-size: 0.9rem;
}

/* Estilos para mensaje de sin datos */
.sin-datos {
    padding: 40px 20px;
}

.icono-sin-datos {
    font-size: 4rem;
    display: block;
    opacity: 0.5;
}

.sin-datos h5 {
    color: #616161;
    font-weight: 600;
}

.sin-datos p {
    font-size: 0.95rem;
}

/* Overlay de carga */
.loading-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background: rgba(44, 62, 80, 0.45);
    z-index: 9999;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
}

.loading-overlay .spinner-border {
    width: 3rem;
    height: 3rem;
    border-width: 0.35em;
}

.loading-overlay p {
    color: #fff;
    font-size: 1.15rem;
    margin-top: 1.2rem;
    text-align: center;
    text-shadow: 0 1px 4px rgba(0,0,0,0.18);
}

/* Estilos para la tabla de verificación */
.tabla-verificacion-container {
    margin-top: 20px;
    margin-bottom: 20px;
    overflow-x: auto;
}

.tabla-verificacion {
    width: 100%;
    border-collapse: collapse;
    background: white;
    box-shadow: 0 2px 8px rgba(0,0,40,0.08);
    border-radius: 8px;
    overflow: hidden;
}

.tabla-verificacion thead tr {
    background: linear-gradient(90deg, #5dade2 0%, #3498db 100%);
    color: white;
}

.tabla-verificacion th {
    padding: 14px 12px;
    text-align: center;
    font-weight: 700;
    font-size: 0.95rem;
    border: 1px solid #2e86c1;
    text-transform: uppercase;
}

.tabla-verificacion .col-aspecto {
    background: linear-gradient(90deg, #2874a6 0%, #1f618d 100%);
    width: 55%;
    text-align: left;
}

.tabla-verificacion .col-opcion {
    background: linear-gradient(90deg, #2874a6 0%, #1f618d 100%);
    width: 15%;
}

.tabla-verificacion tbody tr.seccion-header {
    background: linear-gradient(90deg, #2874a6 0%, #1f618d 100%);
    color: white;
    font-weight: bold;
}

.tabla-verificacion tbody tr.seccion-header td {
    padding: 12px 15px;
    font-size: 0.95rem;
    border: 1px solid #1b4f72;
    text-align: left;
    font-weight: 700;
}

.tabla-verificacion tbody tr:not(.seccion-header) {
    background: #ebf5fb;
}

.tabla-verificacion tbody tr:not(.seccion-header):nth-child(even) {
    background: #d6eaf8;
}

.tabla-verificacion tbody tr:not(.seccion-header):hover {
    background: #aed6f1;
    transition: background 0.2s;
}

.tabla-verificacion td {
    padding: 12px 15px;
    border: 1px solid #aed6f1;
    text-align: center;
}

.tabla-verificacion .aspecto-cell {
    text-align: left;
    color: #21618c;
    padding-left: 15px;
    font-size: 0.92rem;
    line-height: 1.5;
}

.tabla-verificacion .numero-badge {
    display: inline-block;
    background: linear-gradient(135deg, #3498db 0%, #2874a6 100%);
    color: white;
    padding: 4px 10px;
    border-radius: 4px;
    font-weight: 700;
    font-size: 0.85rem;
    margin-right: 10px;
    min-width: 35px;
    text-align: center;
}

.tabla-verificacion .opcion-cell input[type="radio"] {
    cursor: pointer;
    width: 20px;
    height: 20px;
    accent-color: #3498db;
}

.tabla-verificacion .opcion-cell {
    background: rgba(255, 255, 255, 0.7);
}
</style>
<template>
  <div class="pi-extractor">
    <div class="header">
      <h2>Extracteur de Dades del PI (Amb IA Local)</h2>
      <button @click="logout" class="logout-button">Tancar Sessió</button>
    </div>
    <input type="file" @change="handleFileUpload" accept=".docx" />
    
    <button @click="uploadFile" :disabled="!file || isLoading" class="upload-button">
      {{ isLoading ? 'Processant amb IA...' : 'Pujar i Extreure Dades' }}
    </button>
    
    <p v-if="error" class="error-message">Error: {{ error }}</p>
    
    <div v-if="extractedData" class="results-container">
      <h3>✅ Dades Extretes Correctament</h3>
      
      <h4>Dades de l'Alumne</h4>
      <div class="data-group">
        <p><strong>Nom i Cognoms:</strong> {{ extractedData.dadesAlumne.nomCognoms || 'N/D' }}</p>
        <p><strong>Data de Naixement:</strong> {{ extractedData.dadesAlumne.dataNaixement || 'N/D' }}</p>
        <p><strong>Curs:</strong> {{ extractedData.dadesAlumne.curs || 'N/D' }}</p>
      </div>

      <h4>Motiu / Diagnòstic</h4>
      <p class="data-group diagnostic-text">{{ extractedData.motiu.diagnostic || 'N/D' }}</p>

      <h4 v-if="extractedData.adaptacionsGenerals && extractedData.adaptacionsGenerals.length">
        Adaptacions Generals Proposades (Sí)
      </h4>
      <ul class="data-list" v-if="extractedData.adaptacionsGenerals && extractedData.adaptacionsGenerals.length">
        <li v-for="(adaptacio, index) in extractedData.adaptacionsGenerals" :key="'A'+index">{{ adaptacio }}</li>
      </ul>

      <h4 v-if="extractedData.orientacions && extractedData.orientacions.length">
        Orientacions
      </h4>
      <div class="orientations-list data-list" v-if="extractedData.orientacions && extractedData.orientacions.length">
        <div v-for="(orientacio, index) in extractedData.orientacions" :key="'O'+index">
          {{ index + 1 }}. {{ orientacio }}
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();
const file = ref(null);
const extractedData = ref(null);
const isLoading = ref(false);
const error = ref(null);
const apiUrl = 'http://localhost:4000/upload';

const handleFileUpload = (event) => {
  file.value = event.target.files[0];
  extractedData.value = null;
  error.value = null;
};

const uploadFile = async () => {
  if (!file.value) return;

  isLoading.value = true;
  error.value = null;

  const formData = new FormData();
  formData.append('piFile', file.value); 

  try {
    const response = await fetch(apiUrl, {
      method: 'POST',
      body: formData 
    });

    if (!response.ok) {
      const errorText = await response.text();
      throw new Error(`Error del servidor: ${response.status}. Detall: ${errorText.substring(0, 100)}...`);
    }

    const data = await response.json();
    extractedData.value = data.data;

  } catch (err) {
    console.error('Error en la petició o processament:', err);
    error.value = `No es pot extreure les dades. Assegura't que el servidor Node/Ollama està en marxa. Detall: ${err.message}`;
    extractedData.value = null;
  } finally {
    isLoading.value = false;
  }
};

const logout = () => {
  localStorage.removeItem('user');
  router.push('/login');
};
</script>

<style scoped>
.pi-extractor {
  font-family: 'Montserrat', sans-serif;
  max-width: 850px;
  margin: 40px auto;
  padding: 30px;
  border: 1px solid #dcdcdc;
  border-radius: 10px;
  box-shadow: 0 6px 15px rgba(0, 0, 0, 0.1);
  background-color: #fcfcfc;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
}

h2 {
    color: #1a5c88;
    border-bottom: 3px solid #1a5c88; 
    padding-bottom: 15px;
    font-weight: 600;
}

h4 {
    color: #4a4a4a; 
    margin-top: 30px;
    margin-bottom: 10px;
    font-size: 1.2em;
    font-weight: 600;
    border-left: 5px solid #6c757d;
    padding-left: 10px;
}

.upload-button {
    background-color: #28a745;
    color: white;
    border: none;
    padding: 10px 18px;
    border-radius: 5px;
    cursor: pointer;
    margin-left: 10px;
    font-weight: 500;
    transition: background-color 0.3s;
}

.upload-button:hover:not(:disabled) {
    background-color: #1e7e34;
}
.upload-button:disabled {
    background-color: #cccccc;
    cursor: not-allowed;
}

.logout-button {
    background-color: #dc3545;
    color: white;
    border: none;
    padding: 10px 18px;
    border-radius: 5px;
    cursor: pointer;
    font-weight: 500;
    transition: background-color 0.3s;
}

.logout-button:hover {
    background-color: #c82333;
}

.results-container {
  color: black;
  margin-top: 30px;
  padding: 25px;
  background-color: #e9f7ef;
  border: 1px solid #c3e6cb;
  border-radius: 8px;
}

.data-group {
    padding: 15px;
    background-color: #ffffff;
    border: 1px solid #dee2e6;
    border-radius: 6px;
    margin-bottom: 20px;
}
.data-group p {
    margin: 5px 0;
}
strong {
    color: #1a5c88;
}
.diagnostic-text {
    font-style: italic;
    color: #555;
    background-color: #f8f9fa;
    border-left: 4px solid #adb5bd;
    padding: 10px;
    border-radius: 4px;
}

.data-list {
  list-style-type: none;
  padding-left: 0;
  margin-left: 15px;
}

.data-list li, .orientations-list div {
  margin-bottom: 10px;
  padding: 8px 10px;
  background-color: #ffffff;
  border-left: 4px solid #1a5c88;
  border-radius: 4px;
  line-height: 1.4;
}

.data-list li:last-child {
    border-bottom: none;
}
.orientations-list div {
    border-left: 4px solid #6c757d;
}

.error-message {
  color: white;
  background-color: #dc3545;
  font-weight: bold;
  margin-top: 15px;
  padding: 10px;
  border-radius: 4px;
  border: 1px solid #bd2130;
}
</style>
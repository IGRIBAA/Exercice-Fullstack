<template>
  <div class="card">
    <h2>Enregistrer une participation</h2>

    <form class="participation-form" @submit.prevent="enregistrerParticipation">
      <!-- Personne -->
      <label for="personne" class="label">Personne</label>
      <select
        id="personne"
        v-model="selectedPersonne"
        required
        class="select"
      >
        <option value="" disabled>-- Séléctionne une personne </option>
        <option
          v-for="personne in personnes"
          :key="personne.matricule"
          :value="personne.matricule"
        >
          {{ personne.nom }} {{ personne.prenom }}
        </option>
      </select>

      <!-- Projet -->
      <label for="projet" class="label">Projet</label>
      <select
        id="projet"
        v-model="selectedProjet"
        required
        class="select"
      >
        <option value="" disabled>-- Séléctionne un projet --</option>
        <option
          v-for="projet in projets"
          :key="projet.id"
          :value="projet.id"
        >
          {{ projet.nom }}
        </option>
      </select>

      <!-- Rôle -->
      <label for="role" class="label">Rôle</label>
      <input
        id="role"
        type="text"
        v-model="role"
        placeholder="developpeur"
        required
        class="input"
      />

      <!-- Pourcentage (Slider) -->
      <label for="pourcentage" class="label">Pourcentage</label>
      <div class="slider-container">
        <input
          id="pourcentage"
          type="range"
          v-model.number="pourcentage"
          step="0.1"
          min="0"
          max="1"
          class="slider"
        />
        <span class="slider-value">{{ (pourcentage * 100).toFixed(0) }}%</span>
      </div>

      <button type="submit" class="btn-submit">Enregistrer</button>
    </form>

    <!-- Messages de feedback -->
    <div v-if="errorMsg" class="error">{{ errorMsg }}</div>
    <div v-if="successMsg" class="success">{{ successMsg }}</div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

// Données réactives
const personnes = ref([])
const projets = ref([])
const selectedPersonne = ref('')
const selectedProjet = ref('')
const role = ref('')
const pourcentage = ref(0.1) // Valeur par défaut : 0.1 = 10%
const errorMsg = ref('')
const successMsg = ref('')

// Charger la liste des personnes
const chargerPersonnes = async () => {
  try {
    const resp = await axios.get('/api/personnes')
    personnes.value = resp.data._embedded
      ? resp.data._embedded.personnes
      : resp.data
  } catch (err) {
    errorMsg.value = 'Erreur chargement personnes : ' + err
  }
}

// Charger la liste des projets
const chargerProjets = async () => {
  try {
    const resp = await axios.get('/api/projets')
    projets.value = resp.data._embedded
      ? resp.data._embedded.projets
      : resp.data
  } catch (err) {
    errorMsg.value = 'Erreur chargement projets : ' + err
  }
}

// Enregistrer la participation
const enregistrerParticipation = async () => {
  const payload = {
    matricule: selectedPersonne.value,
    codeProjet: selectedProjet.value,
    role: role.value,
    pourcentage: pourcentage.value,
  }
  try {
    await axios.post('/api/gestion/participation', payload)
    successMsg.value = 'Participation enregistrée avec succès.'
    errorMsg.value = ''
    // Réinitialisation
    selectedPersonne.value = ''
    selectedProjet.value = ''
    role.value = ''
    pourcentage.value = 0.1
  } catch (err) {
    successMsg.value = ''
    if (err.response && err.response.data && err.response.data.message) {
      errorMsg.value = err.response.data.message
    } else {
      errorMsg.value = 'Erreur lors de l’enregistrement.'
    }
  }
}

onMounted(() => {
  chargerPersonnes()
  chargerProjets()
})
</script>

<style scoped>

.card {
  max-width: 400px;
  margin: 2rem auto;
  padding: 2rem;
  background-color: #E3E3E3;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1); /* Ombre douce pour la carte */
}

h2 {
  margin-bottom: 1.5rem;
  text-align: center;
  font-size: 1.25rem; /* Taille de la police du titre */
}

/* Formulaire de participation */
.participation-form {
  display: flex;
  flex-direction: column;
  gap: 1rem; /* Espacement entre les éléments du formulaire */
}

.label {
  font-weight: 500;
  margin-bottom: 0.25rem; /* Espacement sous les labels */
}

.select, .input {
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-size: 0.9rem; /* Taille de la police pour les champs */
}

/* Conteneur du slider */
.slider-container {
  display: flex;
  align-items: center;
  gap: 1rem; /* Espacement entre le slider et la valeur */
}

/* Style du slider */
.slider {
  flex: 1;
  -webkit-appearance: none;
  height: 4px;
  background: #ddd; /* Fond du slider */
  border-radius: 2px;
  outline: none;
  cursor: pointer; /* Curseur pour indiquer qu'on peut interagir */
}
.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: #2196f3; /* Couleur du bouton du slider */
  cursor: pointer;
  border: 2px solid #fff;
  box-shadow: 0 0 2px rgba(0,0,0,0.2); /* Ombre autour du bouton */
}
.slider::-moz-range-thumb {
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: #2196f3; /* Couleur du bouton du slider */
  cursor: pointer;
  border: 2px solid #fff;
  box-shadow: 0 0 2px rgba(0,0,0,0.2); /* Ombre autour du bouton */
}

.slider-value {
  width: 40px;
  text-align: center; /* Centrer la valeur affichée du slider */
  color: #2196f3/* Couleur pour une meilleure visibilité */}

.btn-submit {
  padding: 0.6rem 1rem;
  background-color: #2196f3;
  color: #DCDCDC;
  font-weight: 600;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.2s ease-in-out; /* Transition pour le hover */
  font-size: 0.9rem;
}
.btn-submit:hover {
  background-color: #1E90FF; /* Couleur du bouton au survol */
}

/* --- Feedback --- */
.error {
  margin-top: 1rem;
  color: #d32f2f; /* Couleur rouge pour les erreurs */
}
.success {
  margin-top: 1rem;
  color: #388e3c; /* Couleur verte pour les succès */
}
</style>


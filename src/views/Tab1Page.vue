<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Recherche code postal / ville</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Recherche code postal / ville</ion-title>
        </ion-toolbar>
      </ion-header>
      <ion-item>
        <ion-input v-model="postalCode" label="Code postal" labelPlacement="floating" placeholder="60200"
                   @input="searchByPostalCode"></ion-input>
      </ion-item>
      <ion-item>
        <ion-input v-model="cityName" label="Ville" labelPlacement="floating" placeholder="Compiègne"
                   @input="searchByName"></ion-input>
      </ion-item>
      <ion-item v-if="suggestedCities.length > 0">
        <div>
          <div v-for="city in suggestedCities" @click="searchByCode(city)">
            <p>{{ city.nom }} - {{ city.departement.nom }} ({{ city.departement.code }})</p>
          </div>
        </div>
      </ion-item>

      <h2 v-if="city">Recherche:</h2>
      <div class="city" v-if="city">
        <ion-item>
          <ion-label>Nom: {{ city.nom }}</ion-label>
        </ion-item>
        <ion-item>
          <ion-label>Code: {{ city.code }}</ion-label>
        </ion-item>
        <ion-item>
          <ion-label>Code(s) postal/aux:
            <div class="postal-codes">
              <div class="code" v-for="code in city.codesPostaux">
                <span>{{ code }}</span>
                <br>
              </div>
            </div>
          </ion-label>
        </ion-item>
        <ion-item>
          <ion-label>Code département: {{ city.codeDepartement }}</ion-label>
        </ion-item>
        <ion-item>
          <ion-label>Population: {{ city.population }}</ion-label>
        </ion-item>
      </div>

      <div v-if="city != null" class="establishment">
        <ion-button v-if="establishments.length === 0" expand="block" @click="searchEstablishment">Rechercher un/des établissement(s)</ion-button>
        <h2 v-if="establishments.length > 0">Etablissement(s):</h2>
        <div v-if="establishments.length > 0">
          <div v-for="establishment in establishments">
            <ion-item>
              <ion-label>Nom: {{ establishment.properties.nom }}</ion-label>
            </ion-item>
            <ion-item>
              <ion-label>Email: {{ establishment.properties.email }}</ion-label>
            </ion-item>
            <ion-item>
              <ion-label>Téléphone: {{ establishment.properties.telephone }}</ion-label>
            </ion-item>
          </div>
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup>
import {IonButton, IonContent, IonHeader, IonInput, IonItem, IonPage, IonTitle, IonToolbar} from '@ionic/vue';
import {ref} from "vue";

const postalCode = ref()
const cityName = ref()
const suggestedCities = ref([])
const city = ref()
const establishments = ref([])

let controller = new AbortController();
const signal = controller.signal;

const searchByName = () => {
  postalCode.value = ""
  city.value = null

  fetch('https://geo.api.gouv.fr/communes?nom=' + cityName.value + '&fields=departement,codesPostaux&limit=5', {signal})
    .then(async (response) => {
      suggestedCities.value = await response.json()
    })
}

const searchByCode = (cityObject) => {
  cityName.value = cityObject.nom

  fetch('https://geo.api.gouv.fr/communes?code=' + cityObject.code, {signal})
    .then(async (response) => {
      let responseApi = await response.json()
      suggestedCities.value = []
      city.value = responseApi[0]
      establishments.value = []
    })
}

const searchByPostalCode = () => {
  cityName.value = ""
  suggestedCities.value = []
  city.value = null

  fetch('https://geo.api.gouv.fr/communes?codePostal=' + postalCode.value, {signal})
    .then(async (response) => {
      let responseApi = await response.json()

      city.value = responseApi[0]
      establishments.value = []
    })
}

const searchEstablishment = async () => {
  if (city.value) {
    const callApi = await fetch(`https://etablissements-publics.api.gouv.fr/v3/communes/${city.value.code}/mairie`)
    let responseApi = await callApi.json()
    establishments.value = responseApi.features
  }
}
</script>

<style scoped>
.city {
  margin-top: 30px;
}

.postal-codes {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 10px 30%;
}
</style>

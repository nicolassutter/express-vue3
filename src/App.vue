<script setup lang="ts">
import type { Pet } from '../types/pets'
import { onMounted, ref } from 'vue'
import { api } from './modules/api'

/**
 * Contiendra tous les animaux de l'API
 */
const pets = ref<Pet[]>([])

/**
 * Contiendra le premier animal de l'API
 */
const singlePet = ref<Pet | null>(null)

/**
 * La catégorie du nouvel animal à créer, voir le <select>
 */
const selectedPetRace = ref<Pet['type']>('cat')

/**
 * Quand le composant est monté, on récupère les animaux depuis notre API.
 * Une fois la requête terminée, on sauvegarde les animaux localement dans notre ref `pets`.
 */
onMounted(async () => {
  try {
    pets.value = await api.getPets()

    /**
     * Si nous avons récupérer au moins un animal,
     * nous cherchons depuis l'API le premier animal disponible.
     */
    if (pets.value.length) {
      singlePet.value = await api.getPet(pets.value[0].id)
    }
  } catch (error) {
    /* Idéalement, gérer l'erreur */
  }
})

/**
 * Met à jour le nom du premier animal
 */
const updateFirstPetName = async () => {
  const name = prompt('Nom de l\'animal')
  const firstPetID = pets.value[0]?.id

  if (name && firstPetID) {
    try {
      const updatedPet = await api.updatePet(firstPetID, { name })
      pets.value[0] = updatedPet
      
      /**
       * Si on un animal dans `singlePet`, on le met également à jour
       */
      if (singlePet.value) {
        singlePet.value = updatedPet
      }
    } catch (error) {
      /* Idéalement, gérer l'erreur */
    }
  }
}

/**
 * Suppression du dernier animal
 */
const deleteLastPet = async () => {
  const lastPetIndex = pets.value.length - 1
  const lastPetID = pets.value[lastPetIndex]?.id

  if (lastPetID) {
    try {
      await api.deletePet(lastPetID)
      pets.value = pets.value.filter(pet => pet.id !== lastPetID)
    } catch (error) {
      /* Idéalement, gérer l'erreur */
    }
  }
}

/**
 * Création d'un nouvel animal
 */
const createPet = async () => {
  const name = prompt('Nom de l\'animal')

  if (selectedPetRace.value && name) {
    try {
      const newPet = await api.createPet({ name, type: selectedPetRace.value })
      pets.value.push(newPet)
    } catch (error) {
      /* Idéalement, gérer l'erreur */
    }
  }
}
</script>

<template>
  <main class="main prose p-10 max-w-none">
    <h1>Mes animaux</h1>

    <div class="grid grid-cols-1 md:grid-cols-2 gap-3">
      <!-- Tous les animaux -->
      <section class="all-pets">
        <h2>GET <code>/pets</code></h2>

        <div class="pets-wrapper">
          <article
            v-for="pet in pets"
            :key="`pet-id-${pet.id}`"
            class="pet not-prose bg-white shadow-sm rounded-sm p-5"
          >
            <h3 class="font-bold textl-lg mb-3">
              {{ pet.name }}
            </h3>
            <p>Race : {{ pet.type }}</p>
          </article>
        </div>
      </section>

      <!-- Le premier animal depuis l'API -->
      <section class="single-pet">
        <h2>GET <code>/pet/:id</code></h2>

        <article v-if="singlePet"
          class="pet not-prose bg-white shadow-sm rounded-sm p-5"
        >
          <h3 class="font-bold textl-lg mb-3">
            {{ singlePet.name }}
          </h3>

          <p>Race : {{ singlePet.type }}</p>
        </article>

        <p v-else>Cet animal n'existe pas</p>
      </section>

      <!-- MAJ du nom du premier animal -->
      <section class="update-pet">
        <h2>PATCH <code>/pets/:id</code></h2>

        <button
          class="p-3 rounded bg-slate-800 text-white shadow hover:bg-slate-700"
          v-on:click="updateFirstPetName"
        >
          Changer le nom du premier animal
        </button>
      </section>

      <!-- Ajout d'un animal -->
      <section class="create-pet">
        <h2>POST <code>/pets</code></h2>

        <label class="block" for="pet-race">Race du nouvel animal</label>

        <select class="block my-4 rounded p-3 bg-white shadow-sm border border-slate-200" id="pet-race" v-model="selectedPetRace">
          <option value="cat">Chat</option>
          <option value="dog">Chien</option>
        </select>

        <button
          class="p-3 rounded block bg-slate-800 text-white shadow hover:bg-slate-700"
          v-on:click="createPet"
        >
          Créer un animal
        </button>
      </section>

      <!-- Suppression d'un animal -->
      <section class="delete-pet">
        <h2>DELETE <code>/pets/:id</code></h2>


        <button
          class="p-3 rounded bg-slate-800 text-white shadow hover:bg-slate-700"
          v-on:click="deleteLastPet"
        >
          Supprimer le dernier animal
        </button>
      </section>
    </div>
  </main>
</template>

<style>
html,
body {
  height: 100%;
}

#app {
  background-color: theme('colors.slate.100');
  color: theme('colors.slate.700');
  min-height: 100%;
}

.pets-wrapper {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: theme('spacing.3');
}
</style>

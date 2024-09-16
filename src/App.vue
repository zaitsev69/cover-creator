<script setup>
import { ref } from 'vue'
import "./index.css"
// Références pour gérer l'image de fond et les textes
const backgroundImage = ref('')
const texts = ref([])

// Références pour le contenu du texte, la couleur, la police, et la taille
const newTextContent = ref('Your Text')
const newTextColor = ref('#ffffff')
const newTextFont = ref('Arial')
const newTextSize = ref(24) // Taille par défaut

// Indice du texte actuellement sélectionné
const selectedTextIndex = ref(null)

// Gérer le changement d'image
const handleFileChange = (event) => {
  const file = event.target.files[0]
  if (file) {
    const reader = new FileReader()
    reader.onload = () => {
      backgroundImage.value = `url(${reader.result})`
    }
    reader.readAsDataURL(file)
  }
}

// Ajouter un texte
const addText = () => {
  texts.value.push({
    content: newTextContent.value,
    color: newTextColor.value,
    font: newTextFont.value,
    size: newTextSize.value,
    x: 50,
    y: 50,
  })
  newTextContent.value = 'Your Text' // Réinitialiser après ajout
}

// Sélectionner un texte pour modification
const selectText = (index) => {
  selectedTextIndex.value = index
  const selectedText = texts.value[index]
  newTextContent.value = selectedText.content
  newTextColor.value = selectedText.color
  newTextFont.value = selectedText.font
  newTextSize.value = selectedText.size
}

// Mise à jour en direct des propriétés du texte sélectionné
const updateSelectedText = () => {
  if (selectedTextIndex.value !== null) {
    const selectedText = texts.value[selectedTextIndex.value]
    selectedText.content = newTextContent.value
    selectedText.color = newTextColor.value
    selectedText.font = newTextFont.value
    selectedText.size = newTextSize.value
  }
}

// Supprimer un texte
const deleteText = (index) => {
  texts.value.splice(index, 1)
  selectedTextIndex.value = null // Désélectionner après suppression
}

// Drag-and-drop : commencer le drag
const onDragStart = (index, event) => {
  selectedTextIndex.value = index
  event.dataTransfer.effectAllowed = 'move'
}

// Drag-and-drop : relâcher le texte et le positionner
const onDrop = (event) => {
  const { offsetX, offsetY } = event
  if (selectedTextIndex.value !== null) {
    texts.value[selectedTextIndex.value].x = offsetX
    texts.value[selectedTextIndex.value].y = offsetY
  }
}
</script>

<template>
  <header class="bg-black text-white uppercase pl-32 text-4xl py-10">
    Cover Creator
  </header>

  <!-- Container pour la cover et les contrôles -->
  <div class="flex items-start space-x-5 mt-10 ml-10">
    <!-- Carré avec image de fond -->
    <div 
      class="w-[800px] h-[800px] bg-gray-300 flex justify-center items-center bg-cover bg-center border-2 border-black relative"
      :style="{ backgroundImage: backgroundImage }"
      @dragover.prevent
      @drop="onDrop"
    >
      <!-- Affichage des textes -->
      <div 
        v-for="(text, index) in texts" 
        :key="index"
        class="absolute cursor-move"
        :style="{ top: text.y + 'px', left: text.x + 'px', color: text.color, fontFamily: text.font, fontSize: text.size + 'px' }"
        draggable="true"
        @dragstart="onDragStart(index, $event)"
        @click="selectText(index)"
      >
        {{ text.content }}
      </div>
    </div>

    <!-- Contrôles pour ajouter, modifier et lister les textes -->
    <div class="flex flex-col space-y-5 w-64">
      <!-- Upload d'image de fond -->
      <input 
        type="file" 
        accept="image/*" 
        @change="handleFileChange" 
        class="block w-full text-sm text-gray-900 border border-gray-300 rounded-lg cursor-pointer bg-gray-50 focus:outline-none"
      />

      <!-- Contenu du texte -->
      <textarea
        v-model="newTextContent"
        placeholder="Enter text"
        class="border border-gray-300 p-2 rounded"
        @input="updateSelectedText"
      ></textarea>

      <!-- Sélecteur de couleur -->
      <label for="colorPicker" class="block text-sm font-medium text-gray-700">Text Color</label>
      <input 
        type="color" 
        v-model="newTextColor" 
        id="colorPicker" 
        class="w-full cursor-pointer"
        @input="updateSelectedText"
      />

      <!-- Sélecteur de police -->
      <label for="fontPicker" class="block text-sm font-medium text-gray-700">Font</label>
      <select v-model="newTextFont" id="fontPicker" class="border border-gray-300 p-2 rounded" @change="updateSelectedText">
        <option value="Arial">Arial</option>
        <option value="Times New Roman">Times New Roman</option>
        <option value="Courier New">Courier New</option>
        <option value="Georgia">Georgia</option>
        <option value="Verdana">Verdana</option>
      </select>

      <!-- Slider pour la taille du texte -->
      <label for="sizeSlider" class="block text-sm font-medium text-gray-700">Font Size: {{ newTextSize }}px</label>
      <input 
        type="range" 
        id="sizeSlider"
        v-model="newTextSize" 
        min="10" 
        max="100" 
        class="w-full"
        @input="updateSelectedText"
      />

      <!-- Bouton pour ajouter un texte -->
      <button 
        @click="addText" 
        class="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-700"
      >
        Add Text
      </button>

      <!-- Liste des textes créés -->
      <div class="bg-white shadow-md rounded-lg p-4">
        <h3 class="text-lg font-bold mb-3">Texts</h3>
        <ul class="space-y-2">
          <li 
            v-for="(text, index) in texts" 
            :key="index" 
            class="flex justify-between items-center p-2 bg-gray-100 hover:bg-gray-200 rounded-lg cursor-pointer"
            @click="selectText(index)"
          >
            <!-- Affichage du contenu du texte -->
            <span>{{ text.content }}</span>
            <!-- Bouton de suppression -->
            <button 
              @click.stop="deleteText(index)" 
              class="bg-red-500 text-white px-2 py-1 rounded hover:bg-red-600"
            >
              Delete
            </button>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

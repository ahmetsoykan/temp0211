<template>
  <div>
    <input
      type="text"
      v-model="query"
      @input="onInputChange"
      @keyup.enter="handleEnter"
      placeholder="Search..."
    />    
    <ul v-if="suggestions.length && query">
      <li v-for="suggestion in sortedSuggestions" :key="suggestion.name" @click="selectSuggestion(suggestion.name)">
        {{ suggestion.name }} (Popularity: {{ suggestion.popularity }})
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      query: "",
      suggestions: [],
    };
  },
  computed: {
    sortedSuggestions() {
      return [...this.suggestions].sort((a, b) => b.popularity - a.popularity);
    }
  },
  methods: {
    onInputChange() {
      if (this.query.length > 0) {
        this.fetchSuggestions();
      } else {
        this.suggestions = [];
      }
    },
    async fetchSuggestions() {
      try {
        // Fetch suggestions from the backend, expecting an array of objects with `name` and `popularity`
        const response = await fetch(`http://localhost:3000/search?q=${this.query}`);
        this.suggestions = await response.json();
      } catch (error) {
        console.error("Error fetching suggestions:", error);
      }
    },
    async handleEnter() {
      await this.addOrUpdateItem(this.query);
    },
    async addOrUpdateItem(item) {
      if (!item.trim()) return;

      try {
        // Check if the item already exists in suggestions
        const existingItem = this.suggestions.find(s => s.name.toLowerCase() === item.toLowerCase());

        if (existingItem) {
          // Item exists; increase its popularity
          const response = await fetch(`http://localhost:3000/increase-popularity`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ item: existingItem.name }),
          });

          if (!response.ok) throw new Error("Failed to increase popularity");

          existingItem.popularity++;
        } else {
          // Item does not exist; add it
          const response = await fetch('http://localhost:3000/add-item', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ item }),
          });

          if (!response.ok) throw new Error("Failed to add item");

          // Add new item with popularity 1
          this.suggestions.push({ name: item, popularity: 1 });
        }

        // Clear the input and sort suggestions after adding or updating
        this.query = "";
        this.suggestions = this.sortedSuggestions;
      } catch (error) {
        console.error("Error adding/updating item:", error);
      }
    },
    selectSuggestion(suggestion) {
      this.query = suggestion;
      this.suggestions = [];
    },
  },
};
</script>

<style scoped>
ul {
  list-style: none;
  padding: 0;
  margin: 0;
  background-color: #fff;
  border: 1px solid #ddd;
}

li {
  padding: 8px;
  cursor: pointer;
}

li:hover {
  background-color: #f0f0f0;
}

button {
  margin-left: 8px;
  padding: 6px 12px;
  cursor: pointer;
}
</style>

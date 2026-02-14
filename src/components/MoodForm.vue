<template>
  <div>
    <h2>Mood Check-in</h2>
    
    <input v-model="name" placeholder="Your name" />
    <textarea v-model="mood" placeholder="How are you feeling today?"></textarea>
    <button @click="submitMood" :disabled="loading">Submit</button>

    <p v-if="loading">AI Advisor is thinking...</p>
    <p v-if="aiMessage && !loading">AI Advisor: {{ aiMessage }}</p>
    <p v-if="errorMessage" style="color: red;">{{ errorMessage }}</p>

    <h3 v-if="history.length">Mood History</h3>
    <ul>
      <li v-for="(entry, index) in history" :key="index">
        <strong>{{ entry.name }}:</strong> {{ entry.mood }} → <em>{{ entry.aiMessage }}</em>
      </li>
    </ul>
  </div>
</template>

<script>
import api from '../services/api';

export default {
  data() {
    return {
      name: '',
      mood: '',
      aiMessage: '',
      loading: false,
      errorMessage: '',
      history: []
    };
  },
  methods: {
    async submitMood() {
      if (!this.name || !this.mood) {
        this.errorMessage = "Please enter your name and mood.";
        return;
      }

      this.loading = true;
      this.aiMessage = '';
      this.errorMessage = '';

      try {
        const res = await api.post('/mood', {
          full_name: this.name,
          mood_text: this.mood
        });

        this.aiMessage = res.data.ai_message;

        // Add to history
        this.history.push({
          name: this.name,
          mood: this.mood,
          aiMessage: this.aiMessage
        });

        // Clear input fields
        this.name = '';
        this.mood = '';
      } catch (error) {
        console.error(error);
        this.errorMessage = "Error: Could not reach AI Advisor.";
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>

<style scoped>
input, textarea {
  display: block;
  width: 100%;
  padding: 8px;
  margin-bottom: 10px;
}
button {
  padding: 8px 16px;
  margin-bottom: 10px;
}
ul {
  list-style-type: none;
  padding-left: 0;
}
li {
  margin-bottom: 5px;
}
</style>

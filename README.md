# Podcast Recommendation AI with OpenAI and Supabase 🎧

This project uses **OpenAI** for text embeddings and chat completions, and **Supabase** for storing and retrieving semantically relevant podcast episodes.

<kdb>
  <img width="1727" alt="Screenshot 2025-03-01 at 12 26 51 PM" src="https://github.com/user-attachments/assets/02e318b5-c4da-428d-b7da-64b13c44f5bc" />
</kdb>

## 📌 Features
- Generates embeddings from user queries using **OpenAI's text-embedding-ada-002**.
- Finds the most relevant podcast episodes stored in **Supabase** using **pgvector**.
- Generates a natural language response recommending a podcast using **OpenAI's GPT-4**.

---

## 🚀 Getting Started

### 1️⃣ Install Dependencies
Make sure you have **Node.js** installed. Then install the required packages:
```sh
npm install openai @supabase/supabase-js dotenv
```

### 2️⃣ Set Up Environment Variables
Create a `.env` file in the root directory and add the following:
```env
OPENAI_API_KEY=your_openai_api_key
SUPABASE_URL=your_supabase_url
SUPABASE_ANON_KEY=your_supabase_anon_key
```

### 3️⃣ Configure OpenAI and Supabase
Create a `config.js` file to handle API setup:
```js
import OpenAI from 'openai';
import { createClient } from '@supabase/supabase-js';

export const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });
export const supabase = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_ANON_KEY
);
```

### 4️⃣ Run the Recommendation System
```sh
node index.js
```

---

## 🛠 How It Works

### 1️⃣ Generate an Embedding Vector
We use OpenAI to generate an **embedding** for the user’s query:
```js
async function createEmbedding(input) {
  const embeddingResponse = await openai.embeddings.create({
    model: "text-embedding-ada-002",
    input
  });
  return embeddingResponse.data[0].embedding;
}
```

### 2️⃣ Find the Most Relevant Podcast
We query **Supabase** using `match_documents`, which performs a nearest-neighbor search using **pgvector**:
```js
async function findNearestMatch(embedding) {
  const { data } = await supabase.rpc('match_documents', {
    query_embedding: embedding,
    match_threshold: 0.50,
    match_count: 1
  });
  return data[0].content;
}
```

### 3️⃣ Generate a Natural Language Response
Using **GPT-4**, we generate a podcast recommendation:
```js
async function getChatCompletion(text, query) {
  const chatMessages = [{
    role: 'system',
    content: `You are an enthusiastic podcast expert who loves recommending podcasts...`
  }];

  chatMessages.push({
    role: 'user',
    content: `Context: ${text} Question: ${query}`
  });

  const response = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: chatMessages,
    temperature: 0.5,
    frequency_penalty: 0.5
  });

  console.log(response.choices[0].message.content);
}
```

### 4️⃣ Execute the Flow
All functions are combined in `main()`:
```js
async function main(query) {
  const embedding = await createEmbedding(query);
  const match = await findNearestMatch(embedding);
  await getChatCompletion(match, query);
}

const query = "An episode Elon Musk would enjoy";
main(query);
```

---

## 🏆 Summary
This system takes a **user's query**, generates an **embedding**, finds **relevant podcasts** in **Supabase**, and provides a **natural language recommendation** using **GPT-4**. 🚀

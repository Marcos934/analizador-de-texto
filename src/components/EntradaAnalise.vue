<template>
  <div>
    <textarea
      class="textarea is-info textarea is-medium"
      placeholder="Amo animais"
      v-model="frase"
    ></textarea>
    <div class="buttons">
      <button @click="Analize" class="button is-link mt-2">Analizar Texto</button>
    </div>
  </div>
  <div class="mt-4 container is-max-desktop">
    <p class="title is-2">Resultado:</p>
    <label class="title is-6">Negativo: 80%</label>
    <progress
      class="progress is-medium is-danger"
      value="0.80"
      max="1"
    ></progress>
    <label class="title is-6">Neutro: 80% </label>
    <progress class="progress is-medium is-warning" value="0.80" max="1">
      80%
    </progress>

    <label class="title is-6">Positivo: 80% </label>
    <progress class="progress is-medium is-success" value="0.80" max="1">
      1
    </progress>
  </div>

  <hr />
</template>
<script>

export default {
  data() {
    return {
      frase: "",
      resultado: null,
      sentimentos:{
          positivo: null,
          neutro: null,
          negativo: null
      }
    };
  },
  methods: {
    Analize() {
      const {
        TextAnalyticsClient,
        AzureKeyCredential,
      } = require("@azure/ai-text-analytics");
        const key = process.env.VUE_APP_KEY;
        const endpoint = process.env.VUE_APP_ENDPOINT;

      const textAnalyticsClient = new TextAnalyticsClient(
        endpoint,
        new AzureKeyCredential(key)
      );

      async function sentimentAnalysis(client) {
        const sentimentInput = [
          "Olá Mundo. Este é um texto de entrada que eu adoro",
        ];
        const sentimentResult = await client.analyzeSentiment(sentimentInput);

        sentimentResult.forEach((document) => {
          console.log(`ID: ${document.id}`);
          console.log(`\tDocument Sentiment: ${document.sentiment}`);
          console.log(`\tSentences Sentiment(${document.sentences.length}):`);
          document.sentences.forEach((sentence) => {
            console.log(`\t\tSentence sentiment: ${sentence.sentiment}`);
            console.log(`\t\tSentences Scores:`);
            console.log(
              `\t\tPositive: ${sentence.confidenceScores.positive.toFixed(
                2
              )} \tNegative: ${sentence.confidenceScores.negative.toFixed(
                2
              )} \tNeutral: ${sentence.confidenceScores.neutral.toFixed(2)}`
            );
          });
        });

        //   async function languageDetection(client) {

        //     const languageInputArray = [
        //       "Estou feliz hoje"

        //     ];
        //     const languageResult = await client.detectLanguage(languageInputArray);

        //     languageResult.forEach(document => {
        //         console.log(`\tPrimary Language ${document.primaryLanguage.name}`)
        //     });
        // }
        // languageDetection(textAnalyticsClient);
      }
      sentimentAnalysis(textAnalyticsClient);
    },
    teste(){
      
    }
  },
};
</script>
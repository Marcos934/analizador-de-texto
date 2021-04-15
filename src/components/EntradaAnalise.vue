<template>
  <div>
    <textarea
      class="textarea is-info textarea is-medium"
      placeholder="Amo animais"
      v-model="frase"
      maxlength="100"
    ></textarea>
    <div class="buttons">
      <button @click="Analize" class="button is-link mt-2">
        Analizar Texto
      </button>
    </div>
  </div>

  <div class="mt-4" v-if="resposta == 'enviado'">
    <div class="has-text-centered">Analizando aguarde...</div>
    <progress class="progress is-large is-info" max="100">60%</progress>
  </div>
  <div
    class="mt-4 container is-max-desktop"
    v-else-if="resposta == 'analizado'"
  >
    <p class="title is-2">Resultado: {{ resultado }}</p>
    <label class="title is-6">Negativo: {{ sentimentos.negativo }}%</label>
    <progress
      class="progress is-medium is-danger"
      :value="sentimentos.negativo"
      max="1"
    ></progress>
    <label class="title is-6">Neutro: {{ sentimentos.neutro }}% </label>
    <progress
      class="progress is-medium is-warning"
      :value="sentimentos.neutro"
      max="1"
    >
      80%
    </progress>

    <label class="title is-6">Positivo: {{ sentimentos.positivo }}% </label>
    <progress
      class="progress is-medium is-success"
      :value="sentimentos.positivo"
      max="1"
    >
      1
    </progress>
  </div>

  <hr />
</template>
<script>
export default {
  data() {
    return {
      resposta: null,
      frase: "",
      resultado: null,
      sentimentos: {
        positivo: 0,
        neutro: 0,
        negativo: 0,
      },
    };
  },
  methods: {
    async Analize() {
      if (this.verificaTexto() == false) {
        return 0;
      }

      this.dispResposta("enviado");

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

      await sentimentAnalysis(textAnalyticsClient);
      this.dispResposta("analizado");
    },
    dispResposta(valor) {
      this.resposta = valor;
      console.log(valor);
    },
    verificaTexto() {
      if (this.frase.length == 0 || this.frase == "") {
        alert("A frase para análise não pode ser vazia");
        return false;
      } else if (this.frase.length > 100) {
        alert("A frase deve conter no máximo 100 caracteres");
        return false;
      } else {
        return true;
      }
    },
    teste() {
      return alert();
    },
  },
};
</script>
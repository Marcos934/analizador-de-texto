<template>
  <div>
    <textarea
      class="textarea is-info textarea is-medium"
      placeholder="Adotei um cachorro e estou super contente"
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


 <article class="message is-danger mt-2"  v-if="aviso">
      <div class="message-header">
        <p>Analise IA</p>
        <button class="delete" @click="fecharAviso" aria-label="delete"></button>
      </div>
      <div class="message-body">
        Lembre-se que a análise está sendo feita por "robôs", ou melhor, algoritmos de IA... Por tanto não leve em consideração
        caso erre a análise, lembre-se que o computador ainda não têm sentimentos.
      </div>
    </article>


    <p class="title is-2 has-text-centered">{{ resultado }}</p>

    <label class="title is-6"
      >Positivo: {{ (sentimentos.positivo * 100).toFixed(2) }}%
    </label>
    <progress
      class="progress is-medium is-success"
      :value="sentimentos.positivo"
      max="1"
    ></progress>

    <label class="title is-6"
      >Neutro: {{ (sentimentos.neutro * 100).toFixed(2) }}%
    </label>
    <progress
      class="progress is-medium is-warning"
      :value="sentimentos.neutro"
      max="1"
    >
      80%
    </progress>

    <label class="title is-6"
      >Negativo: {{ (sentimentos.negativo * 100).toFixed(2) }}%</label
    >
    <progress
      class="progress is-medium is-danger"
      :value="sentimentos.negativo"
      max="1"
    ></progress>

  <div class="mt-2 title is-6 has-text-start">
      Idioma identificado: <strong>{{idioma}}</strong>
  </div>
  
  </div>

  <hr />
</template>
<script>
export default {
  data() {
    return {
      resposta: null,
      aviso: true,
      frase: "",
      idioma: "",
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
      this.dispResposta("");
      if (this.verificaTexto() == false) {
        return 0;
      }
    
      this.dispResposta("enviado");
        this.aviso = true
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

       
      await this.analiseDeSentimento(textAnalyticsClient);
      this.dispResposta("analizado");
    },

    async analiseDeSentimento(client) {
      const sentimentInput = [this.frase.replace(".", "")];
      const sentimentResult = await client.analyzeSentiment(sentimentInput);


         const languageResult = await client.detectLanguage(sentimentInput);

          languageResult.forEach(document => {
              this.idioma = document.primaryLanguage.name
          });


      sentimentResult.forEach((document) => {
        console.log(`\tDocument Sentiment: ${document.sentiment}`);
        this.trataResultado(document.sentiment);
        document.sentences.forEach((sentence) => {
          console.log(`\t\tSentences Scores:`);

          this.sentimentos.positivo = sentence.confidenceScores.positive.toFixed(
            2
          );
          this.sentimentos.neutro = sentence.confidenceScores.neutral.toFixed(
            2
          );
          this.sentimentos.negativo = sentence.confidenceScores.negative.toFixed(
            2
          );
          

          console.log(
            `\t\tPositive: ${sentence.confidenceScores.positive.toFixed(2)} 
            \tNegative: ${sentence.confidenceScores.negative.toFixed(2)} 
            \tNeutral: ${sentence.confidenceScores.neutral.toFixed(2)}`
          );
        });
      });

       
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
    trataResultado(valor) {
      this.resultado;
      switch (valor) {
        case "positive":
          this.resultado = "Sentimento Positivo";
          break;
        case "negative":
          this.resultado = "Sentimento Negativo";
          break;
        case "neutral":
          this.resultado = "Sentimento Neutro";
          break;
      }
    },
    fecharAviso() {
      return this.aviso = false;
    },
  },
};
</script>
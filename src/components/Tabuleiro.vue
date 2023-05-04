<template>
  <div>
      <div class="container"> 
        <div class="header">
          <h1 style="font-family: 'sans-serif'; font-size:50px; font-weight: bold; color: black;"> Chessuff</h1>
          <div ><img class="logo" :src="logoUff"></div>
        </div>
          <div class="tabuleiro" v-click-outside="limparQuadrados" v-on:click="limparQuadrados" >
              <div v-for="linha in linhas" v-bind:key="linha" class="linha">
                  <Quadrado 
                      :movimentos="movimentos" 
                      :mostraOpcoesPeao="mostraOpcoesPeao" 
                      :adicionaPeca="adicionaPeca" 
                      :adicionaQuadrado="adicionaQuadrado"
                      :removePecaAtual="removePecaAtual"
                      :isPecaSelecionada="isPecaSelecionada"
                      :capturaPeca="capturaPeca"
                      :mudarLado="mudarLado"
                      :linha="linha"
                      :coluna="coluna"
                      :isLadoAtual="isLadoAtual"
                      :pecaSelecionada="pecaSelecionada"
                      :movimentarPecas="movimentarPecas"
                      :id="''+coluna+linha" 
                      v-for="coluna in colunas" 
                      v-bind:key="coluna"
                  />
              </div>
          </div>
      </div>
  </div>
</template>
<script>
import Quadrado from './Quadrado.vue'
import ClickOutside from 'vue-click-outside'

export default {
  // eslint-disable-next-line vue/multi-word-component-names
  name: "Tabuleiro",
  components: {
      Quadrado,
  },
  data: () => {
      return {
          linhas: [1, 2, 3, 4, 5, 6, 7, 8],
          colunas: [1, 2, 3, 4, 5, 6, 7, 8],
          pecas: [],
          quadrados: new Map(),
          movimentos: [],
          pecaSelecionada: {},
          ladoAtual: 'Branco',
          linhaModal: '',
          colunaModal: '',
          ladoModal: '',
          pecasMortasPretas: new Array()
      };
  },
  computed: {
    logoUff() {
       return require(`../assets/UFF_brasão.png`)
    },
  },
  methods: {
      getQuadrado(coluna, linha){
          coluna = String(coluna)
          linha = String(linha)
          const id = coluna+linha
          return this.quadrados.get(id)
      },
      adicionaPeca(linha, coluna, peca){
          this.pecas.push({linha, coluna, peca})
      },
      adicionaQuadrado(id, linha, coluna, ocupado, quadrado){
          this.quadrados.set(id, {linha, coluna, ocupado, quadrado})
      },
      movimentarPecas(peca, linha, coluna){
          setTimeout(() => {
              const qntMovimentos = 8
              var movimentos = []
              for(let i = 1; i<=8; i++) {
                for(let j = 1; j<=8; j++) {
                  const currentId = ""+j+i
                  if(""+coluna+linha !== currentId)
                    movimentos.push({id:''+j+i})
                }
              }

              // for (let index = 1; index <= qntMovimentos; index++) {
              //     var mov = index*aux
              //     movimentos.push({id: ''+coluna+(linha-mov)})
              //     movimentos.push({id: String(coluna+(index*aux))+String(linha-(index*aux))})
              //     movimentos.push({id: String(coluna-(index*aux))+String(linha-(index*aux))}) 
              //     movimentos.push({id: ''+(coluna - mov)+linha}) 
              //     movimentos.push({id: ''+(coluna + mov)+linha}) 
              //     console.log('movi', movimentos)
              // }

              // for (let index = 1; index <= qntMovimentos; index++) {
              //     var mov = index*aux
              //     movimentos.push({id: ''+coluna+(linha-mov)})  
              // }

              this.mostrarQuadradosDisponiveis(movimentos)
              this.pecaSelecionada = peca
          }, 1)
      },
      mostraOpcoesPeao(peca, linha, coluna){
          var aux = peca.lado === 'Branco' ?  1 : -1
          setTimeout(() => {
              const qntMovimentos = linha === 7 && peca.lado === 'Branco' || linha === 2 && peca.lado === 'Preto' ? 2 : 1
              var movimentos = []

              for (let index = 1; index <= qntMovimentos; index++) {
                  var mov = index*aux
                  if(this.getQuadrado(coluna, linha-mov).quadrado.__vue__.ocupado === true)
                      break
                  movimentos.push({id: ''+coluna+(linha-mov)})  
              }

              if(this.capturaDisponivelPeao(coluna+(1*aux), linha-(1*aux), peca.lado))
                  movimentos.push({id: String(coluna+(1*aux))+String(linha-(1*aux))})
              
              if(this.capturaDisponivelPeao(coluna-(1*aux), linha-(1*aux), peca.lado))
                  movimentos.push({id: String(coluna-(1*aux))+String(linha-(1*aux))})
              console.log('movimento', this.movimentos)

              this.mostrarQuadradosDisponiveis(movimentos)
              this.pecaSelecionada = peca
          }, 1)
      },
      getAdversario(lado) {
          if (lado === 'Branco'){
              return  'Preto'
          }else {
              return 'Branco'
          }
      },
      capturaDisponivelPeao(coluna, linha, lado){
          var adversario = this.getAdversario(lado)

          if(this.getQuadrado(coluna, linha) 
             && this.getQuadrado(coluna, linha).quadrado.__vue__.ocupado === true 
             && this.getQuadrado(coluna, linha).quadrado.__vue__.pecaQuadrado.lado === adversario)
              return true
          return false
      },
      mostrarQuadradosDisponiveis(movimentos) {
          this.movimentos = []
          movimentos.forEach(movimento => {
              this.movimentos.push(String(movimento.id))
          })
      },
      limparQuadrados(){
          this.movimentos = []
          this.pecaSelecionada = {}
      },
      removePecaAtual(){
          this.getQuadrado(this.pecaSelecionada.coluna, this.pecaSelecionada.linha).quadrado.firstChild.remove()
          this.getQuadrado(this.pecaSelecionada.coluna, this.pecaSelecionada.linha).quadrado.__vue__.ocupado = false
          this.getQuadrado(this.pecaSelecionada.coluna, this.pecaSelecionada.linha).quadrado.__vue__.pecaQuadrado = {}
      },
      capturaPeca(coluna, linha){
          if(this.getQuadrado(coluna, linha).quadrado.__vue__.pecaQuadrado.lado === 'Preto'){
              this.pecasMortasPretas.push(this.getQuadrado(coluna, linha).quadrado.__vue__.pecaQuadrado)
          }
          this.getQuadrado(coluna, linha).quadrado.firstChild.remove() 
      },
      isPecaSelecionada(lado){
          return ((this.pecaSelecionada.hasOwnProperty('lado')) && lado != this.pecaSelecionada.lado)
      },
      mudarLado(){
          this.ladoAtual = this.getAdversario(this.ladoAtual)
      },
      isLadoAtual(lado){
          return this.ladoAtual === lado ? false : true
      },
    },
    directives: {
        ClickOutside
    }
}
</script>

<style scoped>
.tabuleiro {
  margin-top: 5rem;
  margin-left: 10rem;
  display: inline-block;
  border: 2px solid black;
}

.linha {
  height: 10vh;
}

.container{
  text-align: center;
  height: 100vh;
  background-color: #312e2b;
}

.container h1{
  font-family: 'Viga', sans-serif;
}

h1 span{
  color: #42b983;
}

.logo{
  margin-left: 5rem;
  height: 10rem;
  width: 10rem;
}

.header {
  justify-content: center;
  align-items: center;
  display: flex;
}

@media (max-width: 768px) {
  .linha {
      height: 8vw;
  }
}

.peca{
  height: 5rem;
  width: auto;
}
</style>



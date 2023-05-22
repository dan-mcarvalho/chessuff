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
                      :movimentacaoDePecas ="movimentacaoDePecas"
                      :movimentos="movimentos" 
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
          pecasMortasPretas: new Array(),
          movimentacaoDePecas: null,
      };
  },
  computed: {
    logoUff() {
       return require(`../assets/UFF_brasão.png`)
    },
  },
  mounted(){
    this.movimentacaoDePecas = {
      mostraOpcoesPeao: this.mostraOpcoesPeao,
      mostraOpcoesCavalo:this.mostraOpcoesCavalo,
      mostraOpcoesRei:this.mostraOpcoesRei,
      mostraOpcoesDama: this.mostraOpcoesDama,     
      mostraOpcoesTorre: this.mostraOpcoesTorre,
      mostraOpcoesBispo: this.mostraOpcoesBispo,
    }
    console.log("funcoes", this.movimentacaoDePecas)

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
      isValido(coluna, linha, lado){
            if(this.getQuadrado(coluna, linha) && this.getQuadrado(coluna, linha).quadrado.__vue__.pecaQuadrado.lado != lado)
                return true
        },
      getQuadrados(movimentos) {
          this.movimentos = []
          movimentos.forEach(movimento => {
              this.movimentos.push(String(movimento.id))
          })
      },
      mostraOpcoesCavalo(peca, linha, coluna){
            setTimeout(() => {
                var movimentos = []

                if(this.isValido(coluna+1, linha-2, peca.lado))
                    movimentos.push({id: String(coluna+1)+String(linha-2)})
                
                if(this.isValido(coluna-1, linha-2, peca.lado))
                    movimentos.push({id: String(coluna-1)+String(linha-2)})

                if(this.isValido(coluna+1, linha+2, peca.lado))
                    movimentos.push({id: String(coluna+1)+String(linha+2)})

                if(this.isValido(coluna-1, linha+2, peca.lado))    
                    movimentos.push({id: String(coluna-1)+String(linha+2)})
                
                if(this.isValido(coluna-2, linha+1, peca.lado))    
                    movimentos.push({id: String(coluna-2)+String(linha+1)})

                if(this.isValido(coluna-2, linha-1, peca.lado))    
                    movimentos.push({id: String(coluna-2)+String(linha-1)})

                if(this.isValido(coluna+2, linha+1, peca.lado))    
                    movimentos.push({id: String(coluna+2)+String(linha+1)})

                if(this.isValido(coluna+2, linha-1, peca.lado))    
                    movimentos.push({id: String(coluna+2)+String(linha-1)})

                this.getQuadrados(movimentos)
                this.pecaSelecionada = peca
            }, 1)
        },
        mostraOpcoesBispo(peca, linha, coluna) {
            setTimeout(() => {
                var movimentos = []

                movimentos = this.percorreDiagonal('+', '-', movimentos, coluna, linha, peca.lado)
                movimentos = this.percorreDiagonal('+', '+', movimentos, coluna, linha, peca.lado)
                movimentos = this.percorreDiagonal('-', '-', movimentos, coluna, linha, peca.lado)
                movimentos = this.percorreDiagonal('-', '+', movimentos, coluna, linha, peca.lado)

                this.getQuadrados(movimentos)
                this.pecaSelecionada = peca
            }, 1 )
        },
        mostraOpcoesTorre(peca, linha, coluna){
            setTimeout(() => {
                var movimentos = []

                movimentos = this.percorreHorizontal('-', movimentos, coluna, linha, peca.lado)
                movimentos = this.percorreHorizontal('+', movimentos, coluna, linha, peca.lado)
                movimentos = this.percorreVertical('+', movimentos, coluna, linha, peca.lado)
                movimentos = this.percorreVertical('-', movimentos, coluna, linha, peca.lado)
                
                this.getQuadrados(movimentos)
                this.pecaSelecionada = peca
            }, 1)
        },
        mostraOpcoesRei(peca, linha, coluna){
            setTimeout(() => {
                var movimentos = []

                for (let i = -1; i <= 1; i++) {
                    for (let j = -1; j <= 1; j++) {
                        if(this.isValido(coluna+i, linha+j, peca.lado))
                            movimentos.push({id: String(coluna+i)+String(linha+j)})
                    }
                    
                }

                this.getQuadrados(movimentos)
                this.pecaSelecionada = peca
            }, 1)
        },
        mostraOpcoesDama(peca, linha, coluna){
            setTimeout(() => {
                var movimentos = []

                movimentos = this.percorreHorizontal('-', movimentos, coluna, linha, peca.lado)
                movimentos = this.percorreHorizontal('+', movimentos, coluna, linha, peca.lado)
                movimentos = this.percorreVertical('+', movimentos, coluna, linha, peca.lado)
                movimentos = this.percorreVertical('-', movimentos, coluna, linha, peca.lado)
                movimentos = this.percorreDiagonal('+', '-', movimentos, coluna, linha, peca.lado)
                movimentos = this.percorreDiagonal('+', '+', movimentos, coluna, linha, peca.lado)
                movimentos = this.percorreDiagonal('-', '-', movimentos, coluna, linha, peca.lado)
                movimentos = this.percorreDiagonal('-', '+', movimentos, coluna, linha, peca.lado)

                this.getQuadrados(movimentos)
                this.pecaSelecionada = peca
            }, 1)
        },
        percorreDiagonal(sinal1, sinal2, movimentos, coluna, linha, lado){
            sinal1 = this.getSinal(sinal1)
            sinal2 = this.getSinal(sinal2)
            for (let i = 1; true; i++) {
                if(!this.isValido(coluna+(i*sinal1), linha+(i*sinal2), lado)) break

                movimentos.push({id: String(coluna+(i*sinal1))+String(linha+(i*sinal2))})
                
                if (this.getQuadrado((coluna+(i*sinal1)), (linha+(i*sinal2))) 
                && this.getQuadrado((coluna+(i*sinal1)), (linha+(i*sinal2))).quadrado.__vue__.pecaQuadrado.lado == this.getAdversario(lado)) break
            }

            return movimentos
        },
        percorreVertical(sinal, movimentos, coluna, linha, lado){
            sinal = this.getSinal(sinal)

            for (let i = 1; true; i++) {
                if(!this.isValido(coluna, linha+(i*sinal), lado)) break
                movimentos.push({id: String(coluna)+String(linha+(i*sinal))}) 
                
                if(this.getQuadrado(coluna, linha+(i*sinal)) 
                && this.getQuadrado(coluna, linha+(i*sinal)).quadrado.__vue__.pecaQuadrado.lado == this.getAdversario(lado))
                    break;
            }

            return movimentos
        },
        percorreHorizontal(sinal, movimentos, coluna, linha, lado){
            sinal = this.getSinal(sinal)

            for (let i = 1; true; i++) {
                if(!this.isValido(coluna+(i*sinal), linha, lado)) break
                movimentos.push({id: String(coluna+(i*sinal))+String(linha)}) 
                
                if(this.getQuadrado(coluna+(i*sinal), linha) 
                && this.getQuadrado(coluna+(i*sinal), linha).quadrado.__vue__.pecaQuadrado.lado == this.getAdversario(lado))
                    break;
            }

            return movimentos
        },
        getSinal(sinal){
            if(sinal === '+'){
                return 1
            } else{
                return -1
            }
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



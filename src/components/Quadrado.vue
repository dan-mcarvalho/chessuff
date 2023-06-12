<template>
  <div v-bind:class="getClass()" v-on:click="moverPeca" ref="quadrado" class="quadrado">
  </div>
</template>

<script>
import Peca from './Peca.vue'
import pecas from '../assets/pecas.js'
import Vue from 'vue'

export default {
  // eslint-disable-next-line vue/multi-word-component-names
  name: "Quadrado",
  props: {
      linha: Number,
      coluna: Number,
      id: String,
      isLadoAtual: Function,
      adicionaPeca: Function,
      adicionaQuadrado: Function,
      removePecaAtual: Function,
      capturaPeca: Function,
      isPecaSelecionada: Function,
      mudarLado: Function,
      movimentos: Array,
      movimentacaoDePecas: Object,
      movimentarPecas: Function,
      pecaSelecionada: Object
  },
  data: () => {
      return {
          ocupado: false,
          pecaQuadrado: {}
      }
  },
  methods:{
      getClass(){
          var classe = (this.linha+this.coluna)%2 == 0 ? 'branco' : 'preto' 
          classe+= this.movimentos.includes(this.id) ? ' selecionado' : ''
          return classe
      },
      mostraOpcoes: function (tipo) {
          switch(tipo) {
              case 'Peao':
                  this.movimentacaoDePecas.mostraOpcoesPeao(this.pecaQuadrado, this.linha, this.coluna)
                  break;
              case 'Cavalo':
                  this.movimentacaoDePecas.mostraOpcoesCavalo(this.pecaQuadrado, this.linha, this.coluna)
                  break;
              case 'Rei':
                  this.movimentacaoDePecas.mostraOpcoesRei(this.pecaQuadrado, this.linha, this.coluna)
                  break;
              case 'Dama':
                  this.movimentacaoDePecas.mostraOpcoesDama(this.pecaQuadrado, this.linha, this.coluna)
                  break;
              case 'Bispo':
                  this.movimentacaoDePecas.mostraOpcoesBispo(this.pecaQuadrado, this.linha, this.coluna)
                  break;
              case 'Torre':
              this.movimentacaoDePecas.mostraOpcoesTorre(this.pecaQuadrado, this.linha, this.coluna)
              break;
          }
      },
      moverPeca(){
          if(Object.entries(this.pecaSelecionada).length === 0 && this.pecaSelecionada.constructor === Object) return
          if(this.movimentos.includes(this.id)){
              this.removePecaAtual()
              if(this.ocupado)
                  this.capturaPeca(this.coluna, this.linha)
              var ComponentClass = Vue.extend(Peca)
              let tipo = this.pecaSelecionada.tipo
              let lado = this.pecaSelecionada.lado
              var instance = new ComponentClass({
                  propsData: { tipo: tipo,
                               lado: lado,
                               mostraOpcoes: this.mostraOpcoes,
                               isPecaSelecionada: this.isPecaSelecionada,
                               isLadoAtual: this.isLadoAtual}
              })
              instance.$mount()
              this.$refs.quadrado.appendChild(instance.$el)
              this.pecaQuadrado = {tipo: this.pecaSelecionada.tipo,
                                   lado: this.pecaSelecionada.lado,
                                   linha: this.linha,
                                   coluna: this.coluna}
              this.ocupado = true
              this.mudarLado()
          }
          if(isComputador) {
            
          }
      }
  },
  mounted() {
      pecas.map(peca => {
          if(peca.linha === this.linha && peca.coluna === this.coluna){
              var ComponentClass = Vue.extend(Peca)
              var instance = new ComponentClass({
                  propsData: { tipo: peca.tipo,
                               lado: peca.lado,
                               mostraOpcoes: this.mostraOpcoes,
                               isPecaSelecionada: this.isPecaSelecionada,
                               isLadoAtual: this.isLadoAtual}
              })
              instance.$mount()
              this.$refs.quadrado.appendChild(instance.$el)
              this.adicionaPeca(this.linha, this.coluna, peca.tipo)
              this.adicionaQuadrado(this.id, this.linha, this.coluna, true, this.$refs.quadrado)
              this.ocupado = true
              this.pecaQuadrado = peca
          }
      });
      if(this.ocupado === false)
          this.adicionaQuadrado(this.id, this.linha, this.coluna, false, this.$refs.quadrado)
  }
}
</script>


<style scoped>
.quadrado {
  height: 10vh;
  width: 10vh;
  display: inline-block;
  position: relative;
  transition: all .3s;
}

.preto{
  background-color: #769656;
}

.branco {
  background-color: #eeeed2;
}

.quadrado h1{
  color: orange;
  font-size: 4rem;
  margin-top: 3rem;
  position: absolute;
}

.selecionado {
  border: solid 2px red;
  cursor: pointer;
}

@media (max-width: 768px) {
  .quadrado {
      height: 8vw;
      width: 8vw;
  }
}
</style>

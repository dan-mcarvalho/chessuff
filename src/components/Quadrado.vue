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
      isComputador: Boolean,
      id: String,
      limparMovimentosIa:Function,
      getLadoAtual:Function,
      isLadoAtual: Function,
      adicionaPeca: Function,
      adicionaQuadrado: Function,
      removePecaAtual: Function,
      capturaPeca: Function,
      isPecaSelecionada: Function,
      mudarLado: Function,
      openModal: Function,
      movimentos: Array,
      quadrados: Map,
      movimentacaoDePecas: Object,
      movimentarPecas: Function,
      getMovimentosIa: Function,
      pecaSelecionada: Object
  },
  data: () => {
      return {
          ocupado: false,
          pecaQuadrado: {},
      }
  },
  methods:{
      getClass(){
          var classe = (this.linha+this.coluna)%2 == 0 ? 'branco' : 'preto' 
          classe+= this.movimentos.includes(this.id) ? ' selecionado' : ''
          return classe
      },
      mostraOpcoes: function (tipo, peca=this.pecaQuadrado, linha=this.linha, coluna=this.coluna) {
          switch(tipo) {
              case 'Peao':
                  return this.movimentacaoDePecas.mostraOpcoesPeao(peca, linha, coluna)
              case 'Cavalo':
                  return this.movimentacaoDePecas.mostraOpcoesCavalo(peca, linha, coluna)
              case 'Rei':
                  return this.movimentacaoDePecas.mostraOpcoesRei(peca, linha, coluna)
              case 'Dama':
                  return this.movimentacaoDePecas.mostraOpcoesDama(peca, linha, coluna)
              case 'Bispo':
                  return this.movimentacaoDePecas.mostraOpcoesBispo(peca, linha, coluna)
              case 'Torre':
                  return this.movimentacaoDePecas.mostraOpcoesTorre(peca, linha, coluna)
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
              if((this.pecaSelecionada.tipo === 'Peao') 
                    && (this.pecaSelecionada.lado === 'Branco' && this.linha === 1 
                    || this.pecaSelecionada.lado === 'Preto' && this.linha === 8))
                    this.openModal(this.coluna, this.linha, this.pecaSelecionada.lado)
              this.mudarLado()
          }
          if(this.isComputador && this.getLadoAtual() === "Preto") {
            const possiveisPecas = [];
            for (const value of this.quadrados.values()) {
                if(value.quadrado.__vue__.pecaQuadrado.lado === "Preto") {
                    const pecaAtual = value.quadrado.__vue__.pecaQuadrado
                    possiveisPecas.push(pecaAtual);
                }      
            }
              this.jogadaIA(possiveisPecas);
          }
      },
      jogadaIA(pecas){
          let randomIndex = Math.floor(Math.random() * pecas.length);
          let peca = pecas[randomIndex]
          this.mostraOpcoes(peca.tipo, peca, peca.linha, peca.coluna)
          let movimentos = this.movimentos;
          setTimeout(() => {
            let randomMovimentoIdx = Math.floor(Math.random() * movimentos.length);
            if(movimentos.length > 0 && this.quadrados.get(this.movimentos[randomMovimentoIdx]) !== undefined) {
              const quadrado = this.quadrados.get(this.movimentos[randomMovimentoIdx]).quadrado.__vue__.$refs.quadrado;
              quadrado.dispatchEvent(new Event('click'));
              this.limparMovimentosIa();
              return;
            } else {
              this.jogadaIA(pecas)
              this.limparMovimentosIa();
              return;
            }
            }, 10);
      },
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
  background-color: rgb(100, 100, 100);
  cursor: pointer;
}

@media (max-width: 768px) {
  .quadrado {
      height: 8vw;
      width: 8vw;
  }
}
</style>

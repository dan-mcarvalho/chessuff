<template>
  <img v-on:click="selecionarPeca" v-click-outside="deselecionarPeca" v-bind:class="isSelecionada()" class="peca" :src="pecaImage">
</template>

<script>
import ClickOutside from 'vue-click-outside'

export default {
  // eslint-disable-next-line vue/multi-word-component-names
  name: "Peca",
  props: {
      tipo: String,
      lado: String,
      ladoAtual: String,
      mostraOpcoes: Function,
      limparQuadrados: Function,
      isPecaSelecionada: Function,
      isLadoAtual: Function

  },
  data: () => {
      return {
          selecionada: false
      }
  },
  computed:{
    pecaImage() {
      return require(`../assets/pecas/${this.tipo+this.lado}.png`) 
    }
  },
  methods:{
      selecionarPeca(){
          if(this.isPecaSelecionada(this.lado) || this.isLadoAtual(this.lado)) return
          this.selecionada = !this.selecionada;
          if(this.selecionada===true)
              this.mostraOpcoes(this.tipo)
      },
      isSelecionada(){
          if(this.selecionada === true){
              return 'selecionada';
          }
      },
      deselecionarPeca() {
          this.selecionada = false
      }
  },
  directives: {
      ClickOutside
  }
}
</script>

<style scoped>
.peca{
  height: 65%;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  position: absolute;
  transition: all .2s;
}

.peca:hover{
  cursor: pointer;
}

.selecionada {
  height: 75%;
}
</style>


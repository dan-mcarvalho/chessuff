<template>
  <transition name="modal">
      <div class="modal-mask">
        <div class="modal-wrapper">
          <div class="modal-container">

            <div class="modal-header">
              <slot name="header">
                Selecione o modo de jogo
              </slot>
            </div>

            <div class="modal-body">
              <slot name="body">
                <br> <br>
                <div  class="peca">
                  <button v-on:click="umJogador" >
                    Um jogador
                  </button>
                </div>
                <br>
                <div  class="peca">
                  <button v-on:click="doisJogadores" >
                    Dois jogadores
                  </button>
                </div>
              </slot>
            </div>
          </div>
        </div>
      </div>
  </transition>
</template>

<script>

export default {
  name: "modal-menu",
  props: {
    fecharModal: Function
  },
  data: () => {
      return {
          pecaSelecionada: ''
      }
  },
  methods: {
    umJogador(){
      //TODO: Chamar parte do programa que inicia o jogo com uma I.A
      this.fecharModal()
    },
    doisJogadores(){
      this.fecharModal()
    }
  }
}
</script>

<style scoped>

.modal-mask {
position: fixed;
z-index: 9998;
top: 0;
left: 0;
width: 100%;
height: 100%;
background-color: rgba(0, 0, 0, .5);
display: table;
transition: opacity .3s ease;
}

.modal-wrapper {
display: table-cell;
vertical-align: middle;
}

.modal-container {
width: 55rem;
margin: 0px auto;
padding: 2rem 3rem;
background-color: #fff;
border-radius: .2rem;
box-shadow: 0 .2rem .8rem rgba(0, 0, 0, .33);
transition: all .3s ease;
font-family: 'Viga', sans-serif;
height: 22rem;
}

.modal-header {
margin-top: 0;
color: #42b983;
font-size: 1.8rem;
}

.modal-body {
margin: 2rem 0;
}

.modal-enter {
opacity: 0;
}

.modal-leave-active {
opacity: 0;
}

.modal-enter .modal-container,
.modal-leave-active .modal-container {
-webkit-transform: scale(1.1);
transform: scale(1.1);
}

.peca:hover{
  transform: scale(1.1);
}


</style>


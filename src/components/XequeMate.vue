<template>
  <transition name="modal">
      <div class="modal-mask">
        <div class="modal-wrapper">
          <div class="modal-container">

            <div class="modal-header">
              <slot name="header">
                Xeque Mate!
              </slot>
            </div>
            <div class="modal-header">
              <slot name="header">
                {{  `${ ladoAtual === 'Preto'? 'Brancas': 'Pretas' } vencem`  }}
              </slot>
            </div>
            
            <div class="modal-body">
              <slot name="body">
                <br> <br>
                <div  class="peca">
                  <button v-on:click="confirmar" >
                    ok
                  </button>
                </div>
                <br>
              </slot>
            </div>
          </div>
        </div>
      </div>
  </transition>
</template>

<script>

export default {
  name: "xeque-mate",
  props: {
    ladoAtual: String,
  },
  data: () => {
      return {
          pecaSelecionada: ''
      }
  },
  methods: {
    confirmar(){
      location.reload();
    },
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


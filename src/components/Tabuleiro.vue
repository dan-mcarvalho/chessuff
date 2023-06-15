<template>
  <div>
      <div class="container"> 
        <div class="header">
          <h1 style="font-family: 'sans-serif'; font-size:50px; font-weight: bold; color: white;"> Chessuff</h1>
          <div ><img class="logo" :src="logoUff"></div>
        </div>
          <div class="tabuleiro" v-click-outside="limparQuadrados" v-on:click="limparQuadrados" >
              <div v-for="linha in linhas" v-bind:key="linha" class="linha">
                  <Quadrado 
                      :movimentacaoDePecas ="movimentacaoDePecas"
                      :movimentos="movimentos"
                      :getMovimentosIa="getMovimentosIa"
                      :limparMovimentosIa="limparMovimentosIa"
                      :quadrados="quadrados"
                      :isComputador="isComputador"
                      :adicionaPeca="adicionaPeca" 
                      :adicionaQuadrado="adicionaQuadrado"
                      :removePecaAtual="removePecaAtual"
                      :isPecaSelecionada="isPecaSelecionada"
                      :openModal="openModal"
                      :capturaPeca="capturaPeca"
                      :mudarLado="mudarLado"
                      :linha="linha"
                      :coluna="coluna"
                      :isLadoAtual="isLadoAtual"
                      :getLadoAtual ="getLadoAtual"
                      :pecaSelecionada="pecaSelecionada"
                      :movimentarPecas="movimentarPecas"
                      :ataqueDePecas ="ataqueDePecas"
                      :id="''+coluna+linha" 
                      v-for="coluna in colunas" 
                      v-bind:key="coluna"
                  />
               </div>
               <Menu v-if="showMenu" 
                  @close="showMenu=false"
                    :fecharMenu="fecharMenu"
                    :setModoDeJogo = "setModoDeJogo"
                    />
                <Modal v-if="showModal" 
                    @close="showModal=false"
                    :linha="linhaModal"
                    :coluna="colunaModal"
                    :lado="ladoModal"
                    :evoluirPeca="evoluirPeca"/>
                <XequeMate :ladoAtual="this.ladoAtual"  v-if="isXequeMate"> </XequeMate>                  
          </div>
      </div>
  </div>
</template>
<script>
import Quadrado from './Quadrado.vue'
import ClickOutside from 'vue-click-outside'
import Modal from './Modal.vue'
import XequeMate from './XequeMate.vue'
import Menu from './Menu.vue'

export default {
  // eslint-disable-next-line vue/multi-word-component-names
  name: "Tabuleiro",
  components: {
      Quadrado, Menu, Modal, XequeMate
  },
  data: () => {
      return {
          linhas: [1, 2, 3, 4, 5, 6, 7, 8],
          colunas: [1, 2, 3, 4, 5, 6, 7, 8],
          pecas: [],
          quadrados: new Map(),
          movimentos: [],
          isXeque:false,
          isXequeMate: false,
          ataqueDePecas:null,
          posicaoReiEmXeque: null,
          posicaoAtacanteRei:null,
          areasDeAtaque: [],
          pecaSelecionada: {},
          ladoAtual: 'Branco',
          isComputador: false,
          showMenu: true,
          showModal: false,
          linhaModal: '',
          colunaModal: '',
          ladoModal: '',
          pecasMortasPretas: new Array(),
          movimentacaoDePecas: null,
          propriedadesIA:null
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
    },
    this.ataqueDePecas = {
      ataquePeao: this.ataquePeao,
      ataqueCavalo:this.ataqueCavalo,
      ataqueDama: this.ataqueDama,     
      ataqueTorre: this.ataqueTorre,
      ataqueBispo: this.ataqueBispo,
    }
  },  
  methods: {
        verificarXeque(){
          this.areasDeAtaque.forEach((e)=>{
            const quadradoAtacado = this.quadrados.get(e.id);
            const quadradoAtacadoProps = quadradoAtacado.quadrado.__vue__;
            if(quadradoAtacadoProps.ocupado) {
              const pecaAtual = quadradoAtacadoProps.pecaQuadrado;
              if(pecaAtual.tipo === "Rei") {
                this.isXeque = true;
                this.posicaoReiEmXeque = pecaAtual;
              } 
            }
          });
        },
        ataquePeao(peca, linha, coluna, apenasChecar = false, bloqueioPeao=false, suporte = false) {
          if(bloqueioPeao) {
            var aux = peca.lado === 'Branco' ?  1 : -1
              const qntMovimentos = linha === 7 && peca.lado === 'Branco' || linha === 2 && peca.lado === 'Preto' ? 2 : 1
              var movimentos = []

              for (let index = 1; index <= qntMovimentos; index++) {
                  var mov = index*aux
                  if(this.getQuadrado(coluna, linha-mov).quadrado.__vue__.ocupado === true)
                      break
                  movimentos.push({id: ''+coluna+(linha-mov)})  
              }

            return movimentos;
          }
          var aux = peca.lado === 'Branco' ?  1 : -1
          var ataques = []

          if(this.getQuadrado(coluna+(1*aux), linha-(1*aux)))
            ataques.push({id: String(coluna+(1*aux))+String(linha-(1*aux))}) 
          if(this.getQuadrado(coluna-(1*aux), linha-(1*aux)))           
            ataques.push({id: String(coluna-(1*aux))+String(linha-(1*aux))})
          
          if(apenasChecar) return ataques;

          this.areasDeAtaque = ataques;
          this.posicaoAtacanteRei = peca;
          this.verificarXeque()
          return ataques;
          },
        ataqueCavalo(peca,linha,coluna, apenasChecar = false, suporte = false) {
          var ataques = []
          let auxLado = peca.lado
          if(suporte) auxLado = this.getAdversario(peca.lado);

          if(this.isValido(coluna+1, linha-2, auxLado))
            ataques.push({id: String(coluna+1)+String(linha-2)})
          
          if(this.isValido(coluna-1, linha-2, auxLado))
              ataques.push({id: String(coluna-1)+String(linha-2)})

          if(this.isValido(coluna+1, linha+2, auxLado))
              ataques.push({id: String(coluna+1)+String(linha+2)})

          if(this.isValido(coluna-1, linha+2, auxLado))    
              ataques.push({id: String(coluna-1)+String(linha+2)})
          
          if(this.isValido(coluna-2, linha+1, auxLado))    
              ataques.push({id: String(coluna-2)+String(linha+1)})

          if(this.isValido(coluna-2, linha-1, auxLado))    
              ataques.push({id: String(coluna-2)+String(linha-1)})

          if(this.isValido(coluna+2, linha+1, auxLado))    
              ataques.push({id: String(coluna+2)+String(linha+1)})

          if(this.isValido(coluna+2, linha-1, auxLado))    
              ataques.push({id: String(coluna+2)+String(linha-1)})

          if(apenasChecar) return ataques;
          
          this.areasDeAtaque = ataques;
          this.posicaoAtacanteRei = peca;
          this.verificarXeque()
          return ataques;
        },

        movimentosPossiveisRei(peca,linha,coluna, apenasChecar = false, suporte = false) {
          var movimentos = []
          for (let i = -1; i <= 1; i++) {
              for (let j = -1; j <= 1; j++) {
                  if(this.isValido(coluna+i, linha+j, peca.lado))
                  movimentos.push({id: String(coluna+i)+String(linha+j)})
              }
          }
          return movimentos;
        },
        ataqueDama(peca,linha,coluna, apenasChecar = false, suporte) {
          var ataques = []
          let auxLado = peca.lado
          if(suporte) auxLado = this.getAdversario(peca.lado);
          ataques = this.percorreHorizontal('-', ataques, coluna, linha, auxLado)
          ataques = this.percorreHorizontal('+', ataques, coluna, linha, auxLado)
          ataques = this.percorreVertical('+', ataques, coluna, linha, auxLado)
          ataques = this.percorreVertical('-', ataques, coluna, linha, auxLado)
          ataques = this.percorreDiagonal('+', '-', ataques, coluna, linha, auxLado)
          ataques = this.percorreDiagonal('+', '+', ataques, coluna, linha, auxLado)
          ataques = this.percorreDiagonal('-', '-', ataques, coluna, linha, auxLado)
          ataques = this.percorreDiagonal('-', '+', ataques, coluna, linha, auxLado)

          if(apenasChecar) return ataques;

          this.areasDeAtaque = ataques;
          this.posicaoAtacanteRei = peca;
          this.verificarXeque()
          return ataques;
        },
        ataqueBispo(peca,linha,coluna, apenasChecar= false, suporte = false) {
          var ataques = []
          let auxLado = peca.lado
          if(suporte) auxLado = this.getAdversario(peca.lado);

          ataques = this.percorreDiagonal('+', '-', ataques, coluna, linha, auxLado)
          ataques = this.percorreDiagonal('+', '+', ataques, coluna, linha, auxLado)
          ataques = this.percorreDiagonal('-', '-', ataques, coluna, linha, auxLado)
          ataques = this.percorreDiagonal('-', '+', ataques, coluna, linha, auxLado)

          if(apenasChecar) return ataques;

          this.areasDeAtaque = ataques;
          this.posicaoAtacanteRei = peca;
          this.verificarXeque()
          return ataques;
        },
        ataqueTorre(peca,linha,coluna, apenasChecar= false, suporte = false) {
          var ataques = []
          let auxLado = peca.lado
          if(suporte) auxLado = this.getAdversario(peca.lado);

          ataques = this.percorreHorizontal('-', ataques, coluna, linha, auxLado)
          ataques = this.percorreHorizontal('+', ataques, coluna, linha, auxLado)
          ataques = this.percorreVertical('+', ataques, coluna, linha, auxLado)
          ataques = this.percorreVertical('-', ataques, coluna, linha, auxLado)
          
          if(apenasChecar) return ataques;

          this.areasDeAtaque = ataques;
          this.posicaoAtacanteRei = peca;
          this.verificarXeque()
          return ataques;
        },

      getQuadrado(coluna, linha){
          coluna = String(coluna)
          linha = String(linha)
          const id = coluna+linha
          return this.quadrados.get(id)
      },
      setModoDeJogo(mode){
        this.isComputador = mode;
      },
      getLadoAtual(){
        return this.ladoAtual;
      },
      limparMovimentosIa(){
        this.movimentos = [];
      },
      getMovimentosIa(){
        return this.movimentos;
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

              this.mostrarQuadradosDisponiveis(movimentos)
              this.pecaSelecionada = peca 
          }, 1)
          return this.movimentos;
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
            return this.movimentos;
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
          return this.movimentos;
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
          return this.movimentos;
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
          return this.movimentos;
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
          return this.movimentos;
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
          if(this.isXeque) {
            this.verificarFimDeJogo();
          }
          this.ladoAtual = this.getAdversario(this.ladoAtual)
          this.pecaSelecionada = {};
      },
      verificarFimDeJogo(){
        const solucoesDeXeque = [];
        solucoesDeXeque.push(
          this.reiPodeSeMover(),
          this.capturarAtacanteDoRei(),
          this.pecaCobreRei(),
        )
        const isXequeMate = solucoesDeXeque.every((e)=> e === false)
        if(isXequeMate) {
          this.isXequeMate = true;
        }
      },
      reiPodeSeMover() {
        const rei = this.posicaoReiEmXeque;
        const quadradoAtacanteRei = String(this.posicaoAtacanteRei.coluna) + this.posicaoAtacanteRei.linha
        const atacanteDefendido = this.verificarAtacanteDefendido();

        let areasDeAtaque = this.areasDeAtaque.map((e) => e.id)
        let movimentos = this.movimentosPossiveisRei(rei, rei.linha, rei.coluna)
        const movimentosPossiveis = movimentos.filter((e)=>  {
          if(e.id === quadradoAtacanteRei && atacanteDefendido) return false
          if(!areasDeAtaque.includes(e.id))
            return true
        });
        if(movimentosPossiveis.length > 0) return true;
        return false;
      },
      verificarAtacanteDefendido() {
        const possiveisPecas = [];
        let atacanteDefendido = false;
        const quadradoAtacanteRei = String(this.posicaoAtacanteRei.coluna) + this.posicaoAtacanteRei.linha
        for (const value of this.quadrados.values()) {
            if(value.quadrado.__vue__.pecaQuadrado.lado === this.ladoAtual) {
                const pecaAtual = value.quadrado.__vue__.pecaQuadrado
                possiveisPecas.push(pecaAtual);
            }      
        }
        for(let i = 0; i < possiveisPecas.length; i++) {
          const peca = possiveisPecas[i];
          if(peca.tipo !== "Rei") {
            const areasDefendidas = this.mostrarAtaques(peca, false, true).map((e)=>e.id);
            if(areasDefendidas.includes(quadradoAtacanteRei)) {
              atacanteDefendido = true;
              break;
            }
          } else if (peca.tipo === "Rei"){
            const areasDefendidas = this.movimentosPossiveisRei(peca, peca.linha, peca.coluna)
            if(areasDefendidas.includes(quadradoAtacanteRei)) {
              atacanteDefendido = true;
              break;
            }
          }
        }
        return atacanteDefendido;
      },
      capturarAtacanteDoRei(){
        const possiveisPecas = [];
        let existeCapturaPossivel = false;
        const quadradoAtacanteRei = String(this.posicaoAtacanteRei.coluna) + this.posicaoAtacanteRei.linha
            for (const value of this.quadrados.values()) {
                if(value.quadrado.__vue__.pecaQuadrado.lado === this.getAdversario(this.ladoAtual)) {
                    const pecaAtual = value.quadrado.__vue__.pecaQuadrado
                    possiveisPecas.push(pecaAtual);
                }      
            }
        for(let i = 0; i < possiveisPecas.length; i++) {
          const peca = possiveisPecas[i];
          if(peca.tipo !== "Rei") {
            const possiveisCapturas = this.mostrarAtaques(peca).map((e)=>e.id);
            if(possiveisCapturas.includes(quadradoAtacanteRei)) {
              existeCapturaPossivel = true;
              break;
            }
          }
        }
        return existeCapturaPossivel;
      },
      pecaCobreRei() {
        let caminhoDoAtaque;
        let pecaPodeCobrirRei = false;
        const pecaAtacante = this.posicaoAtacanteRei;
        const rei = this.posicaoReiEmXeque;

        if(pecaAtacante.tipo === "Cavalo" || pecaAtacante.tipo === "Peao"){
          return false;
        }

        if (pecaAtacante.tipo === "Torre") {
          caminhoDoAtaque = this.caminhoAtaqueTorre();
        } 

        if (pecaAtacante.tipo === "Bispo") {
          caminhoDoAtaque = this.caminhoAtaqueBispo();
        }

        if (pecaAtacante.tipo === "Dama") {
          if(rei.linha === pecaAtacante.linha || rei.coluna === pecaAtacante.coluna) 
            caminhoDoAtaque = this.caminhoAtaqueTorre();
          else caminhoDoAtaque = this.caminhoAtaqueBispo();
        }

        for (const value of this.quadrados.values()) {
                if(value.quadrado.__vue__.pecaQuadrado.tipo !== "Rei" && value.quadrado.__vue__.pecaQuadrado.lado === this.getAdversario(this.ladoAtual)) {
                    let casasDeatuacao;
                    const pecaAtual = value.quadrado.__vue__.pecaQuadrado
                    if(pecaAtual.tipo === "Peao") casasDeatuacao = this.mostrarAtaques(pecaAtual, true);
                    else casasDeatuacao = this.mostrarAtaques(pecaAtual);
                    const filterCasasDeatuacao = casasDeatuacao.map(e => e.id)
                    filterCasasDeatuacao.forEach((e)=> {
                      if(caminhoDoAtaque.includes(e)) {
                        pecaPodeCobrirRei = true;
                      }
                    })
                    if(pecaPodeCobrirRei) break;
                }      
        }
        return pecaPodeCobrirRei;
      },
      caminhoAtaqueTorre() {
        const caminhoDoAtaque = [];
        const pecaAtacante = this.posicaoAtacanteRei;
        const rei = this.posicaoReiEmXeque;

        if(pecaAtacante.linha === rei.linha) {
            this.areasDeAtaque.forEach((e)=> {
                const coluna = parseInt(e.id.charAt(0))
                const linha = parseInt(e.id.charAt(1))
                const condition = rei.coluna > pecaAtacante.coluna? coluna > pecaAtacante.coluna :  coluna < pecaAtacante.coluna
                if(linha === pecaAtacante.linha && condition)
                  caminhoDoAtaque.push(e.id);
              });
          } else {
              this.areasDeAtaque.forEach((e)=> {
                const coluna = parseInt(e.id.charAt(0))
                const linha = parseInt(e.id.charAt(1))
                const condition = rei.linha < pecaAtacante.linha? linha < pecaAtacante.linha :  linha > pecaAtacante.linha
                if(coluna === pecaAtacante.coluna && condition)
                  caminhoDoAtaque.push(e.id);
              });
          }
          return caminhoDoAtaque;
      },
      caminhoAtaqueBispo(){
        const caminhoDoAtaque = [];
        const pecaAtacante = this.posicaoAtacanteRei;
        const rei = this.posicaoReiEmXeque;

        if(rei.linha < pecaAtacante.linha && rei.coluna < pecaAtacante.coluna) {
            this.areasDeAtaque.forEach((e)=> {
                const coluna = parseInt(e.id.charAt(0))
                const linha = parseInt(e.id.charAt(1))
                if(coluna < pecaAtacante.coluna && linha < pecaAtacante.linha) {
                  caminhoDoAtaque.push(e.id);
                }
            });
          }
          if(rei.linha < pecaAtacante.linha && rei.coluna > pecaAtacante.coluna) {
            this.areasDeAtaque.forEach((e)=> {
                const coluna = parseInt(e.id.charAt(0))
                const linha = parseInt(e.id.charAt(1))
                if(linha < pecaAtacante.linha && coluna > pecaAtacante.coluna) {
                  caminhoDoAtaque.push(e.id);
                }
            });
          }
          if(rei.linha > pecaAtacante.linha && rei.coluna > pecaAtacante.coluna) {
            this.areasDeAtaque.forEach((e)=> {
                const coluna = parseInt(e.id.charAt(0))
                const linha = parseInt(e.id.charAt(1))
                if(coluna > pecaAtacante.coluna && linha > pecaAtacante.linha) {
                  caminhoDoAtaque.push(e.id);
                }
            });
          }
          if(rei.linha > pecaAtacante.linha && rei.coluna < pecaAtacante.coluna) {
            this.areasDeAtaque.forEach((e)=> {
                const coluna = parseInt(e.id.charAt(0))
                const linha = parseInt(e.id.charAt(1))
                if(linha > pecaAtacante.linha && coluna < pecaAtacante.coluna) {
                  caminhoDoAtaque.push(e.id);
                }
            });
          }
          return caminhoDoAtaque;
      },
      mostrarAtaques(peca, bloqueioPeao, suporte = false) {
        const tipo = peca.tipo;
        const linha = peca.linha;
        const coluna = peca.coluna;
        switch(tipo) {
              case 'Peao':
                  return this.ataqueDePecas.ataquePeao(peca, linha, coluna, true, bloqueioPeao, suporte)
              case 'Cavalo':
                  return this.ataqueDePecas.ataqueCavalo(peca, linha, coluna, true, suporte)
              case 'Dama':
                  return this.ataqueDePecas.ataqueDama(peca, linha, coluna, true, suporte)
              case 'Bispo':
                  return this.ataqueDePecas.ataqueBispo(peca, linha, coluna, true, suporte)
              case 'Torre':
                  return this.ataqueDePecas.ataqueTorre(peca, linha, coluna, true, suporte)
          }
      },
      isLadoAtual(lado){
          return this.ladoAtual === lado ? false : true
      },
      fecharMenu(){
          this.showMenu = false
      },
      openModal(coluna, linha, lado){
            this.showModal = true;
            this.colunaModal = coluna;
            this.linhaModal = linha;
            this.ladoModal = lado
        },
        evoluirPeca(peca, coluna, linha){
            this.showModal = false
            this.getQuadrado(coluna, linha).quadrado.__vue__.pecaQuadrado.tipo = peca
            this.getQuadrado(coluna, linha).quadrado.firstChild.__vue__.tipo = peca
        },
    },
    directives: {
        ClickOutside
    }
}
</script>

<style scoped>
.tabuleiro {
  margin-top: 2rem;
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



<template>
    <div class="container">
      
        <div class="wave">
<!--           Imagens de tamanho grande que alternam entre pastel e suco.
 -->
            <img v-if="bebida != true" src="img/pastel/pastel-paralax.png" class="pastelGrande" alt="Imagem de um pastel grande">

<!--             Logo principal
 -->            <img  src="img/principal/Logo.svg" class="logo" alt="Imagem do logo escrito Pastel de Ideias">
 <!--           Imagens de tamanho pequeno que alternam entre pastel e suco.
 -->
            <img v-if="bebida != true" src="img/pastel/pasteis-img.png" class="pastelDir" alt="Imagem de dois pasteis a direita da tela.">
            <img v-if="bebida != true" src="img/pastel/pasteis-img.png" class="pastelEsq" alt="Imagem de dois pasteis a esquerda da tela.">
            <img v-if="bebida == true" src="img/suco/suco-de-goiaba.png" class="sucoEsq " alt="Imagem de um copo de suco a direita da tela.">
            <img v-if="bebida == true" src="img/suco/suco-de-goiaba.png" class="sucoDir" alt="Imagem de um copo de suco a esquerda da tela.">

<!--             Formulário para envio dos dados do cardápio.
 -->
            <form id="pastel-form" @submit="createCardapio">
                <div class="topPedido">
                    <span class="title">Monte aqui o seu cardápio. O que está esperando?</span>
<!--                     Botão toggle para alternar entre comida e suco.
 -->
                    <span class="comida">Comida</span>
                    <div class="toggle" id="toggle">
                        <input  type="checkbox" id="foo" name="bebida" v-model="bebida" class="btnToggle" />
                        <label for="foo" ></label>
                    </div>
                    <label for="foo"></label>
                    <span class="bebida">Bebida</span>
<!--                     Dados obrigatórios
 -->
                    <label for="tituloPedido"></label>
                    <input class="tPedido" id="tPedido" name="tituloDoPedido" v-model="tituloPedido"  type="text" placeholder="Título do pedido" required>
                    <label for="sabor"></label>
                    <input class="sabor" id="sabor" name="saborProduto" v-model="sabor" type="text" placeholder="Sabor" required>
                    <label for="preco"></label>
                    <input class="preco " id="preco" name="precoCardapio" v-model="preco" type="number" min="1" step="any" placeholder="R$" required>
                </div>
                <div class="Bpedido">
<!--                   Dados opcionais
 -->                    <div>
                        <input class="descricao" id="descricao" name="descricao" v-model="descricao"  type="text" placeholder="Descrição">
<!--                         Dropzone para upload de imagem
 -->                   <div class="upload" id="upload" name="upload" @dragenter.prevent="toggleActive"
                        @dragleave.prevent="toggleActive"
                        @dragover.prevent
                        @drop.prevent="drop"
                        @change="selectedFile"
                        :class="{ 'active-dropzone': active}">
                        <input class="dropzoneFile" type="file"  id="dropzoneFile" />                           

                        <label for="dropzoneFile"><img class="imgUpload" for="dropzoneFile" src="img/principal/upload_image.png" alt="Imagem de upload"></label>
                          <span for="dropzoneFile">Jogue aqui o arquivo de imagem do seu pastel ou clique para localizar a pasta.</span>
                      <!-- Nome do arquivo carregado.-->
                          <p class="uploadR">File: {{ dropzoneFile.name }}</p>
                        </div>
                    </div>
                    <!-- Botão de reset e submit -->
                    <input class="limpar"  type="reset" value="LIMPAR">
                    <input type="submit" class="cadastrar" value="CADASTRAR">
                </div>
                <!-- Mensagem que aparece após envio do formulário
 -->
                 <Message :msg="msg" v-show="msg" />
             </form>
<!--                   Cardápios cadastrados.
 -->          <div class="cardapio1" v-for="cardapio in cardapios" :key="cardapio.id">
                <hr>
                <p><img v-if="cardapio.bebida != true" src="img/pastel/pastelmodal-img.png" class="produtolModal">
                    <img v-if="cardapio.bebida == true" src="img/suco/suco-img.png" class="produtolModal"></p>
                <div class="content" name="titulo" id="titulo" >
                    <span  class="tituloModal">"{{ cardapio.tituloPedido }}"</span>
                    <span class="precoModal">R$ {{ cardapio.preco }}</span>
                </div>
                <p class="descricsabor"><span class="tSabor">Sabor: </span>
                <span class="produto"> {{ cardapio.sabor }} </span></p>
                <p class="descricaoProduto"><span class="tDescri">Descrição: </span><span class="descriProduto">{{ cardapio.descricao }}</span></p>
                <button class="limpar" @click="deleteCardapio(cardapio.id)">EXCLUIR CARDAPIO</button>
            </div>
        </div>
    </div>
</template>



<script>
import Message from './Message';
import { ref } from 'vue';



export default{
    name: "Form",

/*     Codificação para upload em dropzone
 */   setup(){
        const active = ref(false);
        const toggleActive = () => {
          active.value = !active.value;
        };
        let dropzoneFile = ref("");
        const drop = (e) => {
          dropzoneFile.value = e.dataTransfer.files[0]
        };

        const selectedFile = () =>{
        dropzoneFile.value = document.querySelector('.dropzoneFile').files[0]
        };

        return { dropzoneFile, drop, active, toggleActive, selectedFile}
    },

    
    data(){
        return {
          cardapios: null,
          tituloPedido: null,
          sabor: null,
          preco: null,
          descricao: null,
          bebida: null,
          msg: null,
          id: null
         
        }
    },

/*     Consulta (GET) no banco de dados(JSON)
 */    
      methods: {
      async getCardapios(){
        const req = await fetch("http://localhost:3000/cardapios");
        const data = await req.json();

        this.cardapios = data;    
      }, 

/* Método POST para gravar formulário no banco de dados
 */   async createCardapio(e){       

         e.preventDefault()

          const data = {
            tituloPedido: this.tituloPedido,
            sabor: this.sabor,
            preco: this.preco,
            descricao: this.descricao,
            bebida: this.bebida
          }

        const dataJson = JSON.stringify(data)  

        const req = await fetch("http://localhost:3000/cardapios", {
          method: "POST",
          headers: { "Content-Type" : "application/json" },
          body: dataJson
        });

        const res = await req.json()

        console.log(res)

        this.msg = "Cardápio cadastrado com sucesso!"

        // clear message
        setTimeout(() => this.msg = "", 3000)
      
        // limpar campos
        this.tituloPedido = "",
        this.sabor = "",
        this.preco = "",
        this.descricao = "",
        this.bebida = ""
        this.dropzoneFile = ""


        this.getCardapios();
      },
/* Método DELETE para deletar cardápios do banco de dados
 */        
    async deleteCardapio(cardapios) {

      const req = await fetch(`http://localhost:3000/cardapios/${cardapios}`, {
        method: "DELETE"
      });

        const res = await req.json()

        this.getCardapios()

    },
      
    reloadPage(){
      window.location.reload()
    }

  },

    mounted () {
      this.getCardapios();
    },

    components: {
    Message
    }
}
</script>

<style scoped>


.container {
  background-image: linear-gradient(to top, transparent 5%, #fff 50%),
    url('/img/principal/pattern.png');
  max-width: 1110px;
  margin: 0 auto;
  height: 100vh;
  display: flex;
  flex-wrap: wrap;
}

.wave {
  width: 100%;
  margin: 0 auto;
  flex-wrap: wrap;
  background-image: url('/img/principal/wave.svg');
  background-repeat: no-repeat;
  background-size: contain;
  height: 100vh;
  opacity: 1;
}
/* Logo Princial
 */
.logo {
  margin: 0 auto;
  margin-bottom: 20px;
  max-width: 35%;
  padding-top: 60px;
}
.pastelGrande {
  max-width: 250px;
  margin-left: -15px;
  position: absolute;
  border: none;
}
.pastelGrande:hover {
  -webkit-transform: scale(1.1);
  -ms-transform: scale(1.1);
  transform: scale(1.1);
}

.pastelDir {
  margin-top: 8%;
  max-width: 15%;
  float: right;
}

.pastelEsq {
  margin-top: 8%;
  max-width: 15%;
  float: left;
}

/* Suco */
.sucoDir {
  margin-top: 5%;
  max-width: 15%;
  float: left;
}
.sucoEsq {
  margin-top: 5%;
  max-width: 15%;
  float: right;
}
.topPedido {
  display: flex;
  flex-wrap: wrap;
  background-color: #ffca00;
  padding: 10px;
  border-radius: 15px 15px 0 0;
  margin: auto;
  width: 60%;
  background: #ffca00 0% 0% no-repeat padding-box;
}
.title {
  text-align: center;
  font: italic normal bold 14px Roboto;
  letter-spacing: 0px;
  color: #a03400;
  opacity: 1;
}

.tPedido {
  width: 42%;
  margin-right: 10px;
  margin-top: 2px;
  border: 1px solid #e43636;
  font: normal normal normal 10px Roboto;
  border-radius: 5px;
  height: 25px;
  background: #ffffff 0% 0% no-repeat padding-box;
  padding: 0 0 0 15px;
}

.sabor {
  width: 41%;
  margin-right: 10px;
  margin-top: 2px;
  border: 1px solid #e43636;
  font: normal normal normal 10px Roboto;
  border-radius: 5px;
  height: 25px;
  padding: 0 0 0 15px;
  background: #ffffff 0% 0% no-repeat padding-box;
}

.preco {
  width: 13%;
  margin-top: 2px;
  border: 1px solid #e43636;
  font: normal normal normal 10px Roboto;
  border-radius: 5px;
  height: 25px;
  background: #ffffff 0% 0% no-repeat padding-box;
  padding: 0 0 0 15px;
}

.mensagemCardapio{
    margin-top: 5px;
    text-align: center;
    font: italic normal bold 14px Roboto;
    letter-spacing: 0px;
    color: #a03400;
    opacity: 1;
}

.comida {
  margin-left: 31%;
  font: normal normal normal 12px Roboto;
  color: #a03400;
  opacity: 1;
}
.bebida {
  font: normal normal normal 12px Roboto;
  color: #a03400;
  opacity: 1;
}

.Bpedido {
  padding: 10px;
  border-radius: 0 0 15px 15px;
  margin: 0 auto;
  width: 60%;
  background-color: white;
  box-shadow: 0px 0px 30px #740b0b45;
}

.descricao {
  width: 99%;
  height: 50px;
  margin-top: 10px;
  border: 1px solid #e43636;
  font: normal normal normal 10px Roboto;
  border-radius: 5px;
  padding: 0 0 0 5px;

}

.upload {
  width: 99%;
  max-height: 100px;
  margin-top: 15px;
  border: 1px solid #e43636;
  font: normal normal normal 10px Roboto;
  border-radius: 5px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 15px;
  transition: all 0.3s ease;
}

.inputFile {
  visibility: hidden;
}

.imgUpload{
  width: 40px;
  height: 40px;
  opacity: 1;
}

.active-dropzone{

  label{ 
    background-color: #fff;
    color: #e43636;
  }

}


.uploadR{
    text-align: center;
    font: italic normal bold 12px Roboto;
    background-color: #ffca00;
    opacity: 1;
    padding: 0 5px 0 5px;
    border-radius: 5px;
    border: #E43636 solid 2px;
}

.limpar {
  margin-top: 10px;
  margin-left: 25%;
  width: 20%;
  background: #e43636 0% 0% no-repeat padding-box;
  height: 30px;
  border-radius: 30px;
  opacity: 1;
  border: none;
  text-align: center;
  font: normal normal 900 12px Roboto;
  letter-spacing: 0px;
  color: #ffffff;
  opacity: 1;
  cursor: pointer;
}
.limpar:hover {
  background: #a03400;
  color: #ffffff;
}

.cadastrar {
  margin-top: 10px;
  margin-left: 5%;
  width: 20%;
  background: #ffca00 0% 0% no-repeat padding-box;
  height: 30px;
  border-radius: 30px;
  opacity: 1;
  border: none;
  text-align: center;
  font: normal normal 900 12px Roboto;
  letter-spacing: 0px;
  color: #a03400;
  opacity: 1;
  cursor: pointer;
}

.cadastrar:hover {
  background: #a03400;
  color: #ffffff;
}

input:focus {
  outline: none !important;
  border-color: #e43636;
  box-shadow: 0 0 10px #e43636;
}

/*-------------------------------------- botão toggle -------------------------------
 */
.toggle > label {
  position: relative;
  display: block;
  height: 10px;
  width: 30px;
  background: #ffffff 0% 0% no-repeat padding-box;
  border-radius: 11px;
  opacity: 1;
  border-radius: 100px;
  cursor: pointer;
  transition: all 0.3s ease;
  margin: 0 7px 0 7px;
}
.toggle > label:after {
  position: absolute;
  left: -2px;
  top: -3px;
  display: block;
  width: 15px;
  height: 15px;
  border-radius: 100px;
  background: #e33535;
  box-shadow: 0px 3px 3px rgba(0, 0, 0, 0.05);
  content: "";
  transition: all 0.3s ease;
}
.toggle > input:checked ~ label {
  background: #e33535;
}
.toggle > input:checked ~ label:after {
  left: 20px;
  background: #ffffff;
}
.toggle > input {
  display: none;
}

/* Modal */

.cardapio1 {
/*   display: none;
 */}

.content {
  /* display: none; */
  flex-wrap: wrap;
  padding: 10px;
  margin: auto;
  max-width: 60%;
  height: 40px;
  background: #e43636 0% 0% no-repeat padding-box;
  border-radius: 20px 20px 0px 0px;
}

.produtolModal {
  height: 150px;
  width: 150px;
  float: left;
  margin-left: 13%;
}

.cardapio2 {
/*   display: none;
 */}

hr {
  border-color: #e43636;
  margin: 90px 0 15px 0;
}

.tituloModal {
  float: left;
  font: italic normal bold 20px Roboto;
  letter-spacing: 0px;
  color: #ffca00;
  opacity: 1;
}

.precoModal {
  float: right;
  font: italic normal bold 16px Roboto;
  letter-spacing: 0px;
  color: #ffffff;
  opacity: 1;
}

.descricsabor {
  margin: 10px 0 10px 10px;
}

.tSabor {
  text-align: left;
  font: italic normal bold 15px Roboto;
  letter-spacing: 0px;
  color: #a03400;
  opacity: 1;
}

.produto {
  text-align: left;
  font: normal normal normal 12px Roboto;
  letter-spacing: 0px;
  color: #a03400;
  opacity: 1;
}

.tDescri {
  text-align: left;
  font: italic normal bold 15px Roboto;
  letter-spacing: 0px;
  color: #a03400;
  opacity: 1;
}

.descriProduto {
  text-align: left;
  font: normal normal normal 12px Roboto;
  letter-spacing: 0px;
  color: #a03400;
  opacity: 1;
}

</style>

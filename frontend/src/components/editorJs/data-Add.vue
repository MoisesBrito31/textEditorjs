<template>
    <div>
        <div class="card-header mb-3">
            <h4> Editor de Texto JS - novo arquivo</h4>
        </div>
        <div class="m-5 p-5 bg-active">
          <div id="editorjs" ></div>
        </div>
      <b-form>
        <b-form-group class="mt-3 pt-3" >
            <b-button @click="salvarEditor" v-bind:class="{disabled:!podeCadastrar}"  block variant="primary">
                <span v-show="espera">
                <b-spinner type="grow" variant="light"></b-spinner>
                <b-spinner type="grow" variant="light"></b-spinner>
                <b-spinner type="grow" variant="light"></b-spinner>
               </span>
               <span v-show="!espera">Salvar</span>
            </b-button>

            <b-button @click="carregaEditor" v-bind:class="{disabled:!podeCadastrar}"  block variant="primary">
                <span v-show="espera">
                <b-spinner type="grow" variant="light"></b-spinner>
                <b-spinner type="grow" variant="light"></b-spinner>
                <b-spinner type="grow" variant="light"></b-spinner>
               </span>
               <span v-show="!espera">Carregar</span>
            </b-button>
        </b-form-group>
        </b-form>
        <div>
          <!--Alerta de falha-->
          <b-alert
            :show="alertaFalha.time"
            dismissible
            variant="danger"
            @dismissed="alertaFalha.time=0"
            @dismiss-count-down="alertaFalhaTick"
          >
            <p>{{ alertaFalha.msg }}</p>
            <b-progress
              variant="danger"
              :max="alertaFalha.startTime"
              :value="alertaFalha.time"
              height="4px"
            ></b-progress>
          </b-alert>
        </div>
    </div>
</template>

<script lang="js">
import EditorJS from '@editorjs/editorjs';
import Header from '@editorjs/header'; 
import List from '@editorjs/list'; 
import SimpleImage from "@editorjs/simple-image"
import Checklist from '@editorjs/checklist'
import LinkTool from '@editorjs/link';
import { mapGetters } from 'vuex'
export default {
  name: 'editorJS-data-Add',
  created(){
    const editor = new EditorJS({ 
      holder: 'editorjs', 
      tools: { 
        header: Header, 
        list: List,
        image: SimpleImage,
        checklist: {
          class: Checklist,
          inlineToolbar: true,
        },
        linkTool: {
          class: LinkTool,
          config: {
            endpoint: 'http://localhost:8080/fetchUrl', // Your backend endpoint for url data fetching,
          }
        }
      }, 
      data:{"time":1730595179986,"blocks":[{"id":"WVi9BhcP4b","type":"header","data":{"text":"titulo","level":2}},{"id":"8164esLp9v","type":"paragraph","data":{"text":"texto1"}},{"id":"W0RrZ9I-fr","type":"paragraph","data":{"text":"texto2"}}],"version":"2.30.6"},
      onReady:()=>{
        alert("carregado!")
      },
      onChange:(api,event) =>{
        console.log('mudanca no editor: ',event)
      }
    })
    this.editor = editor;
  },
  data(){
    return{
        alertaFalha:{
          'startTime':10,
          'time':0,
          'msg':''
        },
        modalVisible:false,
        erroMsg : '',
        espera:false,
        falha: false,
        obj:{'name':'','description':''},
        editor: {},
        memoria:[],
    }
  },computed:{
    ...mapGetters(['getDominio','getToken']),    
    podeCadastrar(){
        return true;
    }
  },methods:{
    alertaFalhaTick(valor){
      this.alertaFalha.time = valor
    },
    mostarMsgOK(){
      this.$bvToast.toast('Criado!!!', {
            title: "Bem Sucedido",
            variant: "success",
            solid: true
          })
      this.obj = {'name':'','color':''}
    },
    dataFormat(){
            //const form = document.getElementById('img')
            //const file = form.files[0]
            const formdata = new FormData()
            //if(file!==undefined){formdata.append('img',file,file.name)}
            //if(this.obj.id!==undefined){formdata.append('id',this.obj.id)}
            formdata.append('name',this.obj.name)
            formdata.append('color',this.obj.color)
            formdata.append('colorHTML',`color:${this.obj.color};`)
            return formdata
    },
    salvarEditor(){
      this.editor.save().then((outputData)=>{
        console.log('dados do artigo: ',outputData)
        this.memoria = outputData;
      }).catch((error)=>{
        console.log('erro ao tentar salvar',error)
      })
    },
    carregaEditor(){
      this.editor.render(this.memoria);
    },
    async postDadosOS(){
        this.espera=true
        try{
        this.esperando = true
          const response = await fetch(`${this.getDominio}/api/os/estado/`,{
            method:"POST",
            headers:{
            'Authorization': `Token ${this.getToken}`
          },body:this.dataFormat()
        })
        if(response.ok){
          this.mostarMsgOK()
          this.espera = false
        }else{
          switch(response.status){
            case 401:
              throw new Error('Não Autorizado');
            case 404:
              throw new Error('Não Encontrado');
            default:
              throw new Error('Falha interna no Servidor');
          }
        }
      }catch(error){
        this.alertaFalha.msg= error
        this.alertaFalha.time= this.alertaFalha.startTime
        this.espera = false
      }
       
    }

  }

}
</script>
<template>
  <div>
    <Layout />
    <div class="page-container">
      <PageHeader
          title="Cadastro de Produtos"
          helpTitle="Cadastro de Produtos"
          :helpText="helpText"
      />
      <div class="my-container">
        <b-row>
          <div class="col-md-12">
            <b-card>
              <b-form-group label="Empresa">
                <b-form-select
                    class="mb-2"
                    ref="empresa"
                    id="empresa"
                    v-model="idEmpresa"
                    autocomplete="off"
                >
                  <option :value="null"></option>
                  <option
                      v-for="empresa in empresaResponse"
                      :key="empresa.idEmpresa"
                      :value="empresa.idEmpresa"
                  >
                    {{ empresa.nomeFantasia }}
                  </option>
                </b-form-select>
              </b-form-group>
            </b-card>
          </div>
        </b-row>
        <b-row>
          <div class="col-md-6">
            <b-card >
              <b-table
                  class="mb-0 my-table"
                  sticky-header
                  :items="items"
                  :fields="fields"
                  responsive="sm"
                  @row-selected="setRowSelected"
                  ref="mytable"
                  bordered
                  selectable
                  select-mode="single"
                  selected-variant="primary"
                  striped
                  hover
                  style="cursor: pointer"
              ></b-table>
              <div class="mt-2">Total de registros: {{ rows }}</div>
              <hr/>

              <div class="d-flex align-items-center justify-content-center">
                <ul class="pagination mb-0">
                  <b-pagination
                      v-model="currentPage"
                      :total-rows="rows"
                      :per-page="perPage"
                      :disabled="!Boolean(rows)"
                  >
                    <template>
                      <div slot="first-text">
                        <i class="fas fa-angle-double-left"></i>
                      </div>
                      <div slot="prev-text">
                        <i class="fas fa-angle-left"></i>
                      </div>
                      <div slot="next-text">
                        <i class="fas fa-angle-right"></i>
                      </div>
                      <div slot="last-text">
                        <i class="fas fa-angle-double-right"></i>
                      </div>
                    </template>
                  </b-pagination>
                </ul>
              </div>

            </b-card>
          </div>
          <div class="col-md-6">
            <b-card >
              <b-form-group label="Produto">
                <b-form-input

                    type="text"
                    ref="produto"
                    v-model = "produto.produto"
                    autocomplete="off"
                    class="mb-2"
                />
              </b-form-group>

              <b-form-group label="Descrição">
                <b-form-textarea
                    v-model ="produto.descricao"
                    autocomplete="false"
                    class="mb-2"
                    size="2"
                    no-resize
                />
              </b-form-group>

              <b-form-group label="Valor">
                <b-form-input
                    v-model="produto.valorBase"
                    autocomplete="false"
                    class="mb-2"
                    @input="formataValor"
                />
              </b-form-group>

              <b-row class="mt-2 ml-1 mr-1" >

                  <b-form-group
                      label="Imagem de Produto"
                      label-for="dropzone1"
                  >
                    <div  v-if="isImagemProdutoCarregado">
                      <fieldset class="fieldset-border" >
                        <div

                            @click="viewDropZone"
                            style="cursor: pointer"
                            class="espacoImagem"
                        >
                          <img
                              :src="`data:image/jpeg;base64,${produto.imagemProduto}`"
                          />
                        </div>
                      </fieldset>
                    </div>
                    <div v-else>
                      <b-form>
                        <b-form-group
                        >
                          <vue-dropzone
                              class="col-12"
                              id="dropzone1"
                              ref="myVueDropzone1"
                              :use-custom-slot="true"
                              :options="dropzoneOptions"
                              @vdropzone-complete="afterUploadComplete"
                          >
                            <div class="dropzone-custom-content">
                              <i class="display-4 text-muted bx bxs-cloud-upload"></i>
                              <h6>Arraste o arquivo jpg ou clique aqui para selecionar</h6>
                            </div>
                          </vue-dropzone>
                        </b-form-group>
                      </b-form>
                    </div>
                  </b-form-group>

              </b-row>
            </b-card>
          </div>
        </b-row>
      </div>
      <StateButtonBar
          excluir
          excluirDisabled
          novo
          :novoFunction="novo"
          cancelar
          :cancelarFunction="cancelar"
          salvar
          :salvarFunction="salvar"
          :oldObj="oldObj"
          :newObj="newObj"
          :camposObrigatorios="[
          'produto',
          'descricao',
          'valorBase',
          'imagemProduto'

        ]"
      />
    </div>
  </div>
</template>

<script>
import PageHeader from "@/components/page-header";
import StateButtonBar from "@/components/state-button-bar";
import Layout from "@/layout/main";
import { mapActions } from "vuex";
import vue2Dropzone from "vue2-dropzone";

export default {
  components: {
    PageHeader,
    StateButtonBar,
    Layout,
    vueDropzone: vue2Dropzone,
  },

  async mounted() {
    await this.listEmpresas();
  },

  data() {
    return {

      idEmpresa:null,

      fields: [
        { key: "produto", label: "Produto" },
      ],
      items:[],
      isBusy: false,
      currentPage: 1,
      perPage: 5,
      sortBy: "nome",
      sortDesc: false,

      rowSelected:{},
      produtos:[],

      produto:{
        produto:null,
        descricao:null,
        valorBase:null,
        imagemProduto:null,
        valor:null,
        idEmpresa:null,
      },

      produtoReset:{
        produto:null,
        descricao:null,
        valorBase:null,
        imagemProduto:null,
        valor:null,
        idEmpresa:null,
      },

      produtoResponse:{contetn:[]},
      empresaResponse:{contetn:[]},

      dropzoneOptions: {
        url: "https://httpbin.org/post",
        thumbnailWidth: 250,
        maxFilesize: 0.5,
        acceptedFiles: ".jpg",
        parallelUploads: 1,
        autoProcessQueue: true,
        addRemoveLinks: false,
        init: function () {
          // this.on("success", function (file, responseText) {
          //   file.previewTemplate.setAttribute("id", responseText[0].id);
          // });
          this.on("thumbnail", function (file) {
            if (file.width > 1000 || file.height > 1000) {
              file.rejectDimensions();
            } else {
              file.acceptDimensions();
            }
          });
        },
        accept: function (file, done) {
          file.acceptDimensions = done;
          file.rejectDimensions = function () {
            done("Resolução máxima permitida: 1000px x 1000px");
          };
        },
      },

      isSelected:false,
    };
  },

  watch: {
    currentPage: function (novo) {
      this.listProduto( this.idEmpresa,novo - 1);
    },

    idEmpresa:function(novo){
      this.cleanForm();
      if (novo) {
        this.cleanForm();
        this.listProduto(novo, 1);
      }
      else{
        this.cancelar();
        this.produtoResponse = {content:[],totalElements:0};
        this.items=[];
      }
    }
  },

  computed: {
    // Texto do help
    helpText() {
      return `
      <p>
          Lorem ipsum dolor sit amet consectetur adipisicing elit. Quibusdam
          praesentium repudiandae asperiores voluptatem consequatur harum,
          voluptate nulla.
      </p>
      <p>
          Quis consectetur accusantium quia consequuntur hic
          soluta. Minus nisi illum ab architecto exercitationem?
      </p>
      `;
    },

    isImagemProdutoCarregado(){
      if (this.produto.imagemProduto ) return true;
      return false;
    },

    newObj() {
      return this.produto;
    },

    oldObj() {
      return this.rowSelected;
    },

    rows() {
      return this.produtoResponse.totalElements;
    },
  },

  methods: {
    // Métodos do Vuex
    ...mapActions({
      changeLoading: "changeLoading",
    }),

    async listEmpresas() {
      this.changeLoading(true);
      await this.$axios
          .get("dominios/empresa")
          .then((response) => {
            this.empresaResponse = Object.assign([], response.data);
          })
          .catch(() => {
            this.$toasted.error("Falha ao listar empresas!");
          });
      this.changeLoading(false);
    },


    async salvar() {
      if (this.idEmpresa) {
        this.produto = Object.assign({}, this.prepareProduto(this.produto));
        this.changeLoading(true);
        await this.$axios
            .post(`empresa/${this.idEmpresa}/produto`, this.produto)
            .then(() => {
              this.$toasted.success("produto Salvo com sucesso!");
              this.listProduto(this.idEmpresa,1)
              this.cancelar();
            })
            .catch(() => {
              this.$toasted.error("Falha ao salvar Produto!");
            });
        this.changeLoading(false);
      }else{
        this.$toasted.info("Nenhuma empresa selecionada!");
      }
    },


    async listProduto(idEmpresa,page) {
      this.changeLoading(true);
      await this.$axios
          .get(`empresa/${idEmpresa}/produto`, {page})
          .then((response) => {
            this.produtoResponse = Object.assign({}, response.data);
            this.items = Object.assign([], response.data.content);
          })
          .catch(() => {
            this.$toasted.error("Falha ao listar produtos!");
          });
      this.changeLoading(false);
    },


    setRowSelected(record) {

      if (record.length > 0) {
        this.editMode = false;
        this.rowSelected = Object.assign({},this.convertValor(record[0]))
        this.produto = Object.assign({},this.rowSelected);
        this.isSelected=true;
      }
      else {
        this.isSelected=false;
      }
    },


    cleanForm() {
       this.rowSelected = Object.assign({},this.produtoReset);
       this.produto = Object.assign({},this.produtoReset);
       this.isSelected=false;
    },

    novo() {
      this.cleanForm();
      setTimeout(() => {
        this.$refs.empresa.focus();
      });
    },

    cancelar() {
      if (this.isSelected){
        console.log(this.rowSelected);
        this.produto = Object.assign({},this.rowSelected);
      }
      else {
        this.rowSelected = {};
        setTimeout(() => {
          this.cleanForm();
        });
      }
    },

    viewDropZone(){
      this.produto.imagemProduto=null;
    },

    // Métodos do dropzone
    afterUploadComplete(response) {
      if (response.status == "success") {
        let base64Bruto = response.dataURL;
        let base64Liq = base64Bruto.split(",");
        this.produto.imagemProduto = base64Liq[1];
        console.log(this.produto);
      } else {
        this.mask.bandeiraDescription = "Imagem inválida!";
      }
    },

    convertValor(produto) {
      let produtoConvert = Object.assign({},produto);
      produtoConvert.valorBase =  produto.valorBase
          .toFixed(2)
          .toString()
          .trim()
          .replace(".", ",");
      return produtoConvert;
    },

    prepareProduto(produto){
      let produtoConvert = Object.assign({},produto);
      produtoConvert.idEmpresa = this.idEmpresa;
      let valor = produto.valorBase;
      valor = valor.replace(".", "").replace(",", ".").trim();
      produtoConvert.valorBase = Number(valor);
      return produtoConvert;
    },

    formataValor() {
      let digitado = this.produto.valorBase;
      let convertido = parseInt(digitado.replace(/[\D]+/g, ""));
      let tmp = convertido + "";
      tmp = tmp.replace(/([0-9]{2})$/g, ",$1");
      if (tmp.length > 6) {
        tmp = tmp.replace(/([0-9]{3}),([0-9]{2}$)/g, ".$1,$2");
      }
      this.produto.valorBase = tmp.replace(/[a-zA-Z]+/g, "");
    },

  },
};
</script>

<style scoped>

.my-container {
  margin: 10px;
  padding-bottom: 50px;
}

.img-produto img {
  height: 100%;
  width: auto;
}


fieldset.fieldset-border {
  border: 1px groove #ddd !important;
  padding: 0 1.4em 1.4em 1.4em !important;
  margin: 0 0 1.5em 0 !important;
  -webkit-box-shadow: 0px 0px 0px 0px #000;
  box-shadow: 0px 0px 0px 0px #000;
}

legend.fieldset-border {
  font-size: 1em !important;
  font-weight: bold !important;
  text-align: left !important;
  width: inherit; /* Or auto */
  padding: 0 10px; /* To give a bit of padding on the left and right */
  border-bottom: none;
}

img {
  margin-left:auto;
  margin-right:auto ;
  display: block;
  height: 70px;
  width: 90px;
}

.espacoImagem{
  display: table-cell;
  text-align: center;
  vertical-align: middle;
  width: 230px;
  height: 240px;
}

#dropzone1{
  width: 250px;
  height: 260px;
}


</style>

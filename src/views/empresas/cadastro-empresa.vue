<template>
  <div>
    <Layout />
    <div class="page-container">
      <PageHeader
          title="Cadastro de Empresa"
          helpTitle="Cadastro de Empresa"
          :helpText="helpText"
      />
      <div class="my-container">
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
              <b-form-group label="CNPJ">
                <b-form-input
                  ref="cnpj"
                  type="text"
                  v-mask="'##.###.###/####-##'"
                  lazy-formatter
                  v-model = "empresa.cnpj"
                  autocomplete="off"
                  class="mb-2"
                />
              </b-form-group>

              <b-form-group label="Nome Fantasia">
                <b-form-input
                  v-model ="empresa.nomeFantasia"
                  autocomplete="false"
                  class="mb-2"
                />
              </b-form-group>

              <b-form-group label="Razão Social">
                <b-form-input
                  v-model="empresa.razaoSocial"
                  autocomplete="false"
                  class="mb-2"
                />
              </b-form-group>

              <b-row>
                <b-col>
                  <label for="telefone">Telefone</label>
                  <b-form-input
                    v-model="empresa.telefone"
                    autocomplete="false"
                    lazy-formatter
                    class="mb-2"
                    v-mask="'(##) ####-####'"

                  />
                </b-col>

                <b-col>
                  <label for="responsavel">Responsável Legal</label>
                  <b-form-input
                    v-model="empresa.responsavelLegal"
                    autocomplete="false"
                    class="mb-2"
                  />
                </b-col>
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
          'cnpj',
          'nomeFantasia',
          'razaoSocial',
          'telefone',
          'responsavelLegal'
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
import StringMask from 'string-mask'


export default {
  components: {
    PageHeader,
    StateButtonBar,
    Layout,
  },

  async mounted() {
    await this.listEmpresas(0);
  },

  data() {
    return {
      telefone:null,

      fields: [
        { key: "nomeFantasia", label: "Nome Fantasia" },
      ],
      items:[],
      isBusy: false,
      currentPage: 1,
      perPage: 5,
      sortBy: "nomeFantasia",
      sortDesc: false,

      rowSelected:{},
      empresas:[],
      empresa:{
        cnpj:null,
        idEmpresa:null,
        nomeFantasia:null,
        razaoSocial:null,
        responsavelLegal:null,
        telefone:null,
      },
      empresaReset:{
        cnpj:null,
        idEmpresa:null,
        nomeFantasia:null,
        razaoSocial:null,
        responsavelLegal:null,
        telefone:null,
      },

      empresaResponse:{contetn:[]},


    };
  },

  watch: {
    currentPage: function (novo) {
      this.listEmpresas( novo - 1);
    },
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



    newObj() {
      return this.empresa;
    },

    oldObj() {
      return this.rowSelected;
    },

    rows() {
      return this.empresaResponse.totalElements;
    },
  },

  methods: {
    // Métodos do Vuex
    ...mapActions({
      changeLoading: "changeLoading",
    }),



    removerMascaras(e){
      e.cnpj = e.cnpj.replace(/[^a-zA-Z0-9]/g, "");
      e.telefone = e.telefone.replace(/[^a-zA-Z0-9]/g, "");
      return e;
    },


    async salvar() {
      this.empresa = Object.assign({}, this.removerMascaras(this.empresa));
      if (!this.empresa.idEmpresa){
          this.changeLoading(true);
          await this.$axios
              .post('empresa', this.empresa)
              .then(() => {
                this.$toasted.success("Empresa salva com sucesso!");
                this.cleanForm()
                this.currentPage=1;
              })
              .catch(() => {
                this.$toasted.error("Falha ao salvar Empresa!");
              });
          this.changeLoading(false);
      }
      else{
        this.changeLoading(true);
        await this.$axios
            .put(`empresa/${this.empresa.idEmpresa}`, this.empresa)
            .then(() => {
              this.$toasted.success("Empresa atualizada com sucesso!");
              this.cleanForm();
              this.currentPage=1;

            })
            .catch(() => {
              this.$toasted.error("Falha ao atualizar Empresa!");
            });
        this.changeLoading(false);
      }
    },


    async listEmpresas(page) {
        this.isBusy = true;
        console.log('page',page)
        await this.$axios
            .get(`empresa/?page=${ page }` )
            .then((response) => {
              console.log(response.data);
              this.empresaResponse = Object.assign({}, response.data);
              this.items = Object.assign([], response.data.content);
            })
            .catch(() => {
              this.$toasted.error("Falha ao listar empresas!");
            });
        this.isBusy = false;
    },


    setRowSelected(record) {
      if (record.length > 0) {
        this.editMode = true;
        this.rowSelected = Object.assign({}, this.aplicarMascaras(record[0]));
        this.empresa = Object.assign({},this.aplicarMascaras(record[0]));
        console.log(this.empresa);
      }
    },

    aplicarMascaras(e){
      if (e.cnpj) e.cnpj = this.cnpjMask(e.cnpj);
      if (e.telefone) e.telefone = this.telefoneMask(e.telefone);


      return e;
    },



    cleanForm() {
      this.editMode = false;
      this.empresa = Object.assign({},this.empresaReset);
      this.rowSelected={};
    },

    novo() {
      this.cleanForm();
      setTimeout(() => {
        this.$refs.cnpj.focus();
      });
    },

    cancelar() {
      if (this.empresa.idEmpresa){
        this.empresa=Object.assign({},this.rowSelected);
      }
      else{
        this.rowSelected={};
        this.cleanForm();
      }

    },

    cnpjMask(cnpjNovo){
      const cnpj = new StringMask('00.000.000/0000-00', { reverse: true });
      if (cnpj.validate(cnpjNovo)) {
        return cnpj.apply(cnpjNovo)
      } else {
        return cnpjNovo
      }
    },

    celularMask(novoCelular){
      const celular = new StringMask('(00) 00000-0000', { reverse: true });
      if (celular.validate(novoCelular)) {
        return formatterCel.apply(novoCelular)
      } else {
        return novoCelular;
      }
    },

    telefoneMask(novoTelefone){
      const telefone = new StringMask('(00) 0000-0000', { reverse: true });
      if (telefone.validate(novoTelefone)){
        return telefone.apply(novoTelefone)
      }
      else{
        return novoTelefone;
      }
    },


  },
};
</script>

<style scoped>
  .my-container {
    margin: 10px;
    padding-bottom: 50px;
  }
  .img-produto {
    height: 60px;
    display: flex;
    align-items: flex-end;
    justify-content: center;
  }
  .img-produto img {
    height: 100%;
    width: auto;
  }
</style>

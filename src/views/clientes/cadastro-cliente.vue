<template>
  <div>
    <Layout />
    <div class="page-container">
      <PageHeader
          title="Cadastro de Clientes"
          helpTitle="Cadastro de Clientes"
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
              <b-form-group label="CPF">
                <b-form-input
                    ref="cpf"
                    type="text"
                    v-mask="'###.###.###-##'"
                    lazy-formatter
                    v-model = "cliente.cpf"
                    autocomplete="off"
                    class="mb-2"
                />
              </b-form-group>

              <b-form-group label="Nome">
                <b-form-input
                    v-model ="cliente.nome"
                    autocomplete="false"
                    class="mb-2"
                />
              </b-form-group>

              <b-form-group label="Telefone">
                <b-form-input
                    v-model="cliente.telefone"
                    autocomplete="false"
                    lazy-formatter
                    class="mb-2"
                    v-mask="'(##) ####-####'"

                />
              </b-form-group>
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
          'cpf',
          'nome',
          'telefone',

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
    await this.listEmpresas();
  },

  data() {
    return {
      formDisabled:false,
      idEmpresa:null,

      fields: [
        { key: "nome", label: "Nome" },
      ],
      items:[],
      isBusy: false,
      currentPage: 1,
      perPage: 5,
      sortBy: "nome",
      sortDesc: false,

      rowSelected:{},
      clientes:[],

      cliente:{
        cpf:null,
        nome:null,
        telefone:null,
        idEmpresa:null,
      },

      clienteReset:{
        cpf:null,
        nome:null,
        telefone:null,
        idEmpresa:null,
      },

      clienteResponse:{content:[]},
      empresaResponse:{content:[]},


    };
  },

  watch: {
    currentPage: function (novo) {
      this.listCliente( this.idEmpresa,novo - 1);
    },

    idEmpresa:function(novo){
      this.cleanForm();
      if (novo) {
        this.listCliente(novo, 0);
      }
      else{
        this.cancelar();
        this.clienteResponse = {content:[],totalElements:0};
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



    newObj() {
      return this.cliente;
    },

    oldObj() {
      return this.rowSelected;
    },

    rows() {
      return this.clienteResponse.totalElements;
    },
  },

  methods: {
    // Métodos do Vuex
    ...mapActions({
      changeLoading: "changeLoading",
    }),

    async listEmpresas() {

      await this.$axios
          .get("dominios/empresa")
          .then((response) => {
            this.empresaResponse = Object.assign([], response.data);
          })
          .catch(() => {
            this.$toasted.error("Falha ao listar empresas!");
          });

    },

    removerMascaras(e){
      if (e.cpf) e.cpf = e.cpf.replace(/[^a-zA-Z0-9]/g, "");
      if (e.telefone)  e.telefone = e.telefone.replace(/[^a-zA-Z0-9]/g, "");
      return e;
    },

    async salvar() {
      if (this.idEmpresa) {
        this.cliente = Object.assign({}, this.removerMascaras(this.cliente));
        this.cliente.idEmpresa = this.idEmpresa;
        if (!this.cliente.idCliente) {
          this.changeLoading(true);
          await this.$axios
              .post(`empresa/${this.idEmpresa}/cliente`, this.cliente)
              .then(() => {
                this.$toasted.success("Cliente Salvo com sucesso!");
                this.cleanForm();
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
              .put(`empresa/${this.idEmpresa}/cliente/${this.cliente.idCliente}`, this.cliente)
              .then(() => {
                this.$toasted.success("Cliente Salvo com sucesso!");
                this.cleanForm();
                this.currentPage=1;
              })
              .catch(() => {
                this.$toasted.error("Falha ao salvar Empresa!");
              });
          this.changeLoading(false);
        }

      }
      else {
        this.$toasted.info("Nenhuma empresa selecionada!");
      }
    },


    async listCliente(idEmpresa,page) {
      if (idEmpresa) {
        this.changeLoading(true);
        await this.$axios
            .get(`empresa/${idEmpresa}/cliente/?page=${ page }`)
            .then((response) => {
              this.clienteResponse = Object.assign({}, response.data);
              this.items = Object.assign([], response.data.content);
            })
            .catch(() => {
              this.$toasted.error("Falha ao listar clientes!");
            });
        this.changeLoading(false);
      }
      else{
        this.$toasted.info("É necessário selecionar uma empresa!");
      }
    },


    setRowSelected(record) {
      if (record.length > 0) {
        this.formDisabled = true;
        this.editMode = false;
        this.cliente.telefone = null;
        this.rowSelected.telefone = null;
        this.cliente = Object.assign({},this.aplicarMascaras(record[0]))
        this.rowSelected = Object.assign({},this.aplicarMascaras(record[0]))

      }
    },

    aplicarMascaras(e){
      e.cpf = this.cpfMask(e.cpf);
      e.telefone = this.telefoneMask(e.telefone)
      return e;
    },

    cleanForm() {
      this.editMode = false;
      this.cliente = this.removerMascaras(this.cliente);
      this.cliente = Object.assign({},this.clienteReset);
      this.rowSelected = {};
    },

    novo() {
      this.cleanForm();
      setTimeout(() => {
        this.$refs.empresa.focus();
      });
    },

    cancelar() {
      if (this.cliente.idCliente){
        this.cliente = Object.assign({},this.rowSelected);
      }
      else{
        this.rowSelected={};
        setTimeout(() => {
          this.cleanForm();
        });
      }

    },

    cpfMask(cpfNovo){
      const cpf = new StringMask('000.000.000-00', { reverse: true });
      if (cpf.validate(cpfNovo)) {
        return cpf.apply(cpfNovo)
      } else {
        return cpfNovo
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

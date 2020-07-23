<template>
  <div class='home' >
    <v-snackbar
      top
      v-model="snackbar"
      :color="message.type"
      dark
    >
      {{ message.message }}
      <v-btn
        text
        @click="snackbar = false"
      >
        Fechar
      </v-btn>
    </v-snackbar>
    <v-content class='search pa-2'>
      <div>
        <v-row justify='center' align='center'>
          <v-icon size='48' class='mr-4 primary--text'>fa-building</v-icon>
            <h1 class='primary--text tracking-in-expand-fwd'>
              Localizador de Empresas
            </h1>
        </v-row>
      </div>
      <div class="mt-2">
        <v-form v-model='formIsValid' ref="cnpj">
          <v-row>
            <v-spacer></v-spacer>
            <v-col
            offset-xl='1'
            xl='3'
            lg='4'
            sm='7'
            xs='12'
            cols='12'
            >
                <v-text-field
                class='cnpj-textfield'
                v-model='cnpj'
                outlined
                placeholder='CNPJ...'
                color='primary'
                :rules='[rules.cnpj]'
                v-mask='cnpjMask'
                >
                </v-text-field>
            </v-col>
            <v-col
            xl='2'
            lg='2'
            sm='3'
            xs='12'
            cols='12'
            >
                <v-btn
                class='mx-auto'
                rounded
                color='primary'
                x-large
                :block='$vuetify.breakpoint.xsOnly'
                :disabled='!formIsValid'
                @click='consultarEmpresa()'>
                  <span class='buttom-text white--text'>Localizar</span>
                </v-btn>
            </v-col>
            <v-spacer></v-spacer>
          </v-row>
        </v-form>
      </div>
    </v-content>
    <v-content class='results'>
      <v-row justify='center' align='center'>
        <div class="figure" v-if='empresas.length < 1'>
          <v-row justify='center' align='center'>
            <div class="figure">
              <v-img
              src='../assets/localizar.png'
              width='250'
              height='250'
              contain
              >
              </v-img>
            </div>
          </v-row>
          <v-row justify='center' align='center'>
            <p class='white--text'>
              Localize acima a primeira empresa
            </p>
          </v-row>
        </div>
        <v-slide-group :class="setSlideMargin()" multiple show-arrows>
          <v-slide-item
          :class="setSlideMargin()"
          v-for="empresa in empresas"
          :key="empresa.cnpj"
          >
            <Card
            :empresa="empresa"
            @card-clicked="openMaps(empresa)"
            />
          </v-slide-item>
        </v-slide-group>
      </v-row>
    </v-content>
    <v-dialog
    v-if="mapsDialog"
    v-model="mapsDialog"
    fullscreen
    >
      <Map
      :APIkey="MAPS_API"
      :empresa="empresa"
      @close-maps="closeMaps()"
      />
    </v-dialog>
  </div>
</template>

<script>
import axios from 'axios'
import jsonpAdapter from 'axios-jsonp'
import { mask } from 'vue-the-mask'
import Card from '../components/Card'
import Map from '../components/Map'

export default {
  name: 'home',
  directives: {
    mask
  },
  components: {
    Card,
    Map
  },
  data: () => {
    return {
      MAPS_API: process.env.VUE_APP_API_MAPS,
      formIsValid: true,
      cnpj: '',
      empresas: [],
      cnpjMask: '##.###.###/####-##',
      mapsDialog: false,
      snackbar: false,
      message: {
        type: '',
        message: ''
      },
      rules: {
        cnpj: value => (/[0-9]{2}\.?[0-9]{3}\.?[0-9]{3}\/?[0-9]{4}-?[0-9]{2}/g).test(value) || 'CNPJ Inválido'
      },
      empresa: {}
    }
  },
  methods: {
    async consultarEmpresa () {
      const cnpj = this.cnpj.replace(/[^0-9]/g, '')
      try {
        const { data } = await axios.get(`https://receitaws.com.br/v1/cnpj/${cnpj}`, {
          crossDomain: true,
          adapter: jsonpAdapter
        })
        if (data.status === 'ERROR') {
          this.message = {
            type: 'error',
            message: data.message
          }
          return
        }
        this.empresas.unshift(data)
        localStorage.setItem('empresas', JSON.stringify(this.empresas))
        this.$refs.cnpj.reset()
        this.message = {
          type: 'success',
          message: 'Empresa localizada com sucesso!'
        }
      } catch (error) {
        this.message = {
          type: 'error',
          message: 'Erro interno do servidor'
        }
      } finally {
        this.snackbar = true
      }
    },
    openMaps (empresa) {
      this.empresa = empresa
      this.mapsDialog = true
    },
    closeMaps () {
      this.mapsDialog = false
    },
    setSlideMargin () {
      return 'pa-2'
    }
  },
  created () {
    this.empresas = localStorage.getItem('empresas') ? JSON.parse(localStorage.getItem('empresas')) : []
  }
}
</script>

<style lang='less'>
.results {
  min-height: 65vh;
  background: linear-gradient(90deg, rgba(69,154,158,1) 10%, rgba(56,134,105,1) 65%)
}
.search {
  height: 35vh;
}

.v-content {
  align-items: center;
}

@media screen and (min-width: 1600px) {
  .results {
    height: 70vh;
  }
  .search {
    height: 30vh;
  }
}

body::-webkit-scrollbar {
    display: none;
}
</style>

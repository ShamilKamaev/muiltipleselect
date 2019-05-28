  <template>
    <div class="multi-select__container"
     :class="{'is-active': contFocus}"
      v-on-clickaway="handleFocusOut"
      @click="handleFocus">
      <input type="text" 
        :placeholder="textSearchStatus" 
        v-model="searchString"
        @focus="textFieldFocus = true"
        @blur="textFieldFocus = false"
        ref="tetxtInput">
      <ul class="services-list">
        <li class="not-found" v-if="searchStatus && searchString.length > 2 && filtredServices.length == 0">Совпадений не найдено</li>
        <li v-for="service in filtredServices" :key="service.id" @click="addService(service)">{{ service.label }}</li>
      </ul>
    </div>
  </template>

  <script>
  import axios from 'axios'
  import { mixin as clickaway } from 'vue-clickaway';
  
  export default {
    name: 'select-single',
    mixins: [ clickaway ],

    data() {
      return {
        contFocus: false,
        textFieldFocus: false,
        searchString: '',
        searchStatus: false,
        loadedServices: [],
        selectedServices: [],
      }
    },

    props: {
      clinicId: {
        required: true
      },

      branchesIds: {
        type: Array,
        required: false,
      },

      apiUrl: {
        type: String,
        required: true
      }
    },

    methods: {
      handleFocusOut() {
        this.contFocus = false;
       },
      handleFocus() {
        this.contFocus = true;
       },
      addService(service) {
        this.selectedServices.splice(0, 1);
        this.selectedServices.push(service);
        this.searchString = '';
      },
      removeSelected(index) {
        this.selectedServices.splice(index, 1);
      }
    },

    computed: {
      textSearchStatus() {
        if(this.selectedServices.length != 0) {
          return this.textFieldFocus ? 'Начните вводить название услуги' : this.selectedServices[0].label
        } else {
          return this.textFieldFocus ? 'Начните вводить название услуги' : 'Выберите услуги'
        }
      },

      selectedIds() {
        let idsArr = [];
        this.selectedServices.forEach(item => {
          idsArr.push(item.id.toString());
        })
        return idsArr;
      },

      filtredServices() {
        if (this.loadedServices.length != 0) {
          return (
            this.loadedServices.filter(item => {
              return item.label.toLowerCase().indexOf(this.searchString.toLowerCase()) !== -1 && this.selectedIds.indexOf(item.id.toString()) == -1;
            })
          )
        } else return [];
      }
    },

    watch: {
      searchString: function (val) {
        if ((val.length < 3)) {
          this.loadedServices = [];
        }
        if (val.length == 3) {
          let buildedArr = [];
          axios({
            method: 'get',
            url: thsi.apiUrl,
            params: {
              clinicId: this.clicnicId,
              term: val
            },
          }).then(response => {
            this.searchStatus = true;
            response.data.results.forEach(elem => {
              buildedArr.push({
                id: elem.id,
                label: elem.label
              })
              if ('children' in elem) {
                elem.children.forEach(childrenElem => {
                  buildedArr.push({
                    id: childrenElem.id,
                    label: childrenElem.label
                  });
                })
              }
            })
          }).then(() => {
            this.loadedServices = buildedArr;
          })
        }
      }
    }
  }
  </script>
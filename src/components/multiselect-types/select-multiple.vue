  <template>
    <div class="multi-select__container"
     :class="{'is-active': contFocus}"
      v-on-clickaway="handleFocusOut"
      @click="handleFocus">
       <ul class="slected-services" v-show="selectedServices.length !== 0">
        <transition-group name="fade-zoom">
          <li v-for="(service, index) in selectedServices" :key="service.id">{{ service.label }} <i @click="removeSelected(index)"></i></li>
        </transition-group>
      </ul>
      <input type="text" 
        :placeholder="textSearchStatus" 
        v-model="searchString"
        @focus="textFieldFocus = true"
        @blur="textFieldFocus = false"
        ref="tetxtInput">
      <ul class="services-list">
        <li class="not-found" v-if="searchStatus && searchString.length > 2 && filtredServices.length == 0">Совпадений не найдено</li>
        <li v-for="service in filtredServices" :key="service.id" @click="addService(service)"><text-highlight :queries="searchString">{{ service.label }} </text-highlight></li>
      </ul>
    </div>
  </template>

  <script>
  import axios from 'axios'
  import { mixin as clickaway } from 'vue-clickaway';
  import TextHighlight from 'vue-text-highlight';

  export default {
    name: 'select-multiple',
    mixins: [ clickaway ],

    components: {
      'text-highlight': TextHighlight
    },

    data() {
      return {
        contFocus: false,
        textFieldFocus: false,
        searchStatus: false,
        searchString: '',
        loadedServices: [],
        selectedServices: [],
      }
    },

    props: {
      alreadySelected: {
        type: Array,
        default() {
          return [];
        }
      },

      branchesIds: {
        type: Array,
        required: false,
      }, 

      clinicId: {
        required: false,
        default() {
          return 15212;
        }
      },

      apiUrl: {
        type: String,
        required: false,
        default() {
          return 'http://spb.p.test.napopravku.ru/profile/load-smd-tree/';
        }
      }
    },

    computed: {
      textSearchStatus() {
        return this.textFieldFocus ? 'Начните вводить название услуги' : 'Выбирите услуги'
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
            url: this.apiUrl,
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
    },
    

    created() {
      this.alreadySelected.forEach(elem => {
        if(elem.checked == true) {
            this.selectedServices.push({
            id: elem.id,
            label: elem.label
          })
        }
        if ('children' in elem) {
          elem.children.forEach(childrenElem => {
            if(childrenElem.checked == true) {
                this.selectedServices.push({
                id: childrenElem.id,
                label: childrenElem.label
              });
            }
          })
        }
      })

      
    },

    methods: {
      handleFocusOut() {
        this.contFocus = false;
       },
      handleFocus() {
        this.contFocus = true;
       },
      addService(service) {
        this.$refs.tetxtInput.focus();
        this.selectedServices.push(service);
      },
      removeSelected(index) {
        this.$refs.tetxtInput.focus();
        this.selectedServices.splice(index, 1);
      }
    }
  }
  </script>
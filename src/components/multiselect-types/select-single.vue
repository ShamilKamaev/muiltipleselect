  <template>
    <div class="multi-select__container"
     :class="{'is-active': contFocus}"
      v-on-clickaway="handleFocusOut"
      @click="handleFocus">
      <input 
        :class="{'has-selected-value': !textFieldFocus && selectedOptions.length !== 0}"
        type="text" 
        :readonly="!textBox"
        :placeholder="textSearchStatus" 
        v-model="searchString"
        @focus="textFieldFocus = true"
        @blur="textFieldFocus = false">
        <span v-show="searchString.length >= 3 && searchStatus == false" class="progress"></span>
      <ul class="services-list">
        <li class="not-found" v-if="searchStatus && searchString.length > 2 && filtredOptions.length == 0">Совпадений не найдено</li>
        <li v-for="service in filtredOptions" :key="service.id" @click="addService(service)"><text-highlight :queries="searchString">{{ service.label }}</text-highlight></li>
      </ul>
    </div>
  </template>

  <script>
  import axios from 'axios'
  import { mixin as clickaway } from 'vue-clickaway';
  import TextHighlight from 'vue-text-highlight';
  
  export default {
    name: 'select-single',
    mixins: [ clickaway ],

    components: {
      'text-highlight': TextHighlight,
    },

    data() {
      return {
        contFocus: false,
        textFieldFocus: false,
        searchString: '',
        searchStatus: false,
        loadedOptions: [],
        selectedOptions: [],
        changeIteration: 0,
        testVal: '',
      }
    },

    created() {
      if(this.alreadySelected.length !== 0) {
        this.alreadySelected.forEach(item => this.selectedOptions.push(item));
      }

      if(this.async == false) {
        this.$emit('querySettingsEmit', this.querySettings);
      }
    },

    mounted() {
      this.$nextTick().then(() => {
        this.changeIteration++;
      });

      if(!this.textBox) {
        this.$emit('querySettingsEmit', this.querySettings);
      }
    },


    props: {
      symbolsLimit: {
        type: Number,
        required: false,
        default: 3
      },
      placeholder: {
        type: String,
        required: true
      },
      secondPlaceholder: {
        type: String,
        required: true
      },
      async: {
        type: Boolean,
        required: true
      },
      loadingOptions: {
        type: Function,
        required: false
      },
      textBox: {
        type: Boolean,
        required: false,
        default: true
      },

      alreadySelected: {
        type: Array,
        default() {
          return [];
        }
      },
    },

    methods: {
      handleFocusOut() {
        this.contFocus = false;
        this.searchString = '';
        if(this.async) {
          this.loadedOptions = [];
        }
       },
      handleFocus() {
        this.contFocus = true;
       },
      addService(service) {
        this.selectedOptions.splice(0, 1);
        this.selectedOptions.push(service);
        this.searchString = '';
        setTimeout(() => {
          this.contFocus = false;
        }, 0);
      },
      removeSelected(index) {
        this.selectedOptions.splice(index, 1);
      }
    },

    computed: {
      querySettings() {
        return ({
          async: this.async,
          searchString: this.searchString,
          callback: options => {
            this.loadedOptions = options;
            this.searchStatus = false;
          }
        })
      },

      textSearchStatus() {
        if(this.selectedOptions.length != 0) {
          return this.textFieldFocus ? this.secondPlaceholder : this.selectedOptions[0].label
        } else {
          return this.textFieldFocus ? this.secondPlaceholder : this.placeholder
        }
      },

      selectedIds() {
        let idsArr = [];
        this.selectedOptions.forEach(item => {
          idsArr.push(item.id);
        })
        return idsArr;
      },


      buildedOptions() {
        let buildedArr = [];
        if(this.loadedOptions.lenght != 0) {
          this.loadedOptions.forEach(elem => {
            buildedArr.push({
              id: elem.id,
              label: elem.label
            })
          })
        }
        return buildedArr;
      },
      
      filtredOptions() {
        let filtredArr = [];
        if (this.buildedOptions.length != 0) {
          filtredArr = this.buildedOptions.filter(item => {
            return item.label.toLowerCase().indexOf(this.searchString.toLowerCase()) !== -1 && !this.selectedIds.some(sItem => sItem === item.id)});
          return filtredArr;
        } else return [];
      }
    },

    watch: {
      searchString (val) {
        if (val.length < this.symbolsLimit) {
          if(this.async) {
            this.loadedOptions = [];
          }
          this.searchStatus = false;
        }
        if (val.length == this.symbolsLimit) {
          this.$emit('querySettingsEmit', this.querySettings);
        }
      },

      selectedOptions(val) {
        if(this.changeIteration > 0) {
          this.$emit('input', val);
        }
      }
    }
  }
  </script>
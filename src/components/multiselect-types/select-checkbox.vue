  <template>
    <div class="multi-select__container" 
        :class="{'is-active': contFocus}"
        v-on-clickaway="handleFocusOut"
        @click="handleFocus"
        ref="mscont"
        >
      <ul class="slected-services" v-show="selectedParents.length !== 0">
        <transition-group name="fade-zoom">
          <li v-for="(service, index) in selectedParents" :key="service.id">{{ service.label }} {{ service.checkedStatus }}<i @click="removeService(index)"></i></li>
        </transition-group>
      </ul>
      <input type="text" 
        v-model="searchString"
        :placeholder="textSearchStatus"
        @focus="textFieldFocus = true"
        @blur="textFieldFocus = false">
      <ul class="services-list services-list_checboxes">
        <li class="not-found" v-if="searchStatus && searchString.length > 2 && filtredServices.length == 0">Совпадений не найдено</li>
        <li 
          v-for="service in filtredServices" 
          :key="service.id"
          :class="{'is-checked': service.checked}">
          <div @click.stop="addParent(service)"><text-highlight :queries="searchString">{{ service.label }}</text-highlight></div>
          <ul>
            <li 
              v-for="childService in service.children" 
              :key="childService.id"
              :class="{'is-checked': childService.checked}">
              <div @click.stop="addParent(childService)"><text-highlight :queries="searchString">{{ childService.label }}</text-highlight></div>
              </li>
          </ul>
        </li>
      </ul>
    </div>
  </template>

  <script>
  import axios from 'axios'
  import { mixin as clickaway } from 'vue-clickaway';
  import TextHighlight from 'vue-text-highlight';


  export default {
    name: 'select-checkbox',

    mixins: [ clickaway ],
    components: {
      'text-highlight': TextHighlight
    },

    data() {
      return {
        qstatus: false,
        contFocus: false,
        textFieldFocus: false,
        searchStatus: false,
        searchString: '',
        loadedServices: [],
        selectedServices: [],
        selectedParents: [],
        queryCache: [],
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
    

    created() {
        if (this.alreadySelected.length !== 0) {
          this.alreadySelected.forEach(elem => {
            elem.id = elem.id.toString();
            elem.parent = true;
            elem.checkedSum = 0;
            if ('children' in elem) {
              elem.children.forEach(childElem => {
                childElem.id = childElem.id.toString();
                childElem.parent = false;
              });
            }
            this.queryCache.push(elem);
          })
        }
      
    },

    mounted() {
      this.queryCache.forEach(item => {
        this.selectedParents.push(item);
      })
    },


    computed: {
      textSearchStatus() {
        return this.textFieldFocus ? 'Начните вводить название услуги' : 'Выбирите услуги'
      },

      selectedIds() {
        let idsArr = [];
        this.selectedParents.forEach(item => {
          if(item.checked == true) {
            idsArr.push(item.id.toString());
          } 
          if('children' in item) {
            item.children.forEach(childItem => {
              if(childItem.checked == true) {
                idsArr.push(childItem.id.toString());
              }
            })
          }
        })
        return idsArr;
      },

      selectedParentsIds() {
        let idsArr = [];
        this.selectedParents.forEach(item => {
          idsArr.push(item.id.toString());
        })
        return idsArr;
      },


      filtredServices() {
        var mapedArr = [];
        var filtredArr = [];
        
          mapedArr = this.queryCache.map((item, index) => {
            if(this.qstatus) {
              if ('children' in item) {
                item.children.forEach(childElem => {
                  childElem.parentId = item.id.toString();
                });
              }
              
              if (this.selectedIds.some(elem => elem.toString() == item.id.toString())) {
                  item.checked = true;
              } else {
                  item.checked = false;
              }

              if ('children' in item) {
                item.children.forEach(childItem => {
                  if (this.selectedIds.some(elem => elem.toString() == childItem.id.toString())) {
                    childItem.checked = true;
                  } else {
                    childItem.checked = false;
                  }
                });
              }
            }
            

            if('children' in item) {
              item.checkedSelf = item.checked ? 1 : 0;
              item.checkedAll = item.children.length + 1;
              item.checkedSum = item.children.filter(elem => elem.checked == true).length + item.checkedSelf;
              item.checkedStatus = '(' + item.checkedSum + ' из ' + item.checkedAll + ')';
              this.$forceUpdate();
            } else {
              item.checkedStatus = '(1 из 1)';
            }
            return item;
          })
        

        if (mapedArr.length !== 0 && this.searchString.length >= 3) {
          filtredArr = mapedArr.filter(item => {
            return (
              'children' in item ?
              (item.children.some(childItem => childItem.label.toLowerCase().indexOf(this.searchString.toLowerCase()) !== -1) ||
                item.label.toLowerCase().indexOf(this.searchString.toLowerCase()) !== -1) :
              item.label.toLowerCase().indexOf(this.searchString.toLowerCase()) !== -1)
          })
        }
        return filtredArr;
      }
    },

    
    methods: {
      handleFocus() {
        this.contFocus = true;
      },

      handleFocusOut() {
        this.contFocus = false;
      },

      addParent(service) {
        // добавлеие через ребенка
        if(service.parent == false && service.checked == false) {
          if(!this.selectedParentsIds.some(item => item.toString() == service.parentId.toString())) {
            this.filtredServices.forEach(filtredItem => {
              if(filtredItem.id.toString() == service.parentId.toString()) {
                service.checked = true;
                this.selectedParents.push(filtredItem);
              }
            })
          } else { service.checked = true; }
        } 
        // удаление через ребенка
        else if (service.parent == false && service.checked == true) {
          service.checked = false;
          this.selectedParents.forEach((selecteParentItem, index) => {
            if(selecteParentItem.id.toString() == service.parentId.toString() && selecteParentItem.checked == false && selecteParentItem.children.every(item => item.checked == false)) {
              this.selectedParents.splice(index, 1);
            }
          })
        }
        // добавление одиночного елемента
        else if (service.parent == true && service.checked == false && !('children' in service)) {
          service.checked = true;
          this.selectedParents.push(service);
        }

        // удаление одиночного елемента
        else if (service.parent == true && service.checked == true && !('children' in service)) {
          this.selectedParents.forEach((selecteParentItem, index) => {
            if(selecteParentItem.id.toString() == service.id.toString()) {
              this.selectedParents.splice(index, 1);
            }
          })
          service.checked = false;
          
        }

        // добавление через родителя
        else if (service.parent == true && service.checked == false) {
          if('children' in service && !this.selectedParentsIds.some(item => item.toString() == service.id.toString())) {
            service.children.forEach(item => {
              item.checked = true;
            });
            service.checked = true;
            this.selectedParents.push(service);
          } else {
            service.checked = true
          }
        } 
        // удаление через родителя
        else if (service.parent == true && 'children' in service && service.checked == true) {
          service.checked = false;
          this.selectedParents.forEach((selecteParentItem, index) => {
            if(selecteParentItem.id.toString() == service.id.toString() && selecteParentItem.checked == false && selecteParentItem.children.every(item => item.checked == false)) {
              this.selectedParents.splice(index, 1);
            }
          })
        }
      },

      removeService(index) {
        this.selectedParents.splice(index, 1);
      }
    },



    watch: {
      searchString: function (val) {
        if ((val.length < 3)) {
          this.searchStatus = false;
          this.loadedServices = [];
        }
        if (val.length == 3) {
          this.qstatus = true;
          axios({
            method: 'get',
            url: this.apiUrl,
            params: {
              clinicId: this.clicnicId,
              term: val
            },
          }).then(response => {
            
            response.data.results.forEach(elem => {
              elem.id = elem.id.toString();
              elem.checked = false;
              elem.parent = true;
              elem.checkedSum = 0;
              if('children' in elem) {
                elem.children.forEach(childElem => {
                  childElem.id = childElem.id.toString();
                  childElem.parent = false;
                  childElem.checked = false;
                });
              }
              if(this.queryCache.length !== 0 && !this.queryCache.some(item => item.id.toString() == elem.id.toString())) {
                this.queryCache.push(elem);
              } else if(this.queryCache.length == 0) {
                this.queryCache.push(elem);
              }
            })
            this.searchStatus = true;
          })
        }
      }
    }
  }
  </script>
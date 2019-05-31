  <template>
    <div class="multi-select__container" 
        :class="{'is-active': contFocus}"
        v-on-clickaway="handleFocusOut"
        @click="handleFocus"
        >
      <transition-group name="fade-zoom" tag="ul" class="slected-services" v-show="selectedParents.length !== 0">
        <li v-for="(service, index) in selectedParents" :key="service.id">{{ service.label }} {{ service.checkedStatus }}<i @click="removeOption(service, index)"></i></li>
      </transition-group>
      <div class="text-input">
        <input 
          type="text" 
          v-model="searchString"
          :placeholder="textSearchStatus"
          @focus="textFieldFocus = true"
          @blur="textFieldFocus = false">
          <span v-show="searchString.length >= 3 && searchStatus == false" class="progress"></span>
      </div>
      <ul class="services-list services-list_checboxes">
        <li class="not-found" v-if="searchStatus && searchString.length > 2 && filtredOptions.length == 0">Совпадений не найдено</li>
        <li 
          v-for="service in filtredOptions" 
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
  // import vueCustomScrollbar from 'vue-custom-scrollbar'


  export default {
    name: 'select-checkbox',

    mixins: [ clickaway ],
    components: {
      'text-highlight': TextHighlight,
      // vueCustomScrollbar
    },

    data() {
      return {
        changeIteration: 0,
        queryIteration: 0,
        qstatus: false,
        contFocus: false,
        textFieldFocus: false,
        searchStatus: false,
        searchString: '',
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
    },
    

    created() {
      if (this.alreadySelected.length !== 0) {
        this.alreadySelected.forEach(elem => {
          elem.id = elem.id;
          elem.parent = true;
          elem.checkedSum = 0;
          if ('children' in elem) {
            elem.children.forEach(childElem => {
              childElem.id = childElem.id;
              childElem.parent = false;
            });
          }
          this.queryCache.push(elem);
        })
      }
    },

    mounted() {
      this.$nextTick().then(() => {
        this.changeIteration++;
      })

      if(this.selectedParents.length !== 0) {
        this.changeIteration++;
        this.$forceUpdate();
      }

      this.queryCache.forEach(item => {
        this.selectedParents.push(item);
      })
    },


    computed: {
      querySettings() {
        return ({
          async: this.async,
          searchString: this.searchString,
          callback: options => {
            this.queryIteration++;
            options.forEach(elem => {
              elem.id = elem.id;
              elem.checked = false;
              elem.parent = true;
              elem.checkedSum = 0;
              if('children' in elem) {
                elem.children.forEach(childElem => {
                  childElem.id = childElem.id;
                  childElem.parent = false;
                  childElem.checked = false;
                });
              }
              if(this.queryCache.length !== 0 && !this.queryCache.some(item => item.id == elem.id)) {
                this.queryCache.push(elem);
              } else if(this.queryCache.length == 0) {
                this.queryCache.push(elem);
              }

              // обновляем данные в уже выбранных
              if('children' in elem && this.queryIteration == 1) {
                this.alreadySelected.forEach(queryCacheItem => {
                  if(queryCacheItem.id == elem.id) {
                    elem.children.forEach(childElem => {
                      if(!this.selectedIds.some(idItem => idItem == childElem.id)){
                        childElem.checked = false;
                        queryCacheItem.children.push(childElem);
                      }
                    })
                  }
                })
              }
            })

            this.searchStatus = true;
          }
        })
      },


      textSearchStatus() {
        return this.textFieldFocus ? this.secondPlaceholder : this.placeholder
      },

      selectedIds() {
        let idsArr = [];
        this.selectedParents.forEach(item => {
          if(item.checked == true) {
            idsArr.push(item.id);
          } 
          if('children' in item) {
            item.children.forEach(childItem => {
              if(childItem.checked == true) {
                idsArr.push(childItem.id);
              }
            })
          }
        })
        return idsArr;
      },

      selectedParentsIds() {
        let idsArr = [];
        this.selectedParents.forEach(item => {
          idsArr.push(item.id);
        })
        return idsArr;
      },


      filtredOptions() {
        var mapedArr = [];
        var filtredArr = [];
        
          mapedArr = this.queryCache.map((item, index) => {
            if(this.qstatus) {
              if ('children' in item) {
                for(let i = item.children.length - 1; i >= 0; i--) {
                  item.children[i].parentId = item.id;
                  if(item.children[i].id == item.children[i].parentId) {
                    item.children.splice(i, 1);
                  }
                }

                item.children.forEach((childElem, index) => {
                  childElem.parentId = item.id;
                });
              }
              
              if (this.selectedIds.some(elem => elem == item.id)) {
                  item.checked = true;
              } else {
                  item.checked = false;
              }

              if ('children' in item) {
                item.children.forEach((childItem, index) => {
                  // передвигаем совпавшие по названию элемнты на верх
                  // if(childItem.label.toLowerCase().indexOf(this.searchString.toLowerCase()) !== -1) {
                  //   if(item.children[index + 1] != undefined) {
                  //     item.children.splice(index, 1);
                  //     item.children.unshift(childItem);
                  //   }
                  // }
                  ////////////////////////////////////////////////////
                  if (this.selectedIds.some(elem => elem == childItem.id)) {
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
        this.searchString = '';
        this.contFocus = false;
      },

      addParent(service) {
        if(this.searchStatus) {
          // добавлеие через ребенка
        if(service.parent == false && service.checked == false) {
          if(!this.selectedParentsIds.some(item => item == service.parentId)) {
            this.filtredOptions.forEach(filtredItem => {
              if(filtredItem.id == service.parentId) {
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
            if(selecteParentItem.id == service.parentId && selecteParentItem.checked == false && selecteParentItem.children.every(item => item.checked == false)) {
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
            if(selecteParentItem.id == service.id) {
              this.selectedParents.splice(index, 1);
            }
          })
          service.checked = false;
        }

        // добавление через родителя
        else if (service.parent == true && service.checked == false) {
          if('children' in service && !this.selectedParentsIds.some(item => item == service.id)) {
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
            if(selecteParentItem.id == service.id && selecteParentItem.checked == false && selecteParentItem.children.every(item => item.checked == false)) {
              this.selectedParents.splice(index, 1);
            }
          })
        }
        }
        
      },

      removeOption(service, index) {
        this.selectedParents.splice(index, 1);
        if(this.alreadySelected.length !== 0) {
          for(var i = this.alreadySelected.length - 1; i >= 0; i--) {
            if(parseInt(this.alreadySelected[i].id) == parseInt(service.id)) {
              this.alreadySelected.splice(i, 1);
            }
          }

          if(!this.searchStatus) {
            for(var i = this.queryCache.length - 1; i >= 0; i--) {
              if(parseInt(this.queryCache[i].id) == parseInt(service.id)) {
                this.queryCache.splice(i, 1);
              }
            }
          }
        }
      }
    },


    watch: {
      selectedParents: {
        handler: function(newVal) {
          if(this.changeIteration > 0) {
            this.$emit('input', newVal)
          }
        },
        deep: true
      },

      searchString: function (val) {
        if ((val.length < this.symbolsLimit)) {
          this.searchStatus = false;
        }
        if (val.length == this.symbolsLimit) {
          this.qstatus = true;
          this.$emit('querySettingsEmit', this.querySettings);
        }
      }
    }
  }
  </script>
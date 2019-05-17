<template>
  <div id="multiselect">
    <component :is="selectType" :select-type="selectType" :clicnic-id="clinicId"></component>
  </div>
</template>

<script>
  
  import selectMultiple from './components/multiselect-types/select-multiple'
  import selectCheckbox from './components/multiselect-types/select-checkbox'
  import selectSingle from './components/multiselect-types/select-single'

  export default {
    name: 'MultiSelect',

    components: {
      'select-multiple': selectMultiple,
      'select-checkbox': selectCheckbox,
      'select-single': selectSingle,
    },

    props: {
      selectType: {
        type: String,
        required: true
      },

      branchesIds: {
        type: Array,
        required: false,
      }, 

      clicnicId: {
        required: true
      }
    }
  }
</script>

<style lang="scss">
  .visually-hidden {
    position: absolute !important;
    clip: rect(1px 1px 1px 1px);
    /* IE6, IE7 */
    clip: rect(1px, 1px, 1px, 1px);
    padding: 0 !important;
    border: 0 !important;
    height: 1px !important;
    width: 1px !important;
    overflow: hidden;
  }

  .multi-select__container {
    border-radius: 4px;
    border: solid 1px #bcbcbc;
    box-sizing: border-box;
    overflow: hidden;
    font-family: Gotham Pro, sans-serif;
    font-weight: normal;
    outline: none;

    &.is-active {
      border: 1px solid #40b3b7;
      .services-list {
        height: auto;
        max-height: 340px;
        overflow-y: scroll;
      }
    }

    * {
      box-sizing: border-box;
    }

    input[type="text"] {
      font-size: 16px;
      min-height: 48px;
      width: 100%;
      display: block;
      border: none;
      padding-left: 15px;
      padding-right: 15px;
      outline: none;
      border-bottom: 1px solid rgba(34, 36, 38, 0.08);
      font-family: Gotham Pro, sans-serif;
      font-weight: normal;
    }
  }


  .services-list {
    padding-left: 10px;
    list-style-type: none;
    text-align: left;
    margin: 0;
    height: 0;
    overflow: hidden;
    li {
      padding-top: 15px;
      padding-bottom: 15px;
      padding-left: 5px;
      border-bottom: 1px solid rgba(34, 36, 38, 0.08);
      &.not-found {
        padding-top: 15px !important;
        padding-bottom: 15px !important;
      }
      &:hover {
        @media (max-width: 960px) {
          background-color: #eceeef;
        }
      }
    }

    &_checboxes {
      padding-left: 0;
      li {
        position: relative;
        &:hover {
          @media (max-width: 960px) {
            background-color: transparent;
          }
        }


        span {
          position: relative;
          cursor: pointer;
          &::after {
            content: '';
            position: absolute;
            left: -35px;
            top: 12px;
            width: 20px;
            height: 20px;
            border-radius: 4px;
            border: 1px solid #bcbcbc;
            transition: background-color ease 150ms;
          }
        }
        

        &.is-checked {
          &>span {
            &::after {
              border-color: #40b3b7;
              background-color: #40b3b7;
              background-image: url('./assets/images/svg/checked-white.svg');
              background-repeat: no-repeat;
              background-position: center;
            }
          }
          
        }
      }
      &>li {
        padding-left: 50px;
        padding-top: 0;
        padding-bottom: 0;
        &>span {
          display: block;
          padding-top: 15px;
          padding-bottom: 15px;
          border-bottom: 1px solid rgba(34, 36, 38, 0.08);
        }

        &>ul {
          list-style-type: none;
          padding-left: 0;
          &>li {
            border-bottom: none;
            padding-left: 50px;
            padding-top: 0;
            padding-bottom: 0;
            &:last-of-type {
              span {
                border-bottom: none;
              }
            }
            span {
              display: block;
              padding-top: 15px;
              padding-bottom: 15px;
              border-bottom: 1px solid rgba(34, 36, 38, 0.08);
            }
          }
          
        }
      }
    }
  }

  .slected-services {
    padding: 0;
    margin: 0;
    list-style-type: none;
    display: block;
    padding: 10px;

    li {
      position: relative;
      display: inline-block;
      background-color: #40b3b7;
      padding: 8px 30px 8px 8px;
      border: 1px solid #36979a;
      border-radius: 4px;
      margin: 2px;
      color: #fff;
      font-weight: 600;
      font-size: 14px;
      i {
        position: absolute;
        width: 24px;
        height: 24px;
        background-color: transparent;
        border-radius: 50%;
        top: 2px;
        right: 2px;
        background-image: url('./assets/images/svg/white-cross.svg');
        background-position: center;
        background-repeat: no-repeat;
        &:hover {
          background-color: rgba(51, 51, 51, 0.2);
        }
      }
    }
  }



    // animations
  .fade-zoom-enter-active {
    transition: all .3s ease;
  }
  .fade-zoom-leave-active {
    transition: all .3s ease;
  }
  .fade-zoom-enter, .fade-zoom-leave-to {
    transform: scale(0.8);
    opacity: 0;
  }
  
</style>
<template>
  <div id="m-select">
    <component :is="selectType" :clicnic-id="clinicId" :branchesIds="branchesIds" :apiUrl="apiUrl" :already-selected="alreadySelected"></component>
    <!-- <component 
      :is="'select-checkbox'" 
      :clinic-id="'15212'" 
      :api-url="'http://spb.p.test.napopravku.ru/profile/load-smd-tree/'" 
      :alreadySelected=""></component> -->
  </div>
</template>

<script>
  import selectMultiple from './components/multiselect-types/select-multiple'
  import selectCheckbox from './components/multiselect-types/select-checkbox'
  import selectSingle from './components/multiselect-types/select-single'

  export default {
    name: 'm-select',

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
      },

      apiUrl: {
        type: String,
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
              background-image: url("data:image/svg+xml,%3Csvg width='14' height='10' viewBox='0 0 14 10' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath fill-rule='evenodd' clip-rule='evenodd' d='M13.139 0.245006C13.065 0.167498 12.9761 0.1058 12.8776 0.0636454C12.7791 0.0214908 12.6731 -0.000244141 12.566 -0.000244141C12.4589 -0.000244141 12.3528 0.0214908 12.2543 0.0636454C12.1559 0.1058 12.0669 0.167498 11.993 0.245006L5.21599 7.09701L2.37499 4.25001C2.30103 4.1725 2.21212 4.1108 2.11363 4.06865C2.01514 4.02649 1.90912 4.00476 1.80199 4.00476C1.69486 4.00476 1.58884 4.02649 1.49035 4.06865C1.39186 4.1108 1.30294 4.1725 1.22899 4.25001C1.15198 4.32559 1.0908 4.41577 1.04905 4.51527C1.00729 4.61477 0.985779 4.7216 0.985779 4.82951C0.985779 4.93741 1.00729 5.04424 1.04905 5.14374C1.0908 5.24324 1.15198 5.33342 1.22899 5.40901L4.64199 8.83401C4.79199 8.98501 5.01599 9.08601 5.21599 9.08601C5.43195 9.07736 5.63665 8.98733 5.78899 8.83401L13.139 1.42901C13.2946 1.27126 13.3818 1.05859 13.3818 0.837006C13.3818 0.615426 13.2946 0.402755 13.139 0.245006Z' fill='white'/%3E%3C/svg%3E");
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
        background-image: url("data:image/svg+xml,%3Csvg width='9' height='9' viewBox='0 0 9 9' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Crect y='7.7782' width='11' height='1' transform='rotate(-45 0 7.7782)' fill='white'/%3E%3Crect x='0.707153' width='11' height='1' transform='rotate(45 0.707153 0)' fill='white'/%3E%3C/svg%3E");
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

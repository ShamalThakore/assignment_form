<template>
    <div v-if="visible" id="listloader">
      <div class="listloader"></div>
    </div>
  </template>
  
  <script>
  export default {
    name: "SpinnerLoader",
    props: {
      visible: {
        type: Boolean,
        default: false
      }
    }
  };
  </script>
  
  <style scoped>
  #listloader {
    position: fixed;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    background-color: rgba(0, 0, 0, 0.5);
    z-index: 9999;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  
  .listloader {
    border: 5px solid #f3f3f3;
    border-radius: 50%;
    border-top: 5px solid #3498db;
    width: 70px;
    height: 70px;
    animation: spin 1s linear infinite;
  }
  
  @keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
  }
  </style>
  
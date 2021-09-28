<template>
  <Form @drop.prevent="drop" @change="selectedFile"/>

</template>

<script>
  import Form from "../components/Form.vue";
  import { ref } from "vue";

  export default{
    name: "Home",
    components:{
      Form
    },
    setup(){
      let dropzoneFile = ref("");

      const drop = (e) => {
        dropzoneFile.value = e.dataTransfer.files[0]
      };
      
      const selectedFile = () =>{
        dropzoneFile.value = document.querySelector('.dropzoneFile').files[0]
      };

      return { dropzoneFile, drop, selectedFile };

    }
  }  
</script>






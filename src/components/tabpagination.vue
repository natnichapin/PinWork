<script setup>
import {ref} from "vue"
let countPage = ref(1)
defineEmits(['returnPage'])
const props = {
    countPage : {
        type : [Number,undefined]
    },
    TempObjTargetLength :{
        type: Object
    }
    
}
const showPagination =(direction)=>{
 
 if(direction === 'right'){
    console.log(props.TempObjTargetLength.value.vocabs.length);
   if(countPage.value < Math.ceil(props.TempObjTargetLength.vocabs.length/4)){
     countPage.value++
   }
  
 }
 else{
   if(countPage.value >1){
     countPage.value --
   }
   
 }
 


 //เอา Temporary vocab ไปไวใน pagination แล้วส่งกลับมา
 //ทำไม่ได้เพราะ Props ถูกส่งมาครั้งเดียว
 emits('returnPage',countPage.value)
}
console.log(props.countPage);
</script>
 
<template>
      <div class="btn-group mx-auto">
  <button class="btn" @click="showPagination('left')">«</button>
  <button class="btn"> {{ countPage  }}</button>
  <button class="btn" @click="showPagination('right')">»</button>
      </div>
</template>
 
<style scoped>

</style>
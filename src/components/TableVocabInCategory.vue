<script setup>
import { ref , onMounted,onUpdated,onBeforeUnmount} from 'vue'
import MaterialSymbolsEditDocumentOutline from './icons/MaterialSymbolsEditDocumentOutline.vue'
import IconParkSolidBigX from './icons/IconParkSolidBigX.vue'
import FluentEmojiHighContrastCheckMarkButton from './icons/FluentEmojiHighContrastCheckMarkButton.vue'
const EditObject = ref({}) ;
let statusButton = ref([])
let previousNameCategory 

const emits = defineEmits(["editvocab"])
const props = defineProps({
    TemporaryVocabShow :{
        type : Object ,
        required : true ,
    } ,

    countPages : {
      type:Number,
      required:true 
    } ,
    
    ObjectCategoryClicked: {
      type: Object
    }
    
}) 


onUpdated(()=>{
  
  if(props.ObjectCategoryClicked.CategoryName!== previousNameCategory){
    EditObject.value = props.TemporaryVocabShow
    statusButton.value =[]
    previousNameCategory = props. ObjectCategoryClicked.CategoryName
    let obj={}
 props. ObjectCategoryClicked.vocabs.forEach(element => {
    
  
    obj[element.word]=false
    console.log(obj);
    statusButton.value.push(obj)
     
  });
  console.log(statusButton.value);
}
})

onBeforeUnmount(()=>{
  statusButton.value=[]
})
const EditVocabfunction = (editVocab,event) =>{
statusButton.value[event.target.id] = !statusButton.value[event.target.id]
console.log(statusButton.value[event.target.id]);


}

const confrimfunction = (event)=>{
  statusButton.value[event.target.id] = !statusButton.value[event.target.id]
  emits('editvocab',EditObject.value,props.ObjectCategoryClicked.id)

}


</script>
 
<template>
<div>
<table class="table table-zebra w-11/12  mx-auto  text-white ">
            <!-- head -->
            <thead class="">
              <tr>
                <th>No.</th>
                <th>Vocabs</th>
                <th colspan="3" >Meaning</th>
                <th  class="text-center">Edit</th>
              
              </tr>
            </thead>
            <tbody >
              <!-- row 1 -->
              <tr v-for=" (vocab,index) in  EditObject.vocabs" :key="vocab.word"  >
                <!-- หมายเลขคำศัพท์ให้เรียงต่อกัน -->
                <th> {{    (((props.countPages-1)*4)+index)+1 }}</th> 
                <td ><input  v-if="statusButton[vocab.word]" type="text" v-model="vocab.word">   <span v-else > {{ vocab.word }}   </span> </td>
              
                <td colspan="3"> <input v-if="statusButton[vocab.word]" type="text" v-model="vocab.meaning">    <span v-else > {{ vocab.meaning }}  </span> </td>
                <td class="flex space-x-5 justify-center ">
                  <div >  </div>
                     <MaterialSymbolsEditDocumentOutline :id=vocab.word v-if="!statusButton[vocab.word]" class="w-9 h-10 text-white hover:drop-shadow-lg hover:text-stone-500" @click="EditVocabfunction(vocab,$event)" />
                  <FluentEmojiHighContrastCheckMarkButton   :id=vocab.word v-if="statusButton[vocab.word]"  class="w-9 h-10 text-white hover:text-stone-500" @click="confrimfunction" />
                  <IconParkSolidBigX class="w-9 h-10 text-red-600"/>
                </td>
              
                </tr>
             

            </tbody>
          </table>  
</div>
</template>
 
<style scoped>

</style>
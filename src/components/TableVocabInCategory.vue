<script setup>
import { ref , onMounted,onUpdated,onBeforeUnmount} from 'vue'
import MaterialSymbolsEditDocumentOutline from './icons/MaterialSymbolsEditDocumentOutline.vue'
import IconParkSolidBigX from './icons/IconParkSolidBigX.vue'
import FluentEmojiHighContrastCheckMarkButton from './icons/FluentEmojiHighContrastCheckMarkButton.vue'
import { pagination } from '../composable/pagination'

//ยังไงก็ต้องเอา pagination มาวนในนี้ให้ได้ 
//แก้ ให้แก้ไขชื่อ catagory ได้
const EditObjectShowPage = ref([]) ;
let EditAllObject 
let previousNameCategory 
const emits = defineEmits(["editvocab","deletevocab"])
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
    },
    deleteStatus : {
      type : Boolean
    }
    
}) 
onMounted(()=>{
  console.log(props.countPages);
  console.log(props.ObjectCategoryClicked);
})

onUpdated(()=>{

  if(props.ObjectCategoryClicked.CategoryName!== previousNameCategory){
    console.log(props.countPages);
 
    EditObjectShowPage.value = props.TemporaryVocabShow
    console.log(typeof(props.TemporaryVocabShow));
    console.log(props.TemporaryVocabShow);
    console.log(props.ObjectCategoryClicked);
    EditAllObject = props.ObjectCategoryClicked 
   
 
    previousNameCategory = props.ObjectCategoryClicked.CategoryName

  console.log( props.ObjectCategoryClicked.vocabs);
  console.log(EditAllObject);
}

})

const EditVocabfunction = (event) =>{
  console.log(event.target.id);
  console.log(EditObjectShowPage.value.vocabs);
  let editObjVocabStatus = EditObjectShowPage.value.vocabs.find(x=>x.id == event.target.id)
  console.log(editObjVocabStatus);
  editObjVocabStatus.status = !editObjVocabStatus.status
console.log( editObjVocabStatus.status);
}
const confrimfunction = (event)=>{
console.log(event.target.id);
console.log(EditObjectShowPage.value);

  let editObjVocab = EditObjectShowPage.value.vocabs.find(x=>x.id == event.target.id)
  console.log(editObjVocab);
  // issue : Edit ครั้งที่ 2 ไม่ไป (
  console.log(EditAllObject);
  EditAllObject.vocabs = EditAllObject.vocabs.map((vocab) => {
                if (vocab.id == editObjVocab.id) {
                  vocab.word = editObjVocab.word       
                  vocab.meaning = editObjVocab.meaning        
                 return vocab
                         }
                  else{
                    return vocab
                  }
                         
                        })

              
  
  editObjVocab.status =   ! editObjVocab.status
  
  
  emits('editvocab',EditAllObject)
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
              <tr v-for=" (vocab,index) in  EditObjectShowPage.vocabs" :key="index"  >
                <!-- หมายเลขคำศัพท์ให้เรียงต่อกัน -->
                <th> {{    (((props.countPages-1)*4)+index)+1 }}  </th> 
                <td ><input  v-if="vocab.status" type="text" v-model="vocab.word">  <span v-else > {{ vocab.word }}   </span> </td>
              
                <td colspan="3"> 
                  <input v-if="vocab.status" type="text" v-model="vocab.meaning">    
                  <span v-else > {{ vocab.meaning }}  </span> 
                </td>
                <td class="flex space-x-5 justify-center ">
                  <div >  </div>
                     <MaterialSymbolsEditDocumentOutline :id=vocab.id v-if="!vocab.status" class="w-12 h-12 text-white hover:drop-shadow-lg hover:text-stone-500" @click="EditVocabfunction" />
                     <FluentEmojiHighContrastCheckMarkButton   :id=vocab.id v-if="vocab.status"  class="w-12 h-12 text-white hover:text-stone-500" @click="confrimfunction" />
                     <!-- <button class="btn btn-error" :id=vocab.id  @click="$emit('deletevocab',$event)">Delete</button> -->
                     <span v-if="props.deleteStatus" :id=vocab.id  @click="$emit('deletevocab',$event)"  class="text-3xl text-red-500 font-bold hover:text-amber-400">X</span>
                </td>
              
                </tr>
             

            </tbody>
          </table>  
</div>
</template>
 
<style scoped>
</style>
<script setup>
import { getCategories } from '../composable/getCategories';
import {ref,onMounted} from 'vue'
import {myword} from '../class/myword.js'
import Card from '../components/Card.vue'
import tabpagination from '../components/tabpagination.vue';
import {pagination} from '../composable/pagination.js'
import TableVocabInCategory from '../components/TableVocabInCategory.vue';

// แยกคอมโพเนน ของหน้าที่เปลี่ยนได้
onMounted(async ()=>{
  let categories = await getCategories()
 let Vocabs = []
  categories.forEach((x)=>{
      x.vocabs.forEach((y)=>{
        Vocabs.push(new myword(y.word,y.meaning))
      })
      CategoryAll.value.push({id:x.id,CategoryName:x.CategoryName,vocabs:Vocabs})
      Vocabs = []
  })
})
const TemporaryName = ref()
const TemporaryGroupVocabs =ref([]) 
const CategoryAll = ref([])
const TemporaryVocab=ref({word:'',meaning:''})
const EditCategory = ref(false)
const CategoryCard = ref([])
let countPage = ref(1)
let countId = 2
//เช็คว่ากดของปุ่มไหน
let checknumber=ref()
const EditCategoryfunc= (i=NaN) =>{
    checknumber.value=i
    if(isNaN(checknumber.value)){
      checknumber.value=NaN
    }
    EditCategory.value=!EditCategory.value
}
const addVocab = () => {
  //เชคว่าชื่อไม่ซ้ำ
    if(TemporaryGroupVocabs.value.every(x=>x.word !== TemporaryVocab.value.word) && TemporaryVocab.value.word.length>0 && TemporaryVocab.value.meaning.length>0 ){
    
    let TemporaryVocabtest = TemporaryVocab.value.meaning.trim()
   
       if(TemporaryVocabtest.includes(" ")){
        TemporaryVocabtest =TemporaryVocabtest.replace(/\s/gi,",")
       }
  
      TemporaryGroupVocabs.value.push(new myword(TemporaryVocab.value.word,TemporaryVocabtest ))
      
  
        TemporaryVocab.value={word:'',meaning:''}
    }
    else{
        alert('please enter Category name')
    }
}
// กด ปุ่ม close ตอน add vocab
const clear = () =>{
    TemporaryVocab.value={word:'',meaning:''}
}
const deleteVocab = (event) =>{
  
    TemporaryGroupVocabs.value = TemporaryGroupVocabs.value.filter((Vocab)=>Vocab.word!==event.target.id)
}

let TempObjTarget ={} //category ทั้งหมด
const returnPage = (page) =>{
  console.log(TempObjTarget)
  TemporaryShow.value['vocabs'] =pagination(TempObjTarget.vocabs,page)
  countPage.value = page
  }

const showCategory =(id)=>{
  
  //หาตามไอดีที่กด
  TempObjTarget  = CategoryAll.value.find(x=>x.id == id)
  console.log(TempObjTarget);
  TemporaryShow.value['categoryName'] = TempObjTarget.CategoryName
 TemporaryShow.value['vocabs'] = pagination(TempObjTarget.vocabs)
 
}
const page = ref({add:true, show:false})
const TemporaryShow = ref({})
//category 
//กดดูภายใน category
const  changPage=(event,value,i)=>{
     if(value==='show'){
      showCategory(event.target.id)
     }
   
      if (typeof i === "number") {
        page.value['show']=true
        page.value['add']=false
      }
      // if (typeof i === undefined) 
      else{
        page.value['add']=true
        page.value['show']=false
      }

      checknumber.value=i
}
const deleteCategory =async (event)=>{
  let id = event.target.id
  try {
    const res = await fetch(`http://localhost:5000/Categoties/${id}`, {
      method: "DELETE",
    });
    if (res.ok) {
      CategoryAll.value = CategoryAll.value.filter((category) => {
        return category.id != event.target.id;
      }
      );
    } else throw new error("Error, cannot delete data!");
  } catch (error) {
    console.log(error);
  }
  EditCategoryfunc()
  if (page.value['show']) {
    changPage(event)
    console.log("can change")
  }
}
const AddCategory =async (event)=>{
  countId ++ 
  let id = event.target.id
    if(TemporaryName.value && TemporaryGroupVocabs.value.length>0 && CategoryAll.value.every((x)=>x.CategoryName!==TemporaryName.value)){
      try {
    const res = await fetch(`http://localhost:5000/Categoties/${id}`, {
      method: "POST",
      headers: {
    "Content-Type": "application/json",
      },
      body: JSON.stringify({id:countId,CategoryName:TemporaryName.value,vocabs:TemporaryGroupVocabs.value})
    });
    if (res.ok) {
      CategoryAll.value.push({id:countId,CategoryName:TemporaryName.value,vocabs:TemporaryGroupVocabs.value})
      TemporaryGroupVocabs.value=[]
      TemporaryName.value =[]

    } else throw new error("Error, cannot delete data!");
  } catch (error) {
    console.log(error);
  }
    }
    else {
      alert('Can not creat Category')
    }
  
  }
  //edit

const modifyVocabInCategory =async(EditObj)=>{

EditObj.vocabs.forEach(x=> console.log(x))

try {
    const res = await fetch(
            `http://localhost:5000/Categoties/${EditObj.id}`,
            {
                method: 'PUT',
                headers: {
                    'content-type': 'application/json'                },
                body: JSON.stringify({
                  id:EditObj.id,
                  CategoryName:EditObj.CategoryName,
                vocabs: EditObj.vocabs,               
                  })
            }
        )
        if (res.status === 200) {
          TemporaryShow.value['vocabs'] = pagination(TempObjTarget.vocabs)
            // console.log('edit successfully')           
             const modifyCategory = await res.json() //หลังอัพเดทจะคืนค่า ผลลัพธ์มา
            //หา obj ที่ถูกแก้ จาก beck-end แล้วเปลี่ยนค่าของ ที่มีฝั่ง front
            CategoryAll.value = CategoryAll.value.map((Category) => {
                if (Category.id === EditObj.id) {
                  // Category.id = EditObj.id
                  // Category.CategoryName = EditObj.CategoryName
                  Category.vocabs  = EditObj.vocabs
                  
                  return Category
                         }
                
                else{
                  return Category
                }
                
                        })
          
               } 
            else {
            throw new Error('cannot edit!')
        }
    } catch (error) {
        console.log(error)
    }
}


</script>
 
<template>
<div class=" overflow-hidden w-full h-full  flex flex-col text-black bg-gradient-to-b from-amber-900  to-black  ">
    <div class="w-1/6 bg-white text-4xl">  Back </div>
    <div class=" w-full h-full  flex ">
    <div class="w-3/12 h-5/6 space-y-5  flex flex-col overflow-auto  pt-4  mx-auto  mt-10 border-4 rounded-lg border-amber-700 bg-amber-200 relative ">
        <div class="flex flex-col " v-for="(Category,index) in CategoryAll" :key="Category.id">
        <Card 
        :PageObject="page"
        :Category = "Category"
        :index = "index"
        :EditCategory="EditCategory"
        :checknumber="checknumber"
        @changPage="changPage"
        @EditCategoryfunc="EditCategoryfunc"
        @deleteCategory="deleteCategory"
        />
      
        </div>
    </div>
    <!-- หน้าสำหรับ add -->
    <div class="w-4/6 h-5/6 space-y-5  flex flex-col   pt-4  mx-auto  mt-10 border-4 rounded-lg border-amber-700 bg-amber-200 relative " >
      <div class="w-full h-full flex flex-col  " v-show="page.add" >
        <div class="flex w-full h-1/6 space-x-3 justify-center">    
            <label for="NameCategory">Name</label> 
            <input v-model="TemporaryName" type="text" placeholder="Type here" class="input  text-white input-bordered w-full max-w-xs" />
        </div>
        <div class="flex space-x-3 w-full  h-1/6 justify-center"> 
        <label for="NameCategory">Vocabs</label>
        <!-- ปุ่ม หน้า add -->
        <label for="my-modal" class="btn btn-info w-2/12  " >add vocab</label>
        <label  class="btn btn-info w-2/12  " @click="AddCategory">add Category</label>
         </div>
         <div class="flex flex-col m-auto w-11/12 h-3/6 rounded-lg overflow-auto bg-white"> 
                
            <div >

                <div class="overflow-x-auto ">
                <!-- ตาราง หน้า add vocab -->
  <table class="table w-full   text-white">
    <!-- head -->
    <thead>
      <tr>
        <th>No.</th>
        <th>Vocabs</th>
        <th colspan="3" >Meaning</th>
        <th>Edit</th>
      
      </tr>
    </thead>
    <tbody>
      <!-- row 1 -->
      <tr v-for=" (Vocab,index) in TemporaryGroupVocabs" :key="Vocab.word">
        <th>{{ index+1 }}</th>
        <td >{{ Vocab.word }}</td>
       
        <td  colspan="3">
          <div class="inline-block">
            {{ Vocab.meaning}}
          </div>
        </td>
        <td ><span :id=Vocab.word @click="deleteVocab"> X </span></td>
      </tr>
    

    </tbody>
  </table>
</div>

            </div>

 <!-- Modal แอดคำศัพท์ -->
<input type="checkbox" id="my-modal" class="modal-toggle" />
<div class="modal">
  <div class="modal-box flex flex-col space-y-2 border-4 text-black  border-amber-800 bg-amber-200">
    <h3 class="font-bold text-lg">Your vocab </h3>
    <div class="flex w-full h-1/6 space-x-3 justify-center">   
            <label for="NameCategory">Vocab</label> 
            <input type="text" v-model="TemporaryVocab.word" placeholder="Type here" class="input text-white  input-bordered w-full max-w-xs" />
        </div>
        <div class="flex w-full h-1/6 space-x-3 justify-center">   
            <label for="NameCategory">Meaning</label> 
            <input type="text"  v-model="TemporaryVocab.meaning"  placeholder="Type here" class="input text-white  input-bordered w-full max-w-xs" />
        </div>
    <div class="modal-action flex">
      <label for="my-modal" class="btn" @click="clear">Close</label>
      <label for="my-modal" class="btn" @click="addVocab">Add</label>
    </div>
  </div>
</div>
         </div>
        </div> 
        <div class="w-full  h-full flex flex-col overflow-auto space-y-5  " v-show="page.show" >
          <div class="w-full h-10 flex ">

            <button @click="changPage"
            class="w-fit  bg-blue-500 hover:bg-blue-400 text-white font-bold py-2 px-4 border-b-4 border-blue-700 mx-5 hover:border-blue-500 rounded"
            >
            BACK TO ADD PAGE
            </button>
          </div>
         
            
          <div class="bg-amber-100 w-3/5 h-14 mx-auto text-5xl font-LilitaOne  text-zinc-900 rounded-2xl text-center "> {{ TemporaryShow.categoryName }}</div>
                  <div class="w-full  h-4/6 overflow-auto ">
          <TableVocabInCategory :TemporaryVocabShow="TemporaryShow" :countPages="countPage" :ObjectCategoryClicked="TempObjTarget" @editvocab="modifyVocabInCategory" />
        </div>
        <tabpagination :countPage="countPage" :TempObjTargetLength="TempObjTarget"  @returnPage="returnPage" />
        </div>
    </div>
</div>
</div>
</template>
 
<style scoped>
</style>
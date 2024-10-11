
<template>
   <div style="display: flex;align-items: center;justify-content: center; min-height: 600px;">
      <v-card width="600" style="padding: 50px;display: flex; flex-direction: column; gap: 40px;">
       
        <v-card-title  class="text-center">
         Quản lý công việc
        </v-card-title>
        <v-card>
          <v-form @submit.prevent="addTask" ref="form">
           <v-card-text>
            <v-text-field label="Tên công việc" v-model="work.name" variant="outlined" density="compact" :rules="nameRules" required />
            <v-btn color="primary" type="submit" block="" style="margin-top: 20px;">Thêm công việc</v-btn>
           </v-card-text> 
          </v-form>        
        </v-card>
        <v-card>
           <v-card-actions>
             <v-btn color="red" @click="handleTasks" >Tất cả</v-btn>
             <v-btn color="primary" @click="handleTaskDone">Hoàn thành</v-btn>
             <v-btn color="primary" @click="handleTaskNotDone" >Đang thực hiện</v-btn>
           </v-card-actions>
        </v-card>
        <v-card v-for="task in tasks" :key="task.id">
          <div style="display: flex; justify-content: space-between; align-items: center; padding: 5px;">
            <v-checkbox-btn v-model="task.status" @change="updateStatus(task)">
              <template v-slot:label>
                <span :class="{ 'line-through': task.status }">{{ task.name }}</span>
              </template>
            </v-checkbox-btn>
          <div>
             <v-icon @click="editTask(task.id)" color="yellow">mdi-pencil</v-icon>
             <v-icon @click="deletetask(task.id)" color="red">mdi-delete</v-icon>
          </div>
          </div>
          
        </v-card>
      </v-card>
   </div>
</template>

<script setup>
import { onMounted, reactive, ref } from 'vue'
import axios from 'axios'
const nameRules = [
  (v) =>!!v || 'Tên công việc không được để trống',
  (v)=>!(tasks.includes(t=>t.name.toLowerCase()==v.toLowerCase()))||'Tên công việc không được trùng'
]
const work=reactive({
  id:'',
  name:'',
  status:false
})
const typeSubmit=ref('add')
const tasks=reactive([])
const form=ref()
const getAllTasksAPI=()=>{
  // Fetch data from API
  axios.get('http://localhost:3000/tasks')
  .then(res=>tasks.push(...res.data))
  .catch(err=>console.log(err))
}
onMounted(()=>{
   getAllTasksAPI()  // Fetch all tasks when component is mounted
})
//add new task
const addTask=async()=>{
  // Validate form before adding new task
  const {valid}=await form.value.validate()
  if(!valid){
    return;
  }
  if(typeSubmit.value=='add'){
    work.id=new Date().getTime();
  axios.post('http://localhost:3000/tasks',work)
    .then(res=>tasks.push(res.data))
    .catch(err=>console.log(err))
   work.name=''
  }else{
    console.log(work);
    
     axios.put(`http://localhost:3000/tasks/${work.id}`,work)
    .then(res=>tasks.splice(tasks.findIndex(t=>t.id===work.id),1,res.data))
    .catch(err=>console.log(err))
    typeSubmit.value='add'
    work.name=''
  }
  form.value.reset()
}
//edit Task
const editTask=(id)=>{
  const task=tasks.find(t=>t.id===id)
  work.id=task.id;
  work.name=task.name;
  work.status=task.status;
  typeSubmit.value='edit'
}
//delete Task
const deletetask=(id)=>{
  axios.delete(`http://localhost:3000/tasks/${id}`)
   .then(()=>tasks.splice(tasks.findIndex(t=>t.id===id),1))
   .catch(err=>console.log(err))
}
//update Task status
const updateStatus=(task)=>{  
  axios.put(`http://localhost:3000/tasks/${task.id}`,{...task})
   .then(res=>{
    tasks.splice(tasks.findIndex(t=>t.id===task.id),1,res.data)})
   .catch(err=>console.log(err))
}
const handleTaskDone=()=>{
  tasks.length=0;
   axios.get('http://localhost:3000/tasks?status=true')
   .then(res=>tasks.push(...res.data))
}
const handleTaskNotDone=()=>{
  tasks.length=0;
   axios.get('http://localhost:3000/tasks?status=false')
   .then(res=>tasks.push(...res.data))
}
  
const handleTasks=()=>{
  tasks.length=0;
  getAllTasksAPI();

}
</script>

<style>
.line-through {
  text-decoration: line-through;
}
</style>
<template>
  <div class="flex gap-1 md-flex-col">
    <div class="flex flex-col">
      <label>Task</label>
      <InputText v-model="taskDescription"></InputText>
    </div>
    <div class="flex flex-col">
      <label> Duration </label>
      <div class="flex gap-1">
        <InputText v-model="durationTask"></InputText>
        <Dropdown
          v-model="selectedDescDuration"
          :options="descriptionDuration"
          optionLabel="name"
        ></Dropdown>
        <Button @click="createTask()" label="Add New Task"></Button>
      </div>
    </div>
  </div>
  <div class="flex mt-1 gap-0">
    <Checkbox v-model="active" :value="true">
    </Checkbox>
    <label>
      Active
    </label>
  </div>
  <div>
    <DataTable :value="taskList" tableStyle="min-width: 50rem">
      <Column field="id" header="ID"></Column>
      <Column field="task" header="Task Description">
        <template #body="slotProps">
          <div v-if="editable">
            <InputText
              v-model="slotProps.data.task"
              :value="slotProps.data.task"
            ></InputText>
          </div>
          <div v-else>
            <span>
              {{ slotProps.data.task }}
            </span>
          </div></template
        >
      </Column>
      <Column field="duration" header="Duration"> </Column>
      <Column field="deletedAt" header="Deleted At"></Column>
      <Column header="Action">
        <template #body="slotProps">
          <div v-if="!editable" class="flex gap-0 md-flex-col">
            <Button label="Edit" @click="editText(slotProps.data.id)"></Button>
            <Button
              label="Soft Delete"
              @click="deleteTask(slotProps.data.id, 'soft')"
            ></Button>
            <Button
              label="Hard Delete"
              @click="deleteTask(slotProps.data.id, 'hard')"
            ></Button>
          </div>
          <div v-else>
            <Button label="Update" @click="updateTask(slotProps.data)"></Button>
            <Button label="Cancel" @click="cancelEdit()"></Button>
          </div>
        </template>
      </Column>
      <template #empty> No data found. </template>
    </DataTable>
  </div>
</template>

<script setup>
const active = ref(false)
const taskList = ref();
const taskDescription = ref();
const durationTask = ref();
const editable = ref(false);
const taskDescriptionList = ref();
const descriptionDuration = ref([
  { name: "Seconds", code: "s" },
  { name: "Minute", code: "m" },
  { name: "Hour", code: "h" },
  { name: "Day", code: "d" },
  { name: "Week", code: "w" },
  { name: "Month", code: "m" },
  { name: "Year", code: "y" },
]);
const selectedDescDuration = ref();
onMounted(() => {
  loadData();
});
async function loadData(state) {
  try {
    if (state === true) {
      const response = await $fetch(`/api/todos?active=${true}`);
      taskList.value = response;
    } else {
      const response = await $fetch(`/api/todos`);
      taskList.value = response;
    }
  } catch (error) {
    console.error("Cannot Access Server");
  }
}
async function createTask() {
  // TODO : Implement Create Task to Backend
  const data = {
    task: taskDescription.value,
    duration: durationTask.value + selectedDescDuration.value.code,
  }

  try {
    const res = await $fetch('/api/todos', {
      body: data,
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      }
    })

    loadData()
    taskDescription.value = ''
    durationTask.value = ''
    selectedDescDuration.value = ''
  } catch (error) {
    console.log(error)
  }
}
async function editText() {
  editable.value = true;
}
async function cancelEdit() {
  editable.value = false;
  loadData()
}
async function updateTask(data) {
  // TODO : Implement Update Task to Backend
  try {
    const res = await $fetch(`/api/todos/${data.id}`, {
      method: 'PATCH',
      body: {
        task: data.task,
        duration: data.duration
      }
    })

    // loadData()
    // console.log(res)
    editable.value = false
    loadData()
  } catch (error) {
    console.log(error)
  }
}
async function deleteTask(id, typeDelete) {
  // TODO : Implement Condition Hard Delete
  try {
    // const url = typeDelete === 'hard' ? 
    const response = await $fetch(`/api/todos/${id}?`, {
      method: "DELETE",
    });
    return response;
  } catch (error) {
    console.error(error);
  }
}

watch(
  active,
  () => {
    if (active.value[0]) {
      loadData(true)
    } else {
      loadData()
    }
  }
)
</script>

<style scoped>
.flex{
  display: flex;
}
.flex-col{
  flex-direction: column;
}
.gap-1{
  gap: 1rem;
}
.mt-1{
  margin-top: 1rem;
}
.gap-0{
  gap: .5rem;
}

@media screen and (max-width: 900px){
  .md-flex-col{
    flex-direction: column;
  }
}
</style>

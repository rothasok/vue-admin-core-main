<template>
  <Card> <!-- <Button text="Open Modal" @click="toggleModal" /> -->
    <Modal :title="$t('Document-In-Create')" label="Modal Create" :activeModal="show" @close="show = false">
      <Textinput label="Document No" name="num" type="text" placeholder="Document No" v-model="num" />
      <br>
      <Textarea label="Title" name="title" placeholder="Title" v-model="title" />
      <br>

      <Fileinput label="Document" @change="onFileChange" />
      <br>
      <Button icon="heroicons-outline:plus-sm" text="Save" btnClass=" btn-success font-normal btn-sm "
        iconClass="text-lg" @click="uploadFile" />

      <Button icon="heroicons-outline:plus-sm" text="Close" btnClass=" btn-primary font-normal btn-sm "
        iconClass="text-lg" @click="show = false" />
    </Modal>
  </Card>


  <Card> <!-- <Button text="Open Modal" @click="toggleModal" /> -->
    <Modal_eidt :title="$t('Document-In-Edit')" label="Modal Edit" :activeModal="show1" @close="show1 = false">
      <Textinput label="Document No" name="num" type="text" placeholder="Document No" v-model="numEdit"
        :disabled="isDisabled" />
      <br>
      <Textarea label="Title" name="title" placeholder="Title" v-model="titleEdit" :disabled="isDisabled" />
      <br>

      <!-- <Fileinput label="Document" @change="onFileChange" /> -->
      <br>
      <div v-if="image" class="py-4">

        <img :src="image" alt="Fetched Image" style="max-width: 100%;" />
      </div>
      <!-- <p v-if="errorMessage" style="color: red;">{{ errorMessage }}</p> -->

      <Button icon="heroicons-outline:plus-sm" text="Edit" btnClass=" btn-success font-normal btn-sm "
        iconClass="text-lg" @click="editFile" v-if="btnUpdate" />
      <Button icon="heroicons-outline:plus-sm" text="Update" btnClass=" btn-success font-normal btn-sm "
        iconClass="text-lg" @click="updaeFiie" v-if="isVisible" />
      <Button icon="heroicons-outline:plus-sm" text="Close" btnClass=" btn-primary font-normal btn-sm "
        iconClass="text-lg" @click="show1 = false && isDisabled " />
    </Modal_eidt>
  </Card>

  <div class="space-y-5 profile-page">

    <div class="grid grid-cols-12 gap-6">
      <div class="lg:col-span-12 col-span-12 ">
        <Card :title="$t('Document-In')">
          <div class="py-4">
            <!-- <Button icon="heroicons-outline:plus-sm" :text="$t('create_new')" btnClass=" btn-success font-normal btn-sm "
          iconClass="text-lg" @click="$emit(createFile())" /> -->
            <Button icon="heroicons-outline:plus-sm" :text="$t('create_new')"
              btnClass=" btn-success font-normal btn-sm " iconClass="text-lg" @click="toggleModal" />
          </div>
          <vue-good-table :columns="columns" :rows="files" styleClass=" vgt-table bordered centered"
            :sort-options="{ enabled: ture }" :pagination-options="{
              enabled: true,
              perPage: perpage,
            }" :search-options="{
              enabled: true,
              externalQuery: searchTerm,
            }" :select-options="{
              enabled: true,
              selectOnCheckboxOnly: true, // only select when checkbox is clicked instead of the row
              selectioninfoClass: 'custom-class',
              selectionText: 'rows selected',
              clearSelectionText: 'clear',
              disableSelectinfo: true, // disable the select info-500 panel on top
              selectAllByGroup: true, // when used in combination with a grouped table, add a checkbox in the header row to check/uncheck the entire group
            }">
            <template #table-row="props">
              <span v-if="props.column.field == 'action'" class="ml-2">
                <!-- <button type="button" class="btn btn-warning" v-on:click="edit(props.row.id)"><i
              class="fas fa-edit"></i></button> -->
                <!-- <Button icon="heroicons:book-open" :text="$t('view')" btnClass=" btn-primary font-normal btn-sm "
                  iconClass="text-ls" @click="viewFile(props.row._id)" /> -->

              </span>
              <span v-if="props.column.field == 'action'" class="ml-2">
                <!-- <button type="button" class="btn btn-warning" v-on:click="edit(props.row.id)"><i
            class="fas fa-edit"></i></button> -->
                <Button icon="heroicons:pencil-square" :text="$t('view')" btnClass=" btn-warning font-normal btn-sm "
                  iconClass="text-ls" @click="toggleModalEdit(props.row._id)" />

              </span>
              <span v-if="props.column.field == 'action'" class="ml-2">
                <!-- <button type="button" class="btn btn-danger " v-on:click="remove(props.row.id)"><i
            class="fas fa-trash-alt"></i></button> -->
                <Button icon="heroicons-outline:trash" :text="$t('delete')" btnClass=" btn-danger font-normal btn-sm "
                  iconClass="text-ls" @click="deleteFile(props.row._id)" />
              </span>
              <span v-else>
                {{ props.formattedRow[props.column.field] }}
              </span>
            </template>

          </vue-good-table>

        </Card>
      </div>

    </div>


  </div>
</template>


<script>
import Modal from "@/components/Modal";
import Modal_eidt from "@/components/Modal";
import Select from "@/components/Select";

import Card from "@/components/Card";
import { useToast } from "vue-toastification";
// import Modal from "@/components/Modal/Modal.vue";

import Icon from "@/components/Icon";
import Button from "@/components/Button";
import Fileinput from "@/components/Fileinput"
import 'vue-good-table-next/dist/vue-good-table-next.css'
import { VueGoodTable } from 'vue-good-table-next';
import axios from "axios";
import Textinput from "@/components/Textinput";
import Textarea from "@/components/Textarea";
import { inject, onMounted, ref, watch } from 'vue';

// import { ref } from 'vue';
// import profileImg from "@/assets/images/avatar/avatar.png"

export default {
  components: {
    Card,
    Icon,
    Button,
    VueGoodTable,
    Fileinput,
    Modal,
    Textinput,
    Textarea,
    Modal,
    Modal_eidt,
    Select,
  },
  setup() {

    const services = inject('services')
    //const selectedId = ref(null);
    const show = ref(false);
    const show1 = ref(false);
    const toast = useToast();
    //testing
    const image = ref("");
    const numEdit = ref("");
    const titleEdit = ref("");
    const files = ref([]);
    const btnUpdate = ref(true);
    const isVisible = ref(false);

    const isDisabled = ref(true);
    const token = JSON.parse(localStorage.getItem('activeUser'))


    const toggleModal = () => {
      show.value = !show.value;
    };
    onMounted(async () => {
      /** load data select */
      getFiles()

    });

    const getFiles = async () => {
      files.value =[]
      try {
        const response = await services.get(`files`)
        files.value = response.data
        console.log(files.value)
      } catch (error) {
        console.log(error)
      }
    };
    const editFile = () => {
      isVisible.value = !isVisible.value; // Toggle visibility
      btnUpdate.value = !btnUpdate.value; // Toggle visibility
      isDisabled.value = !isDisabled.value; // Toggle visibility


    };
    const deleteFile = async (id) => {
      try {
        services.common_remove(`files/${id}`)
      }
      //console.log(response.data)
      catch (error) {
        console.error('Error delete file:', error);
        return toast.warning(error);
      }
      toast.success("File deleted success")
      getFiles()
    }
    const toggleModalEdit = async (id) => {
      
      show1.value = !show1.value;
      //get data from file
      try{
      const response = await services.get(`files/text/${id}`)
        const fileText = response.data
        numEdit.value = fileText.num
        titleEdit.value = fileText.title  
      } catch (error) {
        console.log(error)
      }
      ///get file
      try {
        const response = await services.file_get(`files/${id}`)
        const fileBlob = new Blob([response.data]);
        //console.log(fileBlob)
        const Url = URL.createObjectURL(fileBlob);
        image.value = Url
      } catch (error) {
        console.log(error)
      }
    };

    return {
      toggleModal, toggleModalEdit, editFile, getFiles, deleteFile,
      show, show1, toast, isDisabled,files,token,
      image, numEdit, titleEdit, isVisible, btnUpdate
    }
  },
  data() {
    return {
      //token: JSON.parse(localStorage.getItem('activeUser')),
      services : inject('services'),
      file_id: '',
      type: 'In',
      file: [],
      userRole:'',
      num:'',
      title:'',
      file_id: '',
      selectedFile: null,
      validExtensions: ['jpg', 'jpeg', 'png', 'gif', 'pdf'],
      maxSize: 2 * 1024 * 1024, // 2 MB,
      columns: [
        {
          label: "ID",
          field: "_id",
        },
        {
          label: "Number",
          field: "num",
        
        },
        {
          label: "Title",
          field: "title",
          type: "string",
        },
        {
          label: "Name",
          field: "originalname",
          type: "string",
        },
        {
          label: "Created On",
          field: "createdDate",
        },
        {
          label: 'Action',
          field: 'action',
          tdClass: 'text-center',
          thClass: 'text-center',
          sortable: false,
        },

      ],

    };
  },

  methods: {
    onFileChange(event) {
      console.log('event')
      const file = event.target.files[0];
      if (file) {
        const fileExtension = file.name.split('.').pop().toLowerCase();
        if (!this.validExtensions.includes(fileExtension)) {
          this.toast.warning("Invalid file type. Only jpg, jpeg, png, and gif files are allowed.");
          //alert("Invalid file type. Only jpg, jpeg, png, and gif files are allowed.");
          this.selectedFile = null;
          return;
        }
        if (file.size > this.maxSize) {
          alert("File size exceeds the 2 MB limit.");
          this.selectedFile = null;
          return;
        }
        //console.log(file)
        this.selectedFile = file;
      }
    },
    async uploadFile() {
      //console.log(this.num)     
      if (!this.selectedFile) {
        this.toast.warning("Please select a valid file first!");
        return;
      }
      // Create a FormData instance
      const userRole = JSON.parse(localStorage.getItem('userData'))
      this.userRole = userRole.role._id

      const formData = new FormData();
      formData.append('file', this.selectedFile);
      formData.append('role', this.userRole);
      formData.append('type', this.type);
      formData.append('num', this.num);
      formData.append('title', this.title);
      // console.log(formData)
      console.log(this.title)
      try {
        const response = await this.services.common_post('files/upload-single/', formData) 
       
        // //const response = await axios.post('http://localhost:3000/v1/files/upload-single/', formData, {
        //   headers: {
        //     'Content-Type': 'multipart/form-data',
        //     authorization: `Bearer ${this.token.accessToken}`, // Add your here
        //   }
        }
        //console.log(response.data)
        // console.log("hhhhhhhhhhhhh")
      catch (error) {
        console.error('Error uploading file:', error);
        return this.toast.warning(error);

      }

      this.getFiles()
      this.show = false
      //location.reload()


      return this.toast.success("File upload success");

    },



  }

}

</script>
<style lang="scss" scoped></style>
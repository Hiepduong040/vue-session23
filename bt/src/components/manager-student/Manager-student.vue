<template>
    <div class="container mx-auto my-4 h-[100vh]">
      <main class="main-content">
        <!-- Header -->
        <header class="flex justify-between mb-3">
          <h3>Quản lý học viên</h3>
          <button class="btn btn-success" @click="showStudentForm">
            Thêm học viên mới
          </button>
        </header>
  
        <!-- Search Bar -->
        <div class="flex items-center justify-end gap-2 mb-3">
          <input
            v-model="searchTerm"
            style="width: 350px"
            type="text"
            class="form-input"
            placeholder="Tìm kiếm theo tên học viên"
          />
          <font-awesome-icon
            icon="arrows-rotate"
            title="Làm mới"
            @click="reloadData"
          />
        </div>
  
        <!-- Students List -->
        <table class="table-auto border-collapse w-full">
          <thead>
            <tr>
              <th>STT</th>
              <th>Tên học viên</th>
              <th>Email</th>
              <th>Địa chỉ</th>
              <th colspan="2">Hành động</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(learner, idx) in filteredLearners" :key="learner.id">
              <td>{{ idx + 1 }}</td>
              <td>{{ learner.name }}</td>
              <td>{{ learner.email }}</td>
              <td>{{ learner.address }}</td>
              <td>
                <span @click="editLearner(learner)" class="btn btn-edit">Sửa</span>
              </td>
              <td>
                <span @click="showDeleteConfirmation(learner)" class="btn btn-delete">Xóa</span>
              </td>
            </tr>
          </tbody>
        </table>
  
      </main>
  
      <!-- Form for adding/editing learners -->
      <div v-if="isFormVisible" class="modal-overlay">
        <form @submit.prevent="handleFormSubmit" class="modal-form">
          <div class="flex justify-between items-center">
            <h4>{{ formMode === "add" ? "Thêm mới học viên" : "Chỉnh sửa học viên" }}</h4>
            <font-awesome-icon icon="xmark" @click="hideForm" />
          </div>
          <div>
            <label class="form-label" for="learnerName">Tên học viên</label>
            <input
              v-model="formData.name"
              id="learnerName"
              type="text"
              class="form-input"
            />
            <small v-if="formErrors.name" class="text-error">{{ formErrors.name }}</small>
          </div>
          <div>
            <label class="form-label" for="email">Email</label>
            <input
              v-model="formData.email"
              id="email"
              type="email"
              class="form-input"
            />
            <small v-if="formErrors.email" class="text-error">{{ formErrors.email }}</small>
          </div>
          <div>
            <label class="form-label" for="address">Địa chỉ</label>
            <input
              v-model="formData.address"
              id="address"
              type="text"
              class="form-input"
            />
  <small v-if="formErrors.address" class="text-error">{{ formErrors.address }}</small>
          </div>
          <div>
            <button class="w-full btn btn-success">
              {{ formMode === "add" ? "Thêm mới" : "Cập nhật" }}
            </button>
          </div>
        </form>
      </div>
  
      <!-- Confirmation Modal for Deleting -->
      <div v-if="isDeleteModalVisible" class="modal-overlay">
        <div class="modal-content">
          <div class="modal-header">
            <h4>Cảnh báo</h4>
            <i class="fa-solid fa-xmark" @click="hideDeleteModal"></i>
          </div>
          <div class="modal-body">
            <span>Bạn có chắc chắn muốn xóa học viên này?</span>
          </div>
          <div class="modal-footer">
            <button class="btn btn-light" @click="hideDeleteModal">Hủy</button>
            <button class="btn btn-danger" @click="removeLearner">Xác nhận</button>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, computed, onMounted } from "vue";
  import axios from "axios";
  
  const learners = ref([]);
  const searchTerm = ref("");
  const isFormVisible = ref(false);
  const formData = ref({ name: "", email: "", address: "" });
  const formMode = ref("add");
  const isDeleteModalVisible = ref(false);
  let selectedLearner = ref(null);
  const formErrors = ref({});
  
  const fetchLearners = async () => {
    try {
      const response = await axios.get("http://localhost:3000/learners");
      learners.value = response.data;
    } catch (error) {
      console.error(error);
    }
  };
  
  const filteredLearners = computed(() => {
    if (!searchTerm.value) return learners.value;
    return learners.value.filter((learner) =>
      learner.name.toLowerCase().includes(searchTerm.value.toLowerCase())
    );
  });
  
  const validateForm = () => {
    formErrors.value = {};
    if (!formData.value.name) {
      formErrors.value.name = "Tên học viên không được để trống.";
    }
    if (!formData.value.email || !formData.value.email.includes("@")) {
      formErrors.value.email = "Email không hợp lệ.";
    }
    if (!formData.value.address) {
      formErrors.value.address = "Địa chỉ không được để trống.";
    }
    return Object.keys(formErrors.value).length === 0;
  };
  
  const addLearner = async () => {
    try {
      const response = await axios.post("http://localhost:3000/learners", formData.value);
      learners.value.push(response.data);
      hideForm();
    } catch (error) {
      console.error(error);
    }
  };
  
  const updateLearner = async () => {
    try {
      await axios.put(`http://localhost:3000/learners/${formData.value.id}`, formData.value);
      const index = learners.value.findIndex((learner) => learner.id === formData.value.id);
      if (index !== -1) learners.value[index] = { ...formData.value };
      hideForm();
    } catch (error) {
      console.error(error);
    }
  };
  
  const handleFormSubmit = () => {
    if (validateForm()) {
      if (formMode.value === "add") {
        addLearner();
      } else {
        updateLearner();
  }
    }
  };
  
  const showStudentForm = () => {
    isFormVisible.value = true;
    formMode.value = "add";
    formData.value = { name: "", email: "", address: "" };
  };
  
  const hideForm = () => {
    isFormVisible.value = false;
    formErrors.value = {};
  };
  
  const editLearner = (learner) => {
    formData.value = { ...learner };
    formMode.value = "edit";
    isFormVisible.value = true;
  };
  
  const removeLearner = async () => {
    try {
      await axios.delete(`http://localhost:3000/learners/${selectedLearner.value.id}`);
      learners.value = learners.value.filter((l) => l.id !== selectedLearner.value.id);
      hideDeleteModal();
    } catch (error) {
      console.error(error);
    }
  };
  
  const showDeleteConfirmation = (learner) => {
    selectedLearner.value = learner;
    isDeleteModalVisible.value = true;
  };
  
  const hideDeleteModal = () => {
    isDeleteModalVisible.value = false;
    selectedLearner.value = null;
  };
  
  const reloadData = () => {
    searchTerm.value = "";
  };
  
  onMounted(() => {
    fetchLearners();
  });
  </script>
  
  <style scoped>
  .container {
    max-width: 1200px;
    margin: auto;
  }
  
  .main-content {
    padding: 20px;
    background: #f9f9f9;
    border-radius: 8px;
  }
  
  .table-auto {
    border: 1px solid #ccc;
  }
  
  .form-input {
    border: 1px solid #ccc;
    padding: 8px;
    border-radius: 4px;
  }
  
  .btn {
    padding: 8px 12px;
    border-radius: 4px;
    cursor: pointer;
  }
  
  .btn-success {
    background-color: #28a745;
    color: white;
  }
  
  .btn-edit {
    background-color: #ffc107;
    color: white;
  }
  
  .btn-delete {
    background-color: #dc3545;
    color: white;
  }
  
  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.7);
    display: flex;
    justify-content: center;
    align-items: center;
  }
  
  .modal-form, .modal-content {
    background: white;
    padding: 20px;
    border-radius: 8px;
    width: 400px;
  }
  
  .text-error 
  .text-error {
    color: red;
    font-size: 0.875rem;
  }
  
  .modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .modal-body {
    margin: 15px 0;
  }
  
  .modal-footer {
    display: flex;
    justify-content: flex-end;
    gap: 10px;
  }
  </style>
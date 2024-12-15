<template>
    <div class="container">
      <div class="card">
        <div class="card-header">
          <h4>
            Students
            <RouterLink to="/students/create" class="btn btn-primary float-end">
              Add Student
            </RouterLink>
          </h4>
        </div>
        <div class="card-body">
          <!-- Search bar -->
          <input 
            type="text" 
            v-model="searchQuery" 
            @input="getStudents" 
            class="form-control mb-3" 
            placeholder="Search by Name, Course, Email, or Phone"
          />
  
          <table class="table table-bordered">
            <thead>
              <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Course</th>
                <th>Email</th>
                <th>Phone</th>
                <th>Created At</th>
                <th>Action</th>
              </tr>
            </thead>
            <tbody v-if="students.length > 0">
              <tr v-for="(student, index) in students" :key="index">
                <td>{{ student.id }}</td>
                <td>{{ student.name }}</td>
                <td>{{ student.course }}</td>
                <td>{{ student.email }}</td>
                <td>{{ student.phone }}</td>
                <td>{{ student.created_at }}</td>
                <td>
                  <RouterLink :to="{ path: '/students/' + student.id + '/edit' }" class="btn btn-success">
                    Edit
                  </RouterLink>
                  <button type="button" @click="deleteStudent(student.id)" class="btn btn-danger">
                    Delete
                  </button>
                </td>
              </tr>
            </tbody>
            <tbody v-else>
              <tr>
                <td colspan="7">Loading...</td>
              </tr>
            </tbody>
          </table>
  
          <!-- Pagination -->
          <nav v-if="pagination.total_items > 0">
            <ul class="pagination justify-content-center">
              <!-- Previous button -->
              <li class="page-item" :class="{ disabled: pagination.current_page === 1 }">
                <button class="page-link" @click="changePage(pagination.current_page - 1)" :disabled="pagination.current_page === 1">
                  Previous
                </button>
              </li>
  
              <!-- Page number buttons -->
              <li
                class="page-item"
                v-for="page in totalPages"
                :key="page"
                :class="{ active: page === pagination.current_page }"
              >
                <button class="page-link" @click="changePage(page)">
                  {{ page }}
                </button>
              </li>
  
              <!-- Next button -->
              <li class="page-item" :class="{ disabled: pagination.current_page === pagination.total_pages }">
                <button class="page-link" @click="changePage(pagination.current_page + 1)" :disabled="pagination.current_page === pagination.total_pages">
                  Next
                </button>
              </li>
            </ul>
          </nav>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import axios from 'axios'
  
  export default {
    name: 'students',
    data() {
      return {
        students: [],
        pagination: {},
        searchQuery: '', // Search query value
        limit: 10, // Items per page
      }
    },
    mounted() {
      this.getStudents(); // Fetch students on initial load
    },
    computed: {
      totalPages() {
        let pages = [];
        for (let i = 1; i <= this.pagination.total_pages; i++) {
          pages.push(i);
        }
        return pages;
      },
    },
    methods: {
      // Fetch students with optional search and pagination
      getStudents(page = 1) {
        axios
          .get('http://127.0.0.1:8000/api/students', {
            params: {
              search: this.searchQuery,
              limit: this.limit,
              page: page,  // Pass the current page to the API
            },
          })
          .then((res) => {
            this.students = res.data.students;  // The student records
            this.pagination = res.data.pagination;  // The pagination info
          })
          .catch((error) => {
            console.log(error);
          })
      },
  
      // Change page based on clicked pagination link
      changePage(page) {
        if (page >= 1 && page <= this.pagination.total_pages) {
          this.getStudents(page);  // Pass the new page to getStudents
        }
      },
  
      deleteStudent(studentId) {
        if (confirm('Are you sure you want to delete this data?')) {
          axios
            .delete(`http://127.0.0.1:8000/api/students/${studentId}/delete`)
            .then((res) => {
              alert(res.data.message);
              this.getStudents(); // Refresh the students list
            })
            .catch((error) => {
              if (error.response && error.response.status === 404) {
                alert(error.response.data.message);
              }
            })
        }
      },
    },
  }
  </script>
  
  <style scoped>
  /* Add your custom styles here if needed */
  </style>
  
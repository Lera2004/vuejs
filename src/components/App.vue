<template>
    <div>
        <input type="text" id="search-input" v-model="search" placeholder="Пошук за прізвищем" />

        <form @submit.prevent="addStudent">
            <input type="url" v-model="newStudent.photo" placeholder="Фото" />
            <input type="text" v-model="newStudent.name" placeholder="ПІБ" required />
            <select v-model="newStudent.group" placeholder="Група">
                <option value="RPZ 20 1/9">RPZ 20 1/9</option>
                <option value="RPZ 20 2/9">RPZ 20 2/9</option>
            </select>
            <input type="number" v-model="newStudent.mark" placeholder="Оцінка" required />
            <input type="checkbox" v-model="newStudent.isDonePr" />
            <button type="submit">Додати</button>
        </form>

        <table>
            <thead>
                <tr>
                    <th>Фото</th>
                    <th>ПІБ</th>
                    <th>Група</th>
                    <th>Оцінка</th>
                    <th>Практика</th>
                    <th>Дії</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="student in filteredStudents" :key="student._id">
                    <td>
                        <img v-if="student.photo" :src="student.photo" alt="Фото студента" width="50" />
                        <img v-else src="" alt="Фото відсутнє" width="50" />
                    </td>
                    <td>
                        <span v-if="student._id !== editingStudentId">{{ student.name }}</span>
                        <input v-else v-model="student.name" />
                    </td>
                    <td>
                        <span v-if="student._id !== editingStudentId">{{ student.group }}</span>
                        <select v-else v-model="student.group">
                            <option value="RPZ 20 1/9">RPZ 20 1/9</option>
                            <option value="RPZ 20 2/9">RPZ 20 2/9</option>
                        </select>
                    </td>
                    <td>
                        <span v-if="student._id !== editingStudentId">{{ student.mark }}</span>
                        <input v-else type="number" v-model="student.mark" />
                    </td>
                    <td>
                        <span v-if="student._id !== editingStudentId">
                            <span v-if="student.isDonePr">&#10004;</span>
                            <span v-else>&#10008;</span>
                        </span>
                        <input v-else type="checkbox" v-model="student.isDonePr" />
                    </td>
                    <td>
                        <button @click="editStudent(student)" v-if="student._id !== editingStudentId" style="margin-bottom: 10px;">
                            <i class="fas fa-pencil-alt" ></i> Редагувати
                        </button>
                        <button @click="updateStudent(student)" v-else style="margin-bottom: 10px;">
                            <i class="fas fa-check"></i> Оновити
                        </button>
                        <button @click="removeStudent(student._id)" >
                            <i class="fas fa-trash-alt"></i> Видалити
                        </button>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</template>

<script>
import axios from "axios";

export default {
    data() {
        return {
            students: [],
            search: "",
            newStudent: {
                name: "",
                group: "",
                mark: null,
                isDonePr: false,
                photo: "",
            },
            editingStudentId: null,
        };
    },
    computed: {
        filteredStudents() {
            const searchQuery = this.search.toLowerCase();
            return this.students.filter(student => student.name.toLowerCase().includes(searchQuery));
        }
    },
    created() {
        this.fetchStudents();
    },
    methods: {
        fetchStudents() {
            axios
                .get("http://34.82.81.113:3000/students")
                .then((response) => {
                    this.students = response.data;
                })
                .catch((error) => {
                    console.error("Помилка при завантаженні студентів:", error);
                });
        },
        removeStudent(id) {
            axios
                .delete(`http://34.82.81.113:3000/students/${id}`)
                .then(() => {
                    this.fetchStudents();
                })
                .catch((error) => {
                    console.error("Помилка при видаленні студента:", error);
                });
        },
        addStudent() {
            axios
                .post("http://34.82.81.113:3000/students", this.newStudent)
                .then(() => {
                    this.fetchStudents();
                    this.newStudent = {
                        name: "",
                        group: "",
                        mark: null,
                        isDonePr: false,
                        photo: "",
                    };
                })
                .catch((error) => {
                    console.error("Помилка при додаванні студента:", error);
                });
        },
        editStudent(student) {
            this.editingStudentId = student._id;
        },
        updateStudent(student) {
            axios
                .put(`http://34.82.81.113:3000/students/${student._id}`, student)
                .then(() => {
                    this.editingStudentId = null;
                })
                .catch((error) => {
                    console.error("Помилка при оновленні студента:", error);
                });
        },
    },
};
</script>

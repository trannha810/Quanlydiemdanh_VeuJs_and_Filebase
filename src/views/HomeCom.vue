<template>
  <div>
    <h1 style="text-align: center; margin-top: 20px">Danh sách lớp</h1>
    <div>
      <input
        type="text"
        v-model="search"
        style="
          width: 30%;
          height: 40px;
          border-radius: 20px 0px 0px 20px;
          padding: 20px;
        "
        placeholder="Tìm kiếm..."
      />
      <button style="border-radius: 0px 20px 20px 0px; height: 43px; width: 40px">
        <i class="fa fa-search" aria-hidden="true"></i>
      </button>
    </div>
    <div class="card m-2" style="background-color: rgba(40, 167, 69, 0)">
      <div class="card-body">
        <table class="table table-striped table-inverse table-responsive" style="margin-bottom: 50px">
          <thead class="thead-inverse">
            <tr>
              <th>Mã lớp</th>
              <th>Giảng viên</th>
              <th>Tên lớp</th>
              <th>Số sinh viên</th>
              <th>Số tiết</th>
              <th>Tùy chọn</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="lop in Searching" :key="lop.id">
              <td scope="row">{{ lop.id }}</td>
              <td>{{ lop.giangvien }}</td>
              <td>{{ lop.tenlop }}</td>
              <td>{{ lop.soluong }}</td>
              <td>{{ lop.sotiet }}</td>
              <td>
                <router-link :to="{ path: `/class/${lop.id}` }">
                  <button type="button" class="btn btn-success">Chi tiết</button>
                </router-link>
                <button type="button" class="btn btn-primary" @click="editLop(lop)">Sửa</button>
                <button type="button" class="btn btn-danger" @click="deleteLop(lop)">Xóa</button>
              </td>
            </tr>
          </tbody>
        </table>

        <h3>Thêm lớp mới</h3>
        <form @submit.prevent="addLop">
          <div class="form-group">
            <label for="giangvien">Giảng viên:</label>
            <input type="text" class="form-control" id="giangvien" v-model="newLop.giangvien" required>
          </div>
          <div class="form-group">
            <label for="tenlop">Tên lớp:</label>
            <input type="text" class="form-control" id="tenlop" v-model="newLop.tenlop" required>
          </div>
          <div class="form-group">
            <label for="soluong">Số sinh viên:</label>
            <input type="number" class="form-control" id="soluong" v-model="newLop.soluong" required>
          </div>
          <div class="form-group">
            <label for="sotiet">Số tiết:</label>
            <input type="number" class="form-control" id="sotiet" v-model="newLop.sotiet" required>
          </div>
          <button type="submit" class="btn btn-success">Thêm</button>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onUnmounted } from "vue";
import { getFirestore, collection, query, onSnapshot, addDoc, deleteDoc, updateDoc, doc } from "firebase/firestore";
import { initializeApp } from "firebase/app";

const firebaseConfig = {
  // Thông tin cấu hình Firebase của bạn
  apiKey: "AIzaSyCk0TFvgdSampCjkzPuEFoUOco-qUu5JoY",
  authDomain: "nhatu-51c28.firebaseapp.com",
  projectId: "nhatu-51c28",
  storageBucket: "nhatu-51c28.appspot.com",
  messagingSenderId: "558381258516",
  appId: "1:558381258516:web:e9ee68b3fd62dd54471401",
  measurementId: "G-BVBX4GQ956"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

export default {
  data() {
    return {
      lop: ref([]),
      search: "",
      newLop: {
        giangvien: "",
        tenlop: "",
        soluong: 0,
        sotiet: 0
      }
    };
  },
  mounted() {
    const latestQuery = query(collection(db, "lop"));

    const livemessages = onSnapshot(latestQuery, (snapshot) => {
      this.lop = snapshot.docs.map((doc) => {
        return {
          id: doc.id,
          giangvien: doc.data().giangvien,
          tenlop: doc.data().tenlop,
          soluong: doc.data().soluong,
          sotiet: doc.data().sotiet
        };
      });
    });
    onUnmounted(() => {
      livemessages();
    });
  },
  computed: {
Searching() {
  if (this.search && typeof this.search === 'string') {
    return this.lop.filter((lop) => {
      return (
        lop.giangvien.toLowerCase().includes(this.search.toLowerCase()) ||
        lop.tenlop.toLowerCase().includes(this.search.toLowerCase())
      );
    });
  } else {
    return this.lop;
  }
}
  },
  methods: {
    addLop() {
      addDoc(collection(db, "lop"), this.newLop)
        .then(() => {
          this.newLop.giangvien = "";
          this.newLop.tenlop = "";
          this.newLop.soluong = 0;
          this.newLop.sotiet = 0;
        })
        .catch((error) => {
          console.error("Error adding document: ", error);
        });
    },
    deleteLop(lop) {
      const lopRef = doc(db, "lop", lop.id);
      deleteDoc(lopRef)
        .then(() => {
          console.log("Document successfully deleted!");
        })
        .catch((error) => {
          console.error("Error removing document: ", error);
        });
    },
    editLop(lop) {
      const lopRef = doc(db, "lop", lop.id);
      updateDoc(lopRef, {
        giangvien: lop.giangvien,
        tenlop: lop.tenlop,
        soluong: lop.soluong,
        sotiet: lop.sotiet
      })
        .then(() => {
          console.log("Document successfully updated!");
        })
        .catch((error) => {
          console.error("Error updating document: ", error);
        });
    }
  }
};
</script>

<style>
body {
  font-family: "Roboto", sans-serif;
  background-color: #f8f9fa;
}

.card {
  border-radius: 15px;
  border: none;
  box-shadow: 0px 0px 15px rgba(0, 0, 0, 0.1);
  background-color: #ffffff;
}

.table thead th {
  border-top: none;
  border-bottom: none;
  font-weight: 500;
  font-size: 14px;
  color: #000;
}

.table td {
  font-weight: 400;
  font-size: 14px;
  color: #000;
}

.btn-success {
  background-color: #28a745;
  border: none;
}

.btn-primary {
  background-color: #007bff;
  border: none;
}

.btn-danger {
  background-color: #dc3545;
  border: none;
}

.btn-success:hover {
  background-color: #218838;
}

.btn-primary:hover {
  background-color: #0069d9;
}

.btn-danger:hover {
  background-color: #c82333;
}
</style>
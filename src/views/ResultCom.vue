<template>
  <div class="container mt-3">
    <h2>Thống Kê</h2>
    <div class="row">
      <div class="col-md-4 mb-3 col-sm-6">
        <div
          class="card text-dark"
          style="background-color: rgba(255, 193, 7, 0.5)"
        >
          <div class="card-body">
            <h4 class="card-title">Sinh Viên</h4>
            <h3 class="card-text">
              <b>{{ sinhvienCount }}</b>
            </h3>
          </div>
        </div>
      </div>
      <div class="col-md-4 mb-3 col-sm-6">
        <div class="card" style="background-color: rgba(23, 162, 184, 0.5)">
          <div class="card-body">
            <h4 class="card-title">Giảng Viên</h4>
            <h3 class="card-text">
              <b>{{ giangvienCount }}</b>
            </h3>
          </div>
        </div>
      </div>
      <div class="col-md-4 mb-3 col-sm-6">
        <div class="card" style="background-color: rgba(40, 167, 69, 0.5)">
          <div class="card-body">
            <h4 class="card-title">Môn Học</h4>
            <h3 class="card-text">
              <b>{{ giangvienCount }}</b>
            </h3>
          </div>
        </div>
      </div>
      <div class="col-md-4 mb-3 col-sm-6">
        <div class="card" style="background-color: rgba(0, 128, 0, 0.5)">
          <div class="card-body">
            <h4 class="card-title">Tiết Học</h4>
            <h3 class="card-text">
              <b>{{ tongSoTiet }}</b>
            </h3>
          </div>
        </div>
      </div>
      <div class="col-md-4 mb-3 col-sm-6">
        <div class="card" style="background-color: rgba(220, 53, 69, 0.5)">
          <div class="card-body">
            <h4 class="card-title">Ngày Vắng</h4>
            <h3 class="card-text">
              <b>{{ tongNgayNghi }}</b>
            </h3>
          </div>
        </div>
      </div>
      <div class="col-md-4 mb-3 col-sm-6">
        <div class="card" style="background-color: rgba(128, 0, 128, 0.5)">
          <div class="card-body">
            <h4 class="card-title">Cấm Thi</h4>
            <h3 class="card-text">
              <b>{{ slcamthi }}</b>
            </h3>
          </div>
        </div>
      </div>
    </div>
    <hr />
    <div class="row">
      <div class="col-md-4">
        <div class="form-group">
          <label for="">Chọn Lớp:</label>
          <select class="form-control" v-model="selectedClass">
            <option value="">Chọn lớp</option>
            <option
              v-for="classItem in lop"
              :key="classItem.id"
              :value="classItem.id"
            >
              {{ classItem.tenlop }}
            </option>
          </select>
        </div>
      </div>
      <div class="col-md-4">
        <div class="form-group">
          <label for="">Chọn Ngày:</label>
          <input
            type="date"
            v-model="selectedDate"
            class="form-control"
            placeholder=""
            aria-describedby="helpId"
          />
        </div>
      </div>
      <div class="col-md-4">
        <br />
        <button type="button" class="btn btn-success" @click="fetchStudentInfo">
          Submit
        </button>
      </div>
    </div>
    <hr />
    <div class="row">
      <table class="table table-striped table-inverse table-responsive">
        <thead class="thead-inverse">
          <tr>
            <th>Stt</th>
            <th>Msvv</th>
            <th>Họ tên</th>
            <th>Điểm danh</th>
            <th>Số ngày vắng</th>
            <th>Tình trạng thi</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(student, index) in selectedClassStudents" :key="index">
            <td>{{ index + 1 }}</td>
            <td>{{ student.mssv }}</td>
            <td>{{ student.ten }}</td>
            <td v-if="student.diemdanh == 0" class="text-success"><b>Có</b></td>
            <td v-if="student.diemdanh == 5" class="text-danger">
              <b>Vắng</b>
            </td>
            <td>{{ student.vang / 5 }} ({{ student.vang }} tiết)</td>
            <td v-if="student.vang <= 45 * 0.3" class="text-success">>
              <b>Có thể thi</b>
            </td>
            <td v-else class="text-danger"><b>Cấm thi</b></td>
            <!-- <td>{{ student.vang >= 45 * 0.3 ? "Cấm thi" : "Có thể thi" }}</td> -->
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
<script>
import {
  getFirestore,
  onSnapshot,
  collection,
  query,
  where,
  //   doc,
  //   getDoc,
  getDocs, 
} from "firebase/firestore";
import { initializeApp } from "firebase/app";

import { onUnmounted, ref } from "vue";
const firebaseConfig = {
 apiKey: "AIzaSyCk0TFvgdSampCjkzPuEFoUOco-qUu5JoY",
  authDomain: "nhatu-51c28.firebaseapp.com",
  projectId: "nhatu-51c28",
  storageBucket: "nhatu-51c28.appspot.com",
  messagingSenderId: "558381258516",
  appId: "1:558381258516:web:e9ee68b3fd62dd54471401",
  measurementId: "G-BVBX4GQ956"
};
const app = initializeApp(firebaseConfig);
// const analytics = getAnalytics(app);
// const db = getFirestore(app);
const db = getFirestore(app);
export default {
  data: () => {
    return {
      test: 0,
      lop: ref([]),
      sinhvienCount: 0,
      giangvienCount: 0,
      tongSoTiet: 0,
      tongNgayNghi: 0,
      slcamthi: 0,
      search: "",
      selectedClass: "",
      selectedDate: "",
      selectedClassStudents: [],
      selectedDateIndex: null,
    };
  },
  mounted() {
    const classQuery = query(collection(db, "lop"));
    const classSnapshot = onSnapshot(classQuery, (snapshot) => {
      this.lop = snapshot.docs.map((doc) => ({
        id: doc.id,
        tenlop: doc.data().tenlop,
      }));
    });
    onUnmounted(classSnapshot);

    this.fetchStudentInfo();
    //sl sinh vien
    const slsv = query(collection(db, "sinhvien"));
    const slsinhvien = onSnapshot(slsv, (snapshot) => {
      this.sinhvienCount = snapshot.docs.length; 
    });
    onUnmounted(slsinhvien);
    //sl giang vien
    const slgv = query(collection(db, "lop"));
    const slgiangvien = onSnapshot(slgv, (snapshot) => {
      this.giangvienCount = snapshot.docs.length; 
    });
    onUnmounted(slgiangvien);

    // sl tiết
    const sltongtiet = query(collection(db, "lop"));
    const slltongtiet = onSnapshot(sltongtiet, (snapshot) => {
      this.tongSoTiet = snapshot.docs.reduce((total, doc) => {
        return total + doc.data().sotiet; 
      }, 0);
    });
    onUnmounted(slltongtiet);

    // số nagyf vắng
    const tongngaynghi = query(collection(db, "ketqua"));
    const ngaynghi = onSnapshot(tongngaynghi, (snapshot) => {
      this.tongNgayNghi = snapshot.docs.reduce((total, doc) => {
        return total + doc.data().tongvang / 5;
      }, 0);
    });
    onUnmounted(ngaynghi);
    // sl cấm thi
    const camthiQuery = query(
      collection(db, "ketqua"),
      where("tongvang", ">", 45 * 0.3)
    );
    const camthiSnapshot = onSnapshot(camthiQuery, (snapshot) => {
      this.slcamthi = snapshot.docs.length;
    });
    onUnmounted(camthiSnapshot);
    //
    const latestQuery = query(collection(db, "lop"));

    const livemessages = onSnapshot(latestQuery, (snapshot) => {
      this.lop = snapshot.docs.map((doc) => {
        return {
          id: doc.id,
          giaovien: doc.data().giaovien,
          tenlop: doc.data().tenlop,
          soluong: doc.data().soluong,
          sotiet: doc.data().sotiet,
        };
      });
    });
    onUnmounted(livemessages);
  },
  methods: {
    async fetchStudentInfo() {
      if (!this.selectedClass || !this.selectedDate) return;

      const dateString = this.selectedDate.split("-").reverse().join("-");

      const studentQuery = query(
        collection(db, "ketqua"),
        where("idlop", "==", this.selectedClass),
        where("ngay", "==", dateString)
      );
      const studentSnapshot = await getDocs(studentQuery);
      this.selectedClassStudents = studentSnapshot.docs.map((doc) => {
        return {
          mssv: doc.data().mssv, 
          ten: doc.data().ten,
          vang: doc.data().tongvang,
          diemdanh: doc.data().diemdanh,
        //   tinhtrang: doc.data().tongvang >= 45 * 0.3 ? "Cấm thi" : "Có thể thi",
        };
      });
    },
  },
};
</script>

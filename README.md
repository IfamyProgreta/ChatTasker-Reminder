
## 📂 **1. Nama Modul: ChatTasker-Reminder**

- **Fungsi Utama**: Mengirimkan notifikasi pengingat kepada pengguna untuk tugas yang mendekati tenggat waktu, agar pengguna dapat menyelesaikan tugas tepat waktu.
- **Teknologi**:
  - *Firebase*: Firebase Cloud Functions (untuk pengingat berbasis cloud)
  - *Alternatif*: Node-cron (untuk penjadwalan dengan cron jobs jika di-host secara lokal)
- **Repositori GitHub**: `ChatTasker-Reminder`

---

## 📘 **2. Deskripsi Modul**

Modul Pengelola Pengingat dirancang untuk mengotomatisasi proses pengiriman notifikasi terkait tenggat waktu tugas. Dengan modul ini, pengguna akan mendapatkan pengingat melalui email atau push notification ketika sebuah tugas mendekati waktu jatuh tempo. Modul ini mendukung pengingat otomatis ketika tugas dibuat atau ketika tenggat waktu tugas diubah.

---

## 🚧 **3. Cara Kerja Modul**

### Alur Kerja

1. **Tugas Baru atau Update Tugas**:
   - Setiap kali tugas baru dibuat atau tanggal jatuh tempo diubah, sistem akan memperbarui jadwal pengingat.
2. **Penjadwalan Pengingat**:
   - Penjadwalan dilakukan berdasarkan tenggat waktu tugas (`dueDate`), dengan mengatur pengingat beberapa waktu sebelumnya (misalnya, 24 jam sebelum tenggat).
3. **Kirim Notifikasi Pengingat**:
   - Sistem mengirimkan notifikasi (email atau push notification) kepada pengguna sebagai pengingat tugas yang akan jatuh tempo.

### Diagram Alur

1. **Tugas Baru atau Update Tugas** ➔ **Database Update** ➔ **Penjadwalan Pengingat** ➔ **Pengingat Dikirim**

---

## 🛠️ **4. Tugas dan Fungsi Utama dalam Pengembangan**

### A. **Setup Proyek**

- **Tugas**: Mengatur struktur proyek untuk Firebase Cloud Functions atau cron jobs dengan Node.js.
- **Langkah**:
  1. Inisialisasi proyek dengan Firebase atau Node-cron, bergantung pada kebutuhan deployment.
  2. **Instalasi Dependensi**:
     ```bash
     npm init -y
     npm install firebase-admin firebase-functions node-cron nodemailer dotenv
     ```

### B. **Menyiapkan Firebase Cloud Functions atau Node-cron untuk Penjadwalan**

- **Tugas**: Menyusun fungsi cloud atau cron job untuk menjalankan pengingat secara berkala.
- **Langkah**:

  1. *Jika menggunakan Firebase Cloud Functions*, buat fungsi cloud yang akan berjalan setiap hari atau sesuai penjadwalan.
  2. *Jika menggunakan Node-cron*, setup cron job untuk menjalankan pengingat secara berkala.

  - *Contoh Firebase Cloud Function*:
    ```javascript
    const functions = require('firebase-functions');
    const admin = require('firebase-admin');
    admin.initializeApp();

    exports.sendReminder = functions.pubsub.schedule('every 24 hours').onRun(async () => {
        // Logic to check for tasks with upcoming due dates and send reminders
    });
    ```

### C. **Membangun Fungsi Notifikasi Pengingat**

- **Tugas**: Membuat fungsi yang mengirimkan pengingat melalui email atau push notification.
- **Langkah**:

  1. Setup koneksi email (gunakan Nodemailer untuk email atau Firebase Cloud Messaging untuk push notification).
  2. Buat fungsi `sendNotification` yang menerima data tugas dan informasi pengguna untuk mengirim notifikasi.

  - *Contoh Kode*:
    ```javascript
    const nodemailer = require('nodemailer');

    const sendNotification = async (userEmail, taskName, dueDate) => {
        const transporter = nodemailer.createTransport({
            service: 'gmail',
            auth: {
                user: process.env.EMAIL,
                pass: process.env.PASSWORD,
            },
        });

        const mailOptions = {
            from: process.env.EMAIL,
            to: userEmail,
            subject: `Reminder: ${taskName} is due soon!`,
            text: `Your task "${taskName}" is due on ${dueDate}. Please complete it on time.`,
        };

        await transporter.sendMail(mailOptions);
    };
    ```

### D. **Sinkronisasi dengan Database untuk Pengingat Otomatis**

- **Tugas**: Mengakses Firebase untuk mendapatkan daftar tugas yang mendekati tenggat waktu.
- **Langkah**:

  1. Ambil data tugas dari Firebase yang memiliki tenggat waktu mendekati (misalnya, dalam 24 jam).
  2. Filter tugas berdasarkan `dueDate` untuk memastikan hanya tugas yang hampir jatuh tempo yang dipilih.

  - *Contoh Kode*:
    ```javascript
    const db = admin.firestore();

    const getTasksNearDeadline = async () => {
        const now = new Date();
        const upcomingTasksSnapshot = await db.collection('tasks')
            .where('dueDate', '<=', new Date(now.getTime() + 24 * 60 * 60 * 1000))
            .get();

        return upcomingTasksSnapshot.docs.map(doc => doc.data());
    };
    ```

### E. **Memperbarui Jadwal Pengingat saat Tugas Baru Ditambahkan atau Diubah**

- **Tugas**: Mengatur agar pengingat di-reset atau diatur ulang setiap kali ada perubahan pada tugas.
- **Langkah**:
  1. Gunakan Cloud Firestore triggers atau observer pada perubahan `dueDate`.
  2. Saat `dueDate` diubah, fungsi `sendReminder` atau penjadwalan ulang pengingat dijalankan.

---

## 🔗 **5. Struktur Proyek**

```
ChatTasker-Reminder/
├── src/
│   ├── functions/
│   │   ├── reminderFunction.js
│   ├── utils/
│   │   ├── notificationService.js
│   ├── index.js
├── .env
├── firebase.json
├── package.json
└── README.md
```

---

## 🔑 **6. Konfigurasi dan Variabel Lingkungan**

### `.env`

```
EMAIL=<YOUR_EMAIL>
PASSWORD=<YOUR_EMAIL_PASSWORD>
FIREBASE_PROJECT_ID=<YOUR_FIREBASE_PROJECT_ID>
FIREBASE_CLIENT_EMAIL=<YOUR_FIREBASE_CLIENT_EMAIL>
FIREBASE_PRIVATE_KEY=<YOUR_FIREBASE_PRIVATE_KEY>
```

### `firebaseConfig.js`

```javascript
const admin = require('firebase-admin');
const serviceAccount = require('path/to/firebase-adminsdk.json');

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount)
});

const db = admin.firestore();
module.exports = { db };
```

---

## 📌 **7. API Endpoint dan Dokumentasi**

| HTTP Method | Endpoint                 | Description                                |
| ----------- | ------------------------ | ------------------------------------------ |
| POST        | `/send-reminder`       | Memicu pengingat manual ke pengguna        |
| GET         | `/tasks/near-deadline` | Mendapatkan daftar tugas mendekati dueDate |

---

## 🔒 **8. Keamanan dan Autentikasi**

- Gunakan autentikasi Firebase untuk pengaksesan data pengguna.
- Pastikan environment variabel untuk email dan Firebase disimpan dengan aman.

---

## 📝 **9. Panduan Pengembangan**

1. **Clone Repositori**:

   ```bash
   git clone https://github.com/yourusername/ChatTasker-Reminder.git
   cd ChatTasker-Reminder
   ```
2. **Instalasi Dependensi**:

   ```bash
   npm install
   ```
3. **Jalankan Firebase Emulator (untuk Testing)**:

   ```bash
   firebase emulators:start
   ```
4. **Deploy Firebase Cloud Function (untuk Deployment)**:

   ```bash
   firebase deploy --only functions
   ```
5. **Testing API**: Gunakan Postman atau alat testing lainnya untuk menguji webhook dan pengingat otomatis.

---

## ✅ **10. Testing dan Pengujian**

- **Unit Testing**: Buat unit test untuk `reminderFunction.js` dan `notificationService.js` menggunakan Jest atau Mocha.
- **Integration Testing**: Uji integrasi antara Firebase dan cron jobs.
- **E2E Testing**: Uji end-to-end untuk memastikan pengingat dikirim sesuai dengan tenggat waktu tugas.

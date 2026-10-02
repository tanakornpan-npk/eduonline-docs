# อ้างอิงโค้ด custom แต่ละไฟล์

ไฟล์ custom ทั้งหมดอยู่ที่ `public_html/wp-content/mu-plugins/` บนเครื่อง local (มิเรอร์กับของจริงบน production) โค้ดต้นฉบับที่ใช้แก้ไขก่อน deploy อยู่ที่ `scratch/` (ดู [05-deploy-workflow.md](05-deploy-workflow.md))

## ku-lms-extensions.php

ไฟล์หลักของระบบ custom แก้ไฟล์นี้บ่อยที่สุด โครงสร้างภายในแบ่งเป็นฟังก์ชันกลุ่มต่อไปนี้:

### กลุ่ม Curriculum / Materials
- `ku_lms_ep_course_quizzes()` — ดึงรายชื่อ quiz และหมู่เรียนของคอร์ส (ใช้ `CurriculumMaterialRepository::get_course_materials()`)
- `ku_lms_ep_material_info()`, `ku_lms_ep_my_materials()`, `ku_lms_ep_course_materials()` — ค้นหาไฟล์สื่อการสอนตามคอร์สเจ้าของ เพื่อแก้ปัญหาไฟล์จากหลายคอร์สปนกันตอนเลือกใน course builder

### กลุ่ม Quiz Schedule / Late Submission
- `ku_lms_ep_get_schedule()` / `ku_lms_ep_save_schedule()` — อ่าน/บันทึกวันเปิด-ปิดสอบและกติกาหักคะแนน
- hook บน `masterstudy_lms_user_quiz_added` — คำนวณจำนวนวันที่ส่งช้าและหักคะแนนอัตโนมัติทันทีที่นิสิตส่ง quiz

### กลุ่ม Export คะแนน (แบบสรุป — ของเดิมก่อนมี Gradebook ละเอียด)
- `ku_lms_ep_export_summary()`, `ku_lms_build_csv()`, `ku_lms_build_excel()` — export คะแนน quiz รายคอร์ส 1 แถวต่อนิสิต พร้อมจำนวนวันที่ late
- ฉีด panel เลือก quiz/หมู่เรียนเข้าไปในหน้า Gradebook เดิมของปลั๊กอิน (ผ่าน `wp_footer` + ดักปุ่ม "Export Quiz Scores" เดิม)

### กลุ่ม Row Numbering
- ฉีด CSS/JS เพิ่มคอลัมน์ลำดับในตาราง list ผู้เรียนที่หน้า Students / Manage Students / Gradebook

### กลุ่ม Detailed Gradebook (ใหม่ล่าสุด)
ดูรายละเอียดเต็มที่ [04-gradebook-feature.md](04-gradebook-feature.md) — ฟังก์ชันหลักคือ:
- `ku_lms_gradebook_build_payload()` — ประกอบข้อมูลตาราง (quiz/assignment/midterm/final) ต่อคอร์ส ใช้ร่วมกันทั้งฝั่งผู้สอนและผู้เรียน
- `ku_lms_ep_gradebook_get()`, `ku_lms_ep_gradebook_my_scores()`, `ku_lms_ep_gradebook_save_score()`, `ku_lms_ep_gradebook_get_settings()` / `_save_settings()`, `ku_lms_ep_gradebook_export()` — REST callback ทั้งหมด

## ku-change-user-email.php

ให้แอดมินเปลี่ยนอีเมลของผู้ใช้คนอื่นได้จากหน้า admin (ของเดิม WordPress ไม่เปิดให้ทำง่ายๆ)

## ku-fix-login-nonce.php

แก้ปัญหานิสิตล็อกอินไม่ได้เพราะ nonce ของฟอร์มล็อกอินหมดอายุก่อนกด submit (มักเกิดกับคนที่เปิดหน้าทิ้งไว้นาน)

## ku-student-id-field.php

เพิ่มฟิลด์ "รหัสนิสิต" ในฟอร์มโปรไฟล์ผู้ใช้ เก็บเป็น user meta `student_id` — ใช้ร่วมกับฟีเจอร์ export/gradebook เกือบทุกตัวเพื่อแสดงรหัสนิสิต

## extend-nonce-life.php

ขยายอายุ nonce มาตรฐานของ WordPress ให้นานขึ้น (ลดปัญหา session หมดอายุระหว่างทำแบบทดสอบ)

## rest-api-no-cache.php

กัน W3 Total Cache ไม่ให้แคช response ของ REST API (`/wp-json/...`) ป้องกันข้อมูลคะแนน/ความคืบหน้าค้างเป็นข้อมูลเก่า

---

## ข้อควรระวังเวลาแก้ไฟล์เหล่านี้

- **แก้ที่ `scratch/` ก่อนเสมอ** แล้วค่อย deploy ไปทับใน `wp-content/mu-plugins/` — อย่าแก้ไฟล์ปลายทางตรงๆ
- เช็ก syntax ด้วย `php -l` ทุกครั้งก่อน deploy
- ไฟล์ `ku-lms-extensions.php` ยาวขึ้นเรื่อยๆ ตาม feature ที่เพิ่ม — ถ้าเริ่มจัดการยาก ให้พิจารณาแยกเป็นหลายไฟล์ mu-plugin ตามฟีเจอร์ (WordPress โหลด mu-plugin ทุกไฟล์ในโฟลเดอร์อัตโนมัติ ไม่ต้อง activate)

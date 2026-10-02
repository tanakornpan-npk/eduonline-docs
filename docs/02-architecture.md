# สถาปัตยกรรม — ส่วนที่พัฒนาเอง

## หลักการออกแบบที่ยึดถือมาตลอด

1. **ไม่แก้ไฟล์ core ของปลั๊กอิน MasterStudy โดยตรงถ้าเลี่ยงได้** เพราะไฟล์จะถูกเขียนทับทุกครั้งที่อัปเดตปลั๊กอิน แทนที่จะแก้ตรงๆ ให้ใช้วิธี:
   - เพิ่มฟังก์ชันผ่าน **mu-plugin** (must-use plugin — ทำงานเสมอ ไม่ต้อง activate)
   - เกาะ hook/filter ที่ปลั๊กอินเปิดให้ (`add_action`, `add_filter`)
   - ถ้าจำเป็นต้องเพิ่ม UI ในหน้าที่ปลั๊กอิน Pro สร้างไว้แล้ว ให้ฉีด HTML/JS ผ่าน `wp_footer` แล้วใช้ jQuery ย้ายตำแหน่งไปจุดที่ต้องการ (ดูตัวอย่างใน [04-gradebook-feature.md](04-gradebook-feature.md))
2. **เพิ่มตาราง/คอลัมน์ฐานข้อมูลใหม่แทนการไปงัดโครงสร้างเดิม** เวลาต้องเก็บข้อมูลที่ระบบเดิมไม่มี
3. ทุก endpoint ที่พัฒนาเองอยู่ภายใต้ REST namespace เดียวกันคือ **`ku-lms/v1`**

> ข้อยกเว้นที่มีอยู่ในระบบ (จากก่อนมีแนวทางนี้): ไฟล์ `includes/init.php` ของปลั๊กอิน masterstudy-lms-learning-management-system ถูกแก้ไขตรงให้โหลดไฟล์ `includes/Hooks/manage_student_passwords.php` เพิ่ม — ถ้าปลั๊กอินนี้ถูกอัปเดตในอนาคต **ต้องเช็กว่าบรรทัดนี้ยังอยู่หรือถูกเขียนทับไปหรือไม่**

## mu-plugins ที่มีอยู่ (`wp-content/mu-plugins/`)

| ไฟล์ | หน้าที่ |
|---|---|
| `ku-lms-extensions.php` | ไฟล์หลัก รวมฟีเจอร์ custom เกือบทั้งหมด (ดูรายละเอียดด้านล่าง) |
| `ku-change-user-email.php` | ให้แอดมิน/ผู้ใช้เปลี่ยนอีเมลผู้ใช้ได้ |
| `ku-fix-login-nonce.php` | แก้ปัญหา nonce หมดอายุตอนล็อกอิน |
| `ku-student-id-field.php` | เพิ่มฟิลด์ "รหัสนิสิต" (`student_id` user meta) ในโปรไฟล์ |
| `extend-nonce-life.php` | ขยายอายุ nonce ของ WordPress |
| `rest-api-no-cache.php` | กัน W3TC ไม่ให้ cache REST API response |

## `ku-lms-extensions.php` — แผนผังฟีเจอร์

ไฟล์นี้คือจุดรวมของส่วนขยายเกือบทั้งหมด แบ่งเป็นหัวข้อ (section) ตามคอมเมนต์ในไฟล์:

1. **DB Migration** — เพิ่มคอลัมน์/ตารางที่ยังไม่มีตอน `init` (idempotent — เช็กก่อนสร้างทุกครั้ง)
2. **REST Endpoints** — ลงทะเบียน route ทั้งหมดใน namespace `ku-lms/v1`
3. **Late Submission Hook** — หักคะแนน quiz อัตโนมัติเมื่อนิสิตส่งช้ากว่ากำหนด (ตั้งค่าได้ต่อ quiz: จำนวนวันที่ late ได้, หักกี่คะแนน, หักแบบต่อวันหรือต่อสัปดาห์)
4. **Gradebook: Export Panel** — ปุ่ม export คะแนน quiz แบบสรุป (CSV/Excel) ในหน้า Gradebook เดิมของปลั๊กอิน
5. **Row Numbering** — เพิ่มคอลัมน์ลำดับในตาราง list ผู้เรียนหลายหน้า (ของเดิมไม่มี)
6. **Course Builder: Quiz Schedule + Materials Filter** — ตั้งวันเปิด-ปิดสอบของ quiz และกรองไฟล์สื่อการสอนตามคอร์สตอนเลือกจาก material library
7. **Detailed Gradebook** — ตารางคะแนนละเอียดแบบเต็มจอ (Quiz/Assignment/Midterm/Final) ของผู้สอน + หน้าดูคะแนนของผู้เรียนเอง (ดู [04-gradebook-feature.md](04-gradebook-feature.md))

## REST API ที่พัฒนาเอง (`ku-lms/v1`)

| Method | Route | ใช้ทำอะไร | สิทธิ์ |
|---|---|---|---|
| GET | `/course/{course_id}/quizzes` | รายชื่อ quiz + หมู่เรียนในคอร์ส | ล็อกอินแล้ว |
| GET/PUT | `/quizzes/{quiz_id}/schedule` | อ่าน/บันทึกวันเปิด-ปิดสอบ + กติกาหักคะแนน late | ล็อกอิน / เจ้าของ quiz |
| GET | `/students/export-summary/{course_id}` | export คะแนน quiz แบบสรุป (CSV/Excel) | เจ้าของคอร์ส/ผู้สอนร่วม |
| GET | `/my-courses` | คอร์สที่ผู้สอนคนปัจจุบันเป็นเจ้าของ | ล็อกอินแล้ว |
| GET | `/my-enrolled-courses` | คอร์สที่ผู้เรียนคนปัจจุบันลงทะเบียนอยู่ | ล็อกอินแล้ว |
| GET | `/materials/info`, `/my-materials`, `/course-materials` | ค้นหา/กรองไฟล์สื่อการสอนตามคอร์สเจ้าของ | ล็อกอินแล้ว |
| GET | `/gradebook/{course_id}` | ตารางคะแนนละเอียดทั้งคอร์ส (quiz/assignment/midterm/final) | เจ้าของคอร์ส/ผู้สอนร่วม |
| GET | `/gradebook/{course_id}/my-scores` | คะแนนของผู้เรียนคนปัจจุบันเอง | ผู้เรียนที่ลงทะเบียนคอร์สนั้น |
| POST | `/gradebook/{course_id}/score` | บันทึกคะแนน assignment/midterm/final ทีละช่อง | เจ้าของคอร์ส/ผู้สอนร่วม (ตามหมู่เรียนที่ได้รับสิทธิ์) |
| GET/POST | `/gradebook/{course_id}/settings` | อ่าน/ตั้งคะแนนเต็มของ midterm/final ต่อคอร์ส | เจ้าของคอร์ส/ผู้สอนร่วม |
| GET | `/gradebook/{course_id}/export` | ส่งออกตารางคะแนนละเอียดเป็นไฟล์ Excel | เจ้าของคอร์ส/ผู้สอนร่วม |

การตรวจสิทธิ์ทั้งหมดใช้ `STM_LMS_Course::check_course_author()` (เจ้าของคอร์สหรือ co-instructor) ร่วมกับ `StudentsRepository::get_allowed_sections()` (จำกัดผู้สอนร่วมที่ถูกกำหนดหมู่เรียนให้เห็น/แก้ได้เฉพาะหมู่เรียนของตัวเอง) — ใช้ฟังก์ชันเดิมของปลั๊กอิน ไม่ได้เขียนระบบสิทธิ์ใหม่

## ตารางฐานข้อมูลที่เพิ่มเอง

| ตาราง | หน้าที่ |
|---|---|
| `wp_ku_lms_gradebook_scores` | เก็บคะแนน midterm/final ของแต่ละนิสิตในแต่ละคอร์ส (ไม่มีที่เก็บในระบบเดิม) |

## คอลัมน์/post meta ที่เพิ่มเอง

| ชื่อ | อยู่ที่ | ใช้ทำอะไร |
|---|---|---|
| `late_days` | คอลัมน์ใหม่ในตาราง `stm_lms_user_quizzes` | จำนวนวันที่นิสิตส่ง quiz ช้า |
| `ku_exam_start_date`, `ku_exam_end_date` | post meta ของ quiz | ช่วงวันเปิด-ปิดสอบ |
| `ku_late_days_allowed`, `ku_late_deduction`, `ku_late_deduction_type` | post meta ของ quiz | กติกาหักคะแนนกรณีส่งช้า |
| `ku_midterm_max_point`, `ku_final_max_point` | post meta ของคอร์ส | คะแนนเต็มของ midterm/final ต่อคอร์ส |
| `student_id` | user meta | รหัสนิสิต |
| `stm_lms_section` | user meta | หมู่เรียนของนิสิต |

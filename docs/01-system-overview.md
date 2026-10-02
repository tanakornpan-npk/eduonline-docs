# ภาพรวมระบบ

## ระบบคืออะไร

eduonline เป็นเว็บ e-Learning (LMS) ของหน่วยงาน สร้างบน **WordPress** โดยใช้ปลั๊กอิน **MasterStudy LMS Learning Management System** (เวอร์ชัน free + Pro ติดตั้งคู่กัน) เป็นแกนหลักของระบบคอร์สเรียน แบบทดสอบ (quiz) งานที่มอบหมาย (assignment) และการให้คะแนน

## เทคโนโลยีหลัก

| ส่วน | ใช้อะไร |
|---|---|
| CMS | WordPress |
| LMS core | masterstudy-lms-learning-management-system (free) |
| LMS ส่วนขยาย | masterstudy-lms-learning-management-system-pro |
| ธีม | masterstudy |
| E-commerce / สมัครสมาชิก | WooCommerce, Paid Memberships Pro |
| Page builder | Elementor, js_composer |
| Cache | W3 Total Cache (page cache + object cache + db cache) |
| Security | All In One WP Security |
| Web/App server | Apache (httpd) + PHP-FPM |
| Database | MySQL (ผ่าน `$wpdb` ของ WordPress) |

## โครงสร้างไฟล์บนเครื่อง local (dev)

```
D:\wamp64\eduonline\
├── CLAUDE.md          # กติกาการทำงานกับ Claude Code ในโปรเจกต์นี้ (ต้องสื่อสารภาษาไทย, อ่าน deploy.md ก่อน deploy)
├── deploy.md          # ขั้นตอน + credential สำหรับ deploy (LOCAL ONLY — ห้ามขึ้น git)
├── public_html/       # มิเรอร์ของเว็บไซต์ WordPress (โค้ดเดียวกับ production)
│   └── wp-content/
│       ├── mu-plugins/   # ปลั๊กอิน custom ที่พัฒนาเอง (always-on, แก้ไม่ได้ผ่านหน้า admin)
│       ├── plugins/      # ปลั๊กอินทั่วไปรวมถึง MasterStudy LMS + Pro
│       └── themes/       # ธีม masterstudy
└── scratch/           # สคริปต์ diagnostic/debug แบบ one-off (check_*.php) ใช้ไล่ปัญหาเฉพาะหน้า
```

## Production

| | |
|---|---|
| โดเมน | `eduonline.csc.ku.ac.th` |
| Web root บน server | `/home/eduonline/public_html/` |
| ผู้ใช้ deploy | `oastkk` (ผ่าน SSH, ใช้ sudo คัดลอกไฟล์เข้า web root เพราะเจ้าของไฟล์จริงคือ user `eduonline`) |
| วิธี deploy | `pscp` (อัปโหลด) + `plink` (รันคำสั่งระยะไกล) จาก PuTTY เท่านั้น — **ห้ามใช้ PHP deploy script หรือ php.bat** |

ดูขั้นตอนแบบเต็มที่ [05-deploy-workflow.md](05-deploy-workflow.md)

## แนวคิดสำคัญของ MasterStudy LMS ที่ต้องรู้ก่อนแก้โค้ด

- **Course (คอร์ส)** = custom post type `stm-courses` แต่ละคอร์สมี curriculum เป็น section → materials (lesson / quiz / assignment)
- **หมู่เรียน (section)** ของนิสิต — ไม่ใช่ curriculum section ของคอร์ส แต่เป็น user meta `stm_lms_section` เก็บไว้ที่ตัวผู้ใช้แต่ละคน ใช้แยกกลุ่มนิสิตในคอร์สเดียวกัน
- **Co-instructor** = ผู้สอนร่วม เก็บใน post meta `co_instructor` ของคอร์ส และสามารถจำกัดให้เห็น/แก้ไขได้เฉพาะบางหมู่เรียนผ่าน post meta `_co_instructor_sections_{user_id}`
- **ตารางฐานข้อมูลหลักที่เกี่ยวกับความคืบหน้า/คะแนน** (prefix ตาม `$wpdb->prefix`):
  - `stm_lms_user_courses` — การลงทะเบียนเรียนของผู้ใช้แต่ละคอร์ส (progress_percent, final_grade, start_time)
  - `stm_lms_user_quizzes` — ผลการทำ quiz แต่ละครั้ง (progress = คะแนน %, status = passed/failed)
  - `stm_lms_user_assignments` — คะแนนงานที่ส่ง (grade, status)
  - `stm_lms_curriculum_materials` / `stm_lms_curriculum_sections` — โครงสร้าง curriculum ของแต่ละคอร์ส

รายละเอียดเพิ่มเติมของส่วนที่พัฒนาเองอยู่ใน [02-architecture.md](02-architecture.md)

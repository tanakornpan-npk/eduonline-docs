# ปัญหาที่ทราบอยู่แล้ว (ยังไม่ได้แก้)

## 1. Fatal error ที่หน้า course player เมื่อเข้าโดยไม่มี course_id

**อาการ**

```
Uncaught TypeError: MasterStudy\Lms\Repositories\CurriculumRepository::get_curriculum():
Argument #1 ($course_id) must be of type int, null given
```

เกิดที่ `CoursePlayerRepository.php:37` → `CurriculumRepository.php:38` เวลามีคนเข้าหน้า course player (`course-player.php`) โดยที่ query ไม่มีค่า course_id ที่ถูกต้อง (มักเกิดจากบอทหรือลิงก์เก่าที่ชี้ URL ไม่ครบ)

**สถานะ:** เกิดขึ้นเป็นระยะมานานแล้ว (พบ log ย้อนหลังอย่างน้อยตั้งแต่เดือนก่อน) ความถี่อยู่ที่ประมาณ 5–10 ครั้ง/วัน ไม่กระทบผู้ใช้จริงที่เข้าผ่านลิงก์ปกติ จึงยังไม่ได้แก้ แต่**เป็น bug ของปลั๊กอิน MasterStudy LMS เอง ไม่ใช่โค้ด custom**

**แนวทางแก้ที่เป็นไปได้ (ยังไม่ได้ทำ):** เกาะ `pre_get_posts` หรือ `template_redirect` เช็ก course_id ก่อนให้ template โหลด ถ้าไม่ valid ให้ redirect ไปหน้าอื่นแทนปล่อยให้ fatal error

**วิธีเช็กว่ายังเกิดอยู่ไหม:**
```
plink -batch -pw <SSH_PASSWORD> -P 22 oastkk@<SERVER_IP> \
  "grep -c 'CurriculumRepository::get_curriculum' /home/eduonline/public_html/wp-content/uploads/wc-logs/fatal-errors-$(date +%Y-%m-%d)-*.log"
```

## 2. คะแนน assignment ที่กรอกผ่านตารางคะแนนละเอียด อาจไม่ตรงกับสถิติ "ส่งงานแล้ว" ของปลั๊กอิน

ดู [04-gradebook-feature.md](04-gradebook-feature.md) หัวข้อที่เก็บข้อมูล — ถ้านิสิตไม่เคยส่งงานแต่ผู้สอนกรอกคะแนนเองผ่านตารางละเอียด ระบบจะสร้างแถวใน `wp_stm_lms_user_assignments` ให้โดยเว้นค่า `status` ว่างไว้ (ค่า default ของตาราง) เพราะยังไม่เคยตรวจสอบว่า enum status เต็มของ Assignments addon (Pro) คืออะไรบ้าง ถ้าหน้าอื่นของปลั๊กอิน (เช่น สถิติ "จำนวนคนส่งงานแล้ว") นับจาก status แบบเฉพาะเจาะจง ตัวเลขอาจไม่ตรงกับความเป็นจริง 100%

**แนวทางแก้:** หา enum ของ `status` ในโมดูล `MasterStudy\Lms\Pro\addons\assignments` แล้วอัปเดตให้ตรง

## วิธีเพิ่มรายการปัญหาใหม่

พบ bug หรือพฤติกรรมแปลกๆ ที่ยังไม่ได้แก้ ให้เพิ่มหัวข้อใหม่ในไฟล์นี้ตามรูปแบบเดิม (อาการ / สถานะ / แนวทางแก้ / วิธีเช็ก) จะได้ไม่ต้องไล่หาซ้ำทุกครั้งที่เจอ

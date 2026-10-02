# eduonline — คู่มือระบบและ Workflow

คู่มืออ้างอิงสำหรับระบบ e-Learning ของ **eduonline.csc.ku.ac.th** สร้างจาก WordPress + MasterStudy LMS (Pro) พร้อมส่วนขยายที่พัฒนาเองเฉพาะหน่วยงาน

> **สำคัญ:** รีโปนี้เก็บ "คู่มือ" เท่านั้น ไม่ใช่โค้ดจริงของเว็บไซต์ และ **ห้าม commit credential ใดๆ** (รหัสผ่าน SSH, sudo, database ฯลฯ) ลงรีโปนี้เด็ดขาด ข้อมูล credential เก็บแยกไว้ในเครื่อง local เท่านั้น (ดู [05-deploy-workflow.md](docs/05-deploy-workflow.md) หัวข้อ "เรื่อง credential")

## สารบัญ

| เอกสาร | เนื้อหา |
|---|---|
| [01-system-overview.md](docs/01-system-overview.md) | ภาพรวมระบบ, เทคโนโลยีที่ใช้, โครงสร้างโฮสติ้ง |
| [02-architecture.md](docs/02-architecture.md) | สถาปัตยกรรม, mu-plugins ที่พัฒนาเอง, REST API ที่เพิ่มเข้ามา |
| [03-custom-code-reference.md](docs/03-custom-code-reference.md) | รายละเอียดโค้ด custom แต่ละไฟล์ ทำหน้าที่อะไรบ้าง |
| [04-gradebook-feature.md](docs/04-gradebook-feature.md) | Deep dive ฟีเจอร์ตารางคะแนนละเอียด (Quiz/Assignment/Midterm/Final) |
| [05-deploy-workflow.md](docs/05-deploy-workflow.md) | ขั้นตอน deploy ขึ้น production แบบปลอดภัย |
| [06-known-issues.md](docs/06-known-issues.md) | ปัญหาที่ทราบอยู่แล้วในระบบ (ยังไม่ได้แก้) |

## ข้อมูลระบบโดยสรุป

| | |
|---|---|
| Production URL | https://eduonline.csc.ku.ac.th |
| CMS / LMS | WordPress + MasterStudy LMS Learning Management System (Pro) |
| Theme | masterstudy |
| Web server | Apache (httpd) + PHP-FPM |
| Cache | W3 Total Cache |
| Local dev path | `D:\wamp64\eduonline\public_html` |

## เริ่มต้นอ่านยังไงดี

- อยากรู้ภาพรวมระบบทั้งหมด → เริ่มที่ [01-system-overview.md](docs/01-system-overview.md)
- จะแก้ไข/เพิ่มฟีเจอร์ custom → อ่าน [02-architecture.md](docs/02-architecture.md) และ [03-custom-code-reference.md](docs/03-custom-code-reference.md) ก่อนเสมอ
- จะ deploy ขึ้น production → อ่าน [05-deploy-workflow.md](docs/05-deploy-workflow.md) ทุกครั้งก่อน deploy
- เจออาการประหลาดบนเว็บ ลองดู [06-known-issues.md](docs/06-known-issues.md) ก่อนว่าเคยเจอมาก่อนหรือยัง

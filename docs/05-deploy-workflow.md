# Workflow การ Deploy

## เรื่อง credential

**ไฟล์นี้ไม่มี credential ใดๆ โดยตั้งใจ** รหัสผ่าน SSH/sudo ของ production เก็บอยู่ที่ไฟล์ `D:\wamp64\eduonline\deploy.md` บนเครื่อง local เท่านั้น ไฟล์นั้น**ห้าม commit ขึ้น git หรือที่สาธารณะใดๆ เด็ดขาด** ถ้าต้องแชร์ให้คนในทีมใช้ช่องทางส่วนตัว (เช่น password manager) ไม่ใช่ repo นี้

## เครื่องมือที่ใช้

Deploy ผ่าน **PuTTY เท่านั้น**: `pscp` (อัปโหลดไฟล์) + `plink` (รันคำสั่ง SSH ระยะไกล) — **ห้ามใช้ PHP deploy script หรือ php.bat** เพราะเสี่ยงต่อการรันโค้ดไม่ครบ/ติด timeout และตรวจสอบ error ยากกว่า

## ภาพรวม pipeline

```
[แก้โค้ดใน scratch/] → [เช็ก syntax php -l local] → [สำรองไฟล์เดิมบน production]
   → [pscp อัปโหลดไป deploy_temp] → [php -l บน production] → [sudo cp เข้า web root + chmod 644]
   → [ล้าง W3TC cache] → [ตรวจสอบว่าเว็บยังทำงานปกติ + ไม่มี error ใหม่]
```

## ขั้นตอนละเอียด

### 1. แก้โค้ดและเช็ก syntax บนเครื่อง local ก่อนเสมอ

แก้ไฟล์ใน `scratch/` (ไม่ใช่แก้ตรงใน `public_html/wp-content/mu-plugins/`) แล้วรัน:

```
php -l "D:\wamp64\eduonline\scratch\ku-lms-extensions.php"
```

### 2. สำรองไฟล์เดิมบน production ก่อนเขียนทับทุกครั้ง

เพราะโฟลเดอร์ `wp-content/mu-plugins/` เป็นของ user `eduonline` ไม่ใช่ `oastkk` ที่ใช้ SSH เข้าไป การ backup และการย้ายไฟล์เข้า web root **ต้องใช้ sudo**

```
plink -batch -pw <SSH_PASSWORD> -P 22 oastkk@<SERVER_IP> ^
  "echo '<SUDO_PASSWORD>' | sudo -S cp /home/eduonline/public_html/wp-content/mu-plugins/ku-lms-extensions.php ^
   /home/eduonline/public_html/wp-content/mu-plugins/ku-lms-extensions.php.bak_$(date +%Y%m%d%H%M%S) && echo backup_ok"
```

### 3. อัปโหลดไฟล์ใหม่ไปที่ staging directory ก่อน

```
pscp -batch -pw <SSH_PASSWORD> -P 22 LOCAL_FILE oastkk@<SERVER_IP>:/home/oastkk/deploy_temp/FILENAME
```

### 4. เช็ก syntax อีกรอบบน production (เผื่อ environment ต่างจาก local)

```
plink -batch -pw <SSH_PASSWORD> -P 22 oastkk@<SERVER_IP> "php -l /home/oastkk/deploy_temp/FILENAME"
```

ถ้ามี syntax error **ห้ามไปขั้นตอนถัดไป** — มิฉะนั้นเว็บทั้งเว็บจะล่มทันทีเพราะ mu-plugin ทำงานทุก request

### 5. ย้ายเข้า web root จริงด้วย sudo

```
plink -batch -pw <SSH_PASSWORD> -P 22 oastkk@<SERVER_IP> ^
  "echo '<SUDO_PASSWORD>' | sudo -S cp /home/oastkk/deploy_temp/FILENAME /home/eduonline/public_html/wp-content/mu-plugins/FILENAME ^
   && echo '<SUDO_PASSWORD>' | sudo -S chmod 644 /home/eduonline/public_html/wp-content/mu-plugins/FILENAME && echo deploy_ok"
```

### 6. ล้าง cache

ระบบใช้ W3 Total Cache มีสคริปต์ช่วยล้าง cache อยู่แล้วที่ `/home/oastkk/flush_w3tc.php` บน production:

```
plink -batch -pw <SSH_PASSWORD> -P 22 oastkk@<SERVER_IP> "cd /home/oastkk && php flush_w3tc.php"
```

### 7. ตรวจสอบหลัง deploy เสมอ — อย่า deploy แล้วจบเลย

- `curl -s -o /dev/null -w '%{http_code}' https://eduonline.csc.ku.ac.th/` → ต้องได้ 200
- เช็ก fatal error log ล่าสุด เทียบเวลาก่อน/หลัง deploy:
  `/home/eduonline/public_html/wp-content/uploads/wc-logs/fatal-errors-YYYY-MM-DD-*.log`
- ถ้าเพิ่ม REST endpoint ใหม่ ให้เขียนสคริปต์เช็กสั้นๆ (require `wp-load.php` แล้วเรียก `rest_get_server()->get_routes()`) ยืนยันว่า route ขึ้นจริง — ดูตัวอย่างแนวทางได้จากสคริปต์ `check_*.php` ใน `scratch/`

### 8. ถ้าพัง — rollback

ไฟล์สำรองจากขั้นตอนที่ 2 อยู่ที่เดิมในชื่อ `FILENAME.bak_YYYYMMDDHHMMSS` สั่ง `sudo cp` กลับได้ทันที (ขั้นตอนเดียวกับข้อ 5 แต่สลับทิศทางไฟล์)

## หมายเหตุเรื่อง timezone เวลาเทียบ log

Log ของ wc-logs ใช้เวลา **UTC** (`+00:00`) แต่ timestamp ไฟล์จาก `ls -la` / `date` ของ shell เป็นเวลาท้องถิ่น (`+07:00`, กรุงเทพ) เวลาเทียบว่า error เกิดก่อน/หลัง deploy ต้องแปลง timezone ให้ตรงกันก่อนเทียบ ไม่งั้นจะเข้าใจผิดว่ามี error ใหม่ทั้งที่จริงๆ เป็น error เก่า

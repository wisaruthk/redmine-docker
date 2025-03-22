# Introduction
Redmine เป็นเครื่องมือ Project Management ที่เหมาะสำหรับผู้เริ่มต้นและสามารถปรับแต่งได้หลายรูปแบบ ที่ข้าพเจ้าได้มีประสบการณ์ใช้งานมา โดยมีฟีเจอร์หลักที่น่าสนใจ ใช้งานได้จริง ดังนี้ :

✅ ข้อดีของ Redmine
ใช้งานง่าย – โครงสร้างชัดเจน มีระบบ Issue Tracking และ Task Management

ปรับแต่งได้ – รองรับการเพิ่มช่องกรอกข้อมูล (Custom Fields) ตามต้องการ

รองรับการ Export ข้อมูล – สามารถส่งออกเป็น CSV, PDF และออกรายงานได้

ระบบจัดการผู้ใช้และสิทธิ์การเข้าถึง – สามารถกำหนดสิทธิ์ของผู้ใช้ได้อย่างละเอียด

รองรับหลายโครงการ (Multi-Project) – สามารถบริหารหลายโปรเจคได้ในระบบเดียว

Open Source และฟรี – ไม่มีค่าใช้จ่าย สามารถติดตั้งและใช้งานได้เอง

กระผมได้มีโอกาสในการนำมาใช้งานจริงซึ่งสามารถตอบโจทย์ของการบริหารทีมได้ค่อนข้างดีทีเดียว เพราะติดตั้งบนเครื่องส่วนตัวหรือ server ส่วนตัวได้

สำหรับ Repo นี้สำหรับนำไปใช้บน Docker ได้อย่างสะดวกๆครับ

---

# สำหรับการเข้าใช้เมื่อติดตั้งครั้งแรก
1. เมื่อติดตั้งเสร็จเข้าระบบด้วย url: http://localhost:8090/
2. **Username/Password:** `admin/admin`
3. ต้องเข้าไปกด initial default Role ในหน้าจอ Admin นะครับ

---

# การติดตั้ง Plugin

## Installing a plugin
1. คัดลอกโฟลเดอร์ปลั๊กอินของคุณไปที่ `{RAILS_ROOT}/plugins` หากคุณดาวน์โหลดปลั๊กอินจาก GitHub โดยตรง ให้เข้าไปที่โฟลเดอร์ปลั๊กอินของคุณและใช้คำสั่งนี้:

   ```sh
   git clone git://github.com/user_name/name_of_the_plugin.git
   ```

2. หากปลั๊กอินต้องการ migration ให้รันคำสั่งต่อไปนี้ที่ `{RAILS_ROOT}` เพื่ออัปเกรดฐานข้อมูล (ควรทำการสำรองข้อมูลฐานข้อมูลก่อน):

   ```sh
   bundle exec rake redmine:plugins:migrate RAILS_ENV=production
   ```

3. รีสตาร์ท Redmine

   คุณสามารถดูรายการปลั๊กอินที่ติดตั้งได้ที่ **Administration -> Plugins** และกำหนดค่าปลั๊กอินที่ติดตั้งใหม่ (หากจำเป็นต้องตั้งค่าเพิ่มเติม)

## Uninstalling a plugin
1. หากปลั๊กอินมี migration ให้รันคำสั่งนี้เพื่อดาวน์เกรดฐานข้อมูล (ควรทำการสำรองข้อมูลฐานข้อมูลก่อน):

   ```sh
   bundle exec rake redmine:plugins:migrate NAME=plugin_name VERSION=0 RAILS_ENV=production
   ```

2. ลบปลั๊กอินออกจากโฟลเดอร์ `{RAILS_ROOT}/plugins`
3. รีสตาร์ท Redmine

[ดูรายละเอียดเพิ่มเติมเกี่ยวกับการย้าย Redmine](https://www.redmine.org/projects/redmine/wiki/HowTo_Migrate_Redmine_to_a_new_server_to_a_new_Redmine_version)

---

# การ Backup และ Restore ฐานข้อมูลใน Docker

## Backup ฐานข้อมูล
```sh
docker exec CONTAINER /usr/bin/mysqldump -u root --password=root --skip-extended-insert DATABASE > backup.sql
```

หรือ

```sh
docker exec CONTAINER /usr/bin/mysqldump -u root --password=root --skip-extended-insert DATABASE | gzip > my_app_backup.sql.gz
```

## Restore ฐานข้อมูล
```sh
cat backup.sql | docker exec -i CONTAINER /usr/bin/mysql -u root --password=root DATABASE
```

---

# การ Backup และ Unzip ไฟล์

## Backup ไฟล์อัปโหลด
```sh
tar -czvf name-of-archive.tar.gz /path/to/directory-or-file
```

## Unzip ไฟล์
```sh
tar -xzvf archive.tar.gz
```

---

# การ Reset Password ของ MySQL ใน Docker

1. เพิ่ม `--skip-grant-tables` ใน command ของ `docker-compose`
2. หยุดคอนเทนเนอร์ฐานข้อมูล:
   ```sh
   docker-compose stop db
   ```
3. เริ่มต้นคอนเทนเนอร์ฐานข้อมูลใหม่:
   ```sh
   docker-compose up -d db
   ```
4. เข้าไปที่ MySQL container:
   ```sh
   docker-compose exec -it db bash
   ```
5. เข้า MySQL:
   ```sh
   mysql -u root
   ```
6. ใช้คำสั่งต่อไปนี้เพื่ออัปเดตรหัสผ่าน:
   ```sql
   USE mysql;
   UPDATE user SET authentication_string=PASSWORD('newpassword') WHERE User='root';
   ```
7. ตรวจสอบข้อมูล:
   ```sql
   SELECT user, Host, password_last_changed, authentication_string FROM user WHERE user='root';
   ```
8. ออกจาก MySQL และรีสตาร์ท Docker
9. ทดสอบการเข้าสู่ระบบ MySQL:
   ```sh
   docker-compose exec -it db mysql -u root -p
   ```
```


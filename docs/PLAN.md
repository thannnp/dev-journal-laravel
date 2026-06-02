# Build Plan — Learning Laravel Blog

Checklist แยกเฟส ทำจากบนลงล่าง แต่ละเฟสจบแล้วค่อยขึ้นเฟสถัดไป
นิยามศัพท์ดูที่ [../CONTEXT.md](../CONTEXT.md) · เหตุผลเชิงสถาปัตยกรรมดูที่ [adr/0001-inertia-and-rest-api-on-shared-domain.md](adr/0001-inertia-and-rest-api-on-shared-domain.md)

---

## เฟส 1 — Scaffold โครงเริ่มต้น ✅ เสร็จ (2026-06-03)

เป้าหมาย: ได้โปรเจกต์ที่ login/register ได้ และหน้าแรก Inertia + React + TS + Tailwind รันขึ้น

- [x] ติดตั้งโปรเจกต์ใหม่ด้วย React starter kit (TypeScript) — ตั้งชื่อ APP_NAME = **DevJournalLaravel**
- [x] database = **SQLite** (`DB_CONNECTION=sqlite`, ไฟล์ `database/database.sqlite`)
- [x] `composer install` และ `npm install` ครบ
- [x] `php artisan migrate` (มี 10 ตาราง: users/sessions/cache/passkeys/...)
- [x] `npm run dev` + `php artisan serve` → เปิดเว็บได้
- [x] **register → login → logout** ผ่าน UI ได้จริง (users มี 1 แถว)
- [x] commit แรก + push ขึ้น GitHub (`origin/main` → thannnp/dev-journal-laravel)

**เกณฑ์ผ่านเฟส:** ✅ สมัครสมาชิก/ล็อกอินได้ และหน้า dashboard ของ starter โผล่หลัง login

---

## เฟส 2 — Database, Models, Relationships

เป้าหมาย: schema ครบ + Eloquent relationships ทำงาน + มีข้อมูลตัวอย่างให้เล่น

### Migrations
- [ ] `posts` — `user_id` (FK→users, NOT NULL), `title`, `slug` (unique), `body` (text), `cover_image` (nullable), `published_at` (nullable timestamp), timestamps
- [ ] `tags` — `name`, `slug` (unique)
- [ ] `post_tag` (pivot) — `post_id` FK, `tag_id` FK, unique([post_id, tag_id])
- [ ] `comments` — `user_id` (FK, NOT NULL), `post_id` (FK, NOT NULL), `body` (text), timestamps
- [ ] `php artisan migrate` ผ่านไม่มี error

### Models + relationships
- [ ] `Post` — `author()` belongsTo User · `comments()` hasMany · `tags()` belongsToMany · scope `published()`
- [ ] `Comment` — `user()` belongsTo · `post()` belongsTo
- [ ] `Tag` — `posts()` belongsToMany
- [ ] `User` — `posts()` hasMany · `comments()` hasMany
- [ ] กำหนด `$fillable` (หรือ `$guarded`) ให้ทุก model
- [ ] cast `published_at` เป็น datetime ใน `Post`

### Seed ข้อมูลทดสอบ
- [ ] `PostFactory`, `TagFactory`, `CommentFactory`
- [ ] `DatabaseSeeder` — สร้าง user 2-3 คน, posts (มีทั้ง draft และ published), tags, comments
- [ ] `php artisan migrate:fresh --seed` ได้ข้อมูลครบ

**เกณฑ์ผ่านเฟส:** ใน `php artisan tinker` เรียก `Post::published()->with('author','tags','comments')->get()` แล้วได้ข้อมูลถูกต้อง relationship โหลดครบ

---

## เฟส 3 — Inertia CRUD ของ Post (ฝั่ง web ก่อน)

เป้าหมาย: จัดการ Post ครบวงจรผ่าน UI พร้อม authorization

### Authorization
- [ ] `PostPolicy` — `view` (published เห็นได้ทุกคน / draft เห็นเฉพาะ author), `create` (login), `update`/`delete` (เฉพาะ author)
- [ ] register policy + ใส่ `authorize()` ใน controller

### Validation + Logic
- [ ] `StorePostRequest`, `UpdatePostRequest` (validate title/body/tags/cover_image)
- [ ] `Actions/Post/CreatePost` — save post + sync tags + จัดการ cover image + set `published_at`
- [ ] `Actions/Post/UpdatePost`
- [ ] `Actions/Post/DeletePost`

### Controller + หน้า React
- [ ] `PostController` (resource controller) — index/show/create/store/edit/update/destroy เรียก Action
- [ ] index: `Posts/Index.tsx` — list published (+ของตัวเองที่เป็น draft), pagination
- [ ] show: `Posts/Show.tsx` — เนื้อหา + author + tags + comments
- [ ] create/edit: `Posts/Form.tsx` — ฟอร์มเขียน/แก้ (ใช้ `useForm` ของ Inertia), อัปโหลดรูปปก, เลือก tags
- [ ] กำหนด TypeScript types ของ props (`Post`, `Tag`, ฯลฯ) ใน `resources/js/types`
- [ ] ปุ่ม "เผยแพร่ / บันทึกเป็น draft" ทำงานถูกต้อง

**เกณฑ์ผ่านเฟส:** login แล้วสร้าง/แก้/ลบโพสต์ของตัวเองได้, แก้ของคนอื่นไม่ได้ (403), draft คนอื่นมองไม่เห็น

---

## เฟส 4 — Comment + Tag

เป้าหมาย: ปฏิสัมพันธ์บนโพสต์ครบ

### Comment (flat, ต้อง login)
- [ ] `StoreCommentRequest` (validate body)
- [ ] `CommentController@store` — สร้าง comment ผูก user+post (middleware `auth`)
- [ ] `CommentPolicy` — `delete` เฉพาะเจ้าของ comment (หรือ author ของ post)
- [ ] ฟอร์ม comment + รายการ comment ใน `Posts/Show.tsx`
- [ ] guest เห็น comment ได้ แต่ฟอร์มโผล่เฉพาะตอน login

### Tag
- [ ] tag input ในฟอร์มโพสต์ (พิมพ์ชื่อ → หาเจอเดิมหรือสร้างใหม่, sync ผ่าน Action)
- [ ] หน้า/ฟิลเตอร์ดูโพสต์ตาม tag (เช่น `GET /posts?tag=laravel`) — optional แต่แนะนำ
- [ ] generate slug ของ tag อัตโนมัติ

**เกณฑ์ผ่านเฟส:** comment ได้เฉพาะตอน login, ลบ comment ตัวเองได้, ใส่/ถอด tag บนโพสต์ได้

---

## เฟส 5 — REST API ของ Post (จุดที่เห็นคุณค่าของ ADR-0001)

เป้าหมาย: เสิร์ฟ Post เป็น JSON ผ่าน API แยก โดย **reuse Policy + Action เดิม** ไม่เขียน logic ซ้ำ

- [ ] ติดตั้ง/เปิด **Laravel Sanctum** (token-based) + migrate `personal_access_tokens`
- [ ] เพิ่ม `HasApiTokens` ใน `User`
- [ ] endpoint ออก token (เช่น `POST /api/tokens` หรือ login API) สำหรับทดสอบด้วย Postman
- [ ] `PostResource` (+`PostCollection` ถ้าต้องการ) — กำหนด shape ของ JSON
- [ ] `Api/PostController`:
  - [ ] `index`/`show` — public, คืนเฉพาะ **published** (ใช้ scope เดิม)
  - [ ] `store`/`update`/`destroy` — `auth:sanctum` + **เรียก Policy + Action ชุดเดียวกับฝั่ง Inertia**
- [ ] ลงทะเบียน route ใน `routes/api.php`
- [ ] คืน HTTP status ให้ถูก (201 create, 204 delete, 403 unauthorized, 422 validation)
- [ ] ทดสอบครบใน Postman: อ่าน public ได้, เขียนต้องมี token, แก้โพสต์คนอื่นโดน 403

**เกณฑ์ผ่านเฟส:** Postman เรียก CRUD ได้ตามสิทธิ์ และ logic ตรงกับฝั่ง web เป๊ะ (เพราะใช้ Action/Policy ร่วมกัน)

---

## เฟส 2+ (ต่อยอดภายหลัง — ยังไม่ทำตอนนี้)

- [ ] Threaded comments (เพิ่ม `parent_id` self-reference)
- [ ] Role / permission (admin / author / reader)
- [ ] ขยาย REST API ไปครอบ comment / tag
- [ ] Full-text search ของ post
- [ ] Feature/Unit tests ด้วย Pest

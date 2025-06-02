---
title: "วิธีติดตั้งและใช้งาน Hugo แบบง่ายๆ" 
description: "คู่มือการติดตั้งและเริ่มต้นใช้งาน Hugo Static Site Generator สำหรับมือใหม่ ทำเว็บบล็อกด้วย Hugo และ Stack Theme"
date: 2025-05-29 00:00:00+0000
categories:
    - technology
tags:
    - Hugo
    - Web Development
    - Static Site Generator
    - Blog
image: hugo-web-framework-800.webp
resources:
- src: hugo-web-framework-800.webp
  params:
    alt: ภาพแสดง Hugo Web Framework
    srcset:
      - hugo-web-framework-400.webp 400w
      - hugo-web-framework-800.webp 800w
---

# วิธีติดตั้งและใช้งาน Hugo แบบง่ายๆ

Hugo เป็น Static Site Generator ที่มีประสิทธิภาพสูง รวดเร็ว และใช้งานง่าย เหมาะสำหรับการสร้างเว็บบล็อก หรือเว็บไซต์ส่วนตัว ในบทความนี้เราจะมาเรียนรู้วิธีการติดตั้งและใช้งาน Hugo พร้อมกับธีม Stack ที่มีดีไซน์สวยงาม

## ข้อดีของ Hugo

- **เร็วมาก**: Hugo สร้างเว็บไซต์ได้เร็วที่สุดในบรรดา Static Site Generator ทั้งหมด
- **ใช้งานง่าย**: ไม่ต้องเขียนโค้ดซับซ้อน ใช้ Markdown เขียนเนื้อหาได้เลย
- **ปลอดภัย**: เพราะเป็น Static Site จึงไม่มีช่องโหว่ด้านความปลอดภัย
- **ฟรี**: เป็น Open Source สามารถใช้งานได้ฟรี
- **มีธีมหลากหลาย**: มีธีมให้เลือกใช้มากมาย รองรับการปรับแต่งได้ตามต้องการ

## ขั้นตอนการติดตั้ง Hugo

### สำหรับ macOS

ติดตั้งผ่าน Homebrew:

```bash
brew install hugo
```

### สำหรับ Windows

ติดตั้งผ่าน [Chocolatey](https://chocolatey.org/):

```bash
choco install hugo -confirm
```

หรือดาวน์โหลด Binary จาก [Hugo Releases](https://github.com/gohugoio/hugo/releases)

### ตรวจสอบการติดตั้ง

หลังติดตั้งเสร็จ ตรวจสอบเวอร์ชันด้วยคำสั่ง:

```bash
hugo version
```

## การสร้างเว็บไซต์ใหม่

1. สร้างโปรเจ็คใหม่:

```bash
hugo new site myblog
cd myblog
```

2. เพิ่มธีม Stack ซึ่งเป็นธีมที่เราจะใช้:

```bash
git init
git submodule add https://github.com/CaiJimmy/hugo-theme-stack themes/hugo-theme-stack
```

3. คัดลอกไฟล์ตัวอย่างจากธีม:

```bash
cp -r themes/hugo-theme-stack/exampleSite/config.yaml .
mv config.yaml hugo.toml
```

## การตั้งค่าพื้นฐาน

แก้ไขไฟล์ `hugo.toml` ให้เป็นดังนี้:

```toml
baseURL = 'https://yourusername.github.io/'  # เปลี่ยนเป็น URL ของคุณ
languageCode = 'th-th'                     # ภาษาไทย
title = 'ชื่อบล็อกของคุณ'
theme = 'hugo-theme-stack'

[params]
    description = "คำอธิบายเว็บไซต์ของคุณ"
    
    # ตั้งค่า Sidebar
    [params.sidebar]
        emoji = "🌟"                        # อิโมจิที่แสดงข้าง Avatar
        subtitle = "คำอธิบายสั้นๆ"
        avatar = "img/avatar.png"           # รูป Avatar ของคุณ

    # ตั้งค่า Footer
    [params.footer]
        since = 2024                        # ปีเริ่มต้นของบล็อก
        customText = "สร้างด้วย ❤️ และ Hugo" # ข้อความในส่วน Footer

    # ตั้งค่าการแบ่งปัน (Social Sharing)
    [params.social]
        github = "yourusername"
        twitter = "yourusername"
```

## การสร้างบทความใหม่

1. สร้างบทความใหม่:

```bash
hugo new post/my-first-post/index.md
```

2. แก้ไขเนื้อหาในไฟล์ที่สร้าง เพิ่ม Front Matter และเนื้อหา:

```markdown
---
title: "บทความแรกของฉัน"
description: "คำอธิบายบทความ"
date: 2025-05-29
categories:
    - Blog
tags:
    - First Post
image: cover.jpg  # รูปปก (ถ้ามี)
---

เนื้อหาบทความของคุณ...
```

## การทดสอบเว็บไซต์

รันเซิร์ฟเวอร์สำหรับทดสอบ:

```bash
hugo server -D
```

เปิดเบราว์เซอร์ไปที่ `http://localhost:1313` เพื่อดูเว็บไซต์

## การ Deploy เว็บไซต์บน GitHub Pages

1. สร้างไฟล์ GitHub Actions ที่ `.github/workflows/gh-pages.yml`:

```yaml
name: GitHub Pages

on:
  push:
    branches:
      - main  # หรือ master (ตามชื่อ branch หลักของคุณ)

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

2. Push โค้ดขึ้น GitHub:

```bash
git add .
git commit -m "Initial commit"
git push origin main
```

3. ตั้งค่า GitHub Pages:
   - ไปที่ Settings > Pages
   - เลือก Source เป็น "GitHub Actions"
   - รอสักครู่ เว็บไซต์จะ deploy ที่ `https://yourusername.github.io`

## เคล็ดลับการใช้งาน Hugo

1. การจัดการรูปภาพ
   - เก็บรูปภาพไว้ในโฟลเดอร์เดียวกับบทความ (`post/my-post/images/`)
   - ใช้ชื่อไฟล์ที่สื่อความหมายและเป็นภาษาอังกฤษ
   - ลดขนาดรูปภาพก่อนใช้งานเพื่อความเร็วในการโหลด

2. การจัดหมวดหมู่
   - ใช้ Categories สำหรับหมวดหมู่หลัก (เช่น Blog, Tutorial)
   - ใช้ Tags สำหรับคำค้นที่เกี่ยวข้อง
   - ไม่ควรใส่ Tags มากเกินไป (3-5 tags ต่อบทความ)

3. เทคนิค SEO
   - ใส่ Description ทุกครั้ง
   - ตั้งชื่อบทความให้น่าสนใจและตรงประเด็น
   - ใช้ URL ที่อ่านง่าย (slug)
   - ใส่ Alt text ให้รูปภาพ
   - แบ่งเนื้อหาเป็นหัวข้อย่อยด้วย H2, H3
   - เขียนเนื้อหาที่มีคุณภาพและเป็นประโยชน์

4. การปรับแต่ง Theme
   - ศึกษาโครงสร้างธีมที่ `themes/hugo-theme-stack/`
   - สร้างไฟล์ Override ที่ `assets/` เพื่อปรับแต่ง CSS/JS
   - ไม่แก้ไขไฟล์ในโฟลเดอร์ `themes/` โดยตรง

## การบำรุงรักษาเว็บไซต์

1. อัพเดท Hugo เป็นประจำ:
```bash
brew upgrade hugo  # สำหรับ macOS
```

2. อัพเดทธีม:
```bash
git submodule update --remote --merge
```

3. ตรวจสอบ Performance:
   - ใช้ [Google PageSpeed Insights](https://pagespeed.web.dev/)
   - ตรวจสอบ Mobile Responsiveness
   - ดู Analytics เพื่อปรับปรุงเนื้อหา

## สรุป

Hugo เป็นเครื่องมือที่ทรงพลังสำหรับการสร้างเว็บไซต์สถิต การเริ่มต้นใช้งานไม่ยากอย่างที่คิด เพียงทำตามขั้นตอนข้างต้น คุณก็จะได้เว็บบล็อกที่เร็ว ปลอดภัย และดูแลรักษาง่าย หากต้องการศึกษาเพิ่มเติม สามารถดูได้จากแหล่งข้อมูลด้านล่าง

## แหล่งข้อมูลเพิ่มเติม

- [Hugo Documentation](https://gohugo.io/documentation/)
- [Hugo Theme Stack](https://github.com/CaiJimmy/hugo-theme-stack)
- [Hugo Theme Stack Documentation](https://stack.jimmycai.com/)
- [Markdown Guide](https://www.markdownguide.org/)

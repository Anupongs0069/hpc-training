### **🖥️ คู่มือฝึกอบรม: การใช้งาน Mesa 3D และ OpenGL บน LANTA Supercomputer**  
**📌 วัตถุประสงค์:**  
✅ ใช้ **Mesa 3D** บน LANTA สำหรับ **Off-Screen Rendering** (ไม่มี GUI)  
✅ เรียนรู้การใช้ **OSMesa** เพื่อสร้างกราฟิก 3D บน HPC  
✅ ฝึกการใช้ **Slurm Job Scheduling** เพื่อรันงาน OpenGL บน Compute Node  

---

# **🔹 1. ทำไมต้องใช้ HPC สำหรับการเรนเดอร์ 3D?**
💡 **HPC (High-Performance Computing) มีข้อดีเหนือกว่าคอมพิวเตอร์ทั่วไป**  
| **ปัจจัย** | **PC ทั่วไป** | **HPC LANTA** |
|-----------|------------|------------------|
| พลังประมวลผล | **CPU 4-8 คอร์** | **128 คอร์ขึ้นไป** |
| GPU | **ขึ้นกับฮาร์ดแวร์** | **ใช้ CPU เรนเดอร์แทน** (OSMesa) |
| ขนาดภาพที่รองรับ | **ไม่เกิน 4K** | **รองรับภาพความละเอียดสูง 16K+** |
| ประสิทธิภาพ | **ช้าในการเรนเดอร์ใหญ่ ๆ** | **รันพร้อมกันหลายงานได้** |

✅ **LANTA Supercomputer สามารถรันการเรนเดอร์ขนาดใหญ่ที่ต้องใช้พลังคำนวณสูงได้ในเวลาสั้น ๆ!** 🚀  

> 📌 **Mesa 3D บน LANTA รองรับการเรนเดอร์แบบ "Off-Screen" เท่านั้น (ไม่มี GUI) โดยใช้ OSMesa**

---

# **🔹 2. การเข้าสู่ระบบ LANTA**
1️⃣ **เปิด Terminal หรือ MobaXterm**  
2️⃣ **เชื่อมต่อเข้าสู่ LANTA**
```bash
ssh hpcuser@lanta.nstda.or.th
```

---

# **🔹 3. โหลด Mesa 3D และตรวจสอบการติดตั้ง**
📌 **ใช้ `module load` เพื่อตั้งค่าสภาพแวดล้อม**
```bash
module load Mesa/23.1.9-cpeIntel-23.09
```

📌 **ตรวจสอบว่า Mesa ถูกโหลดสำเร็จ**
```bash
echo $LD_LIBRARY_PATH
```
✅ คุณควรเห็น `/opt/cray/pe/Mesa/23.1.9-cpeIntel-23.09/lib64` ซึ่งหมายความว่า Mesa ถูกโหลดเรียบร้อยแล้ว  

> ❌ **LANTA ไม่มี `glxinfo` เพราะไม่มี GUI**  
> **เราจะใช้ OSMesa แทน**

---

# **🔹 4. การเขียนโค้ด OpenGL สำหรับ Off-Screen Rendering**
📌 **สร้างไฟล์โค้ด**
```bash
nano offscreen_opengl.c
```

📄 **คัดลอกโค้ดต่อไปนี้ลงในไฟล์ `offscreen_opengl.c`**
```c
#include <GL/osmesa.h>
#include <GL/gl.h>
#include <stdio.h>

#define WIDTH 800
#define HEIGHT 600

int main() {
    OSMesaContext ctx = OSMesaCreateContext(GL_RGBA, NULL);
    unsigned char buffer[WIDTH * HEIGHT * 4];
    
    if (!OSMesaMakeCurrent(ctx, buffer, GL_UNSIGNED_BYTE, WIDTH, HEIGHT)) {
        printf("Failed to create OSMesa context!\n");
        return 1;
    }

    glClearColor(1.0, 0.0, 0.0, 1.0); // ตั้งค่าพื้นหลังเป็นสีแดง
    glClear(GL_COLOR_BUFFER_BIT);
    glFlush();

    FILE *fp = fopen("output.ppm", "wb");
    fprintf(fp, "P6\n%d %d\n255\n", WIDTH, HEIGHT);
    fwrite(buffer, 1, WIDTH * HEIGHT * 3, fp);
    fclose(fp);

    OSMesaDestroyContext(ctx);
    printf("Rendered output.ppm successfully!\n");
    return 0;
}
```

✅ **โค้ดนี้จะเรนเดอร์พื้นหลังสีแดงลงในไฟล์ `output.ppm` โดยไม่ต้องใช้จอแสดงผล**

---

# **🔹 5. คอมไพล์โค้ด**
📌 **ใช้ Mesa 3D Library (`OSMesa`) เพื่อคอมไพล์**
```bash
gcc offscreen_opengl.c -o offscreen_opengl -lOSMesa
```
✅ ถ้าคอมไพล์ผ่าน จะได้ไฟล์ `offscreen_opengl` พร้อมใช้งาน  

---

# **🔹 6. รัน OpenGL Off-Screen Rendering บน Compute Node**
📌 **รันโปรแกรม**
```bash
./offscreen_opengl
```
✅ ถ้าสำเร็จ คุณจะเห็นข้อความ:
```
Rendered output.ppm successfully!
```
📌 **ตรวจสอบผลลัพธ์**
```bash
ls -lh output.ppm
```
✅ คุณควรเห็นไฟล์ `output.ppm` ที่สร้างขึ้นมา

---

# **🔹 7. ดูผลลัพธ์ (ไม่มี GUI)**
**ถ้าไม่มีโปรแกรมดูภาพบน LANTA เราต้องดาวน์โหลดมาดูบนเครื่องเราเอง**
📌 **ดาวน์โหลดไฟล์กลับมาที่เครื่อง**
```bash
scp hpcuser@transfer.lanta.nstda.or.th:/home/hpcuser/output.ppm .
```
📌 **เปิดภาพด้วย ImageMagick**
```bash
display output.ppm
```
✅ คุณจะเห็น **พื้นหลังสีแดง** ที่เรนเดอร์โดย OSMesa 🎨  

---

# **🔹 8. รัน OpenGL Rendering ผ่าน Slurm**
📌 **สร้างไฟล์ `submit_mesa.sh`**
```bash
nano submit_mesa.sh
```

📄 **เพิ่มโค้ดนี้ลงไป**
```bash
#!/bin/bash
#SBATCH -p compute
#SBATCH -N 1
#SBATCH --ntasks=1
#SBATCH -t 00:30:00
#SBATCH -J mesa_test
#SBATCH -A cb9009xx

module load Mesa/23.1.9-cpeIntel-23.09

./offscreen_opengl
```
📌 **ส่งงานไปที่ HPC**
```bash
sbatch submit_mesa.sh
```
📌 **เช็คสถานะงาน**
```bash
myqueue
```
📌 **เช็คผลลัพธ์**
```bash
cat slurm-xxxxx.out
ls -lh output.ppm
```
✅ คุณจะเห็นว่า **ไฟล์ `output.ppm` ถูกสร้างโดย Compute Node**

---

# **🔹 9. สรุป**
✅ **ใช้ Mesa 3D (OSMesa) สำหรับ Off-Screen Rendering บน HPC**  
✅ **สร้าง OpenGL โค้ดโดยไม่ต้องใช้ GUI**  
✅ **รัน OpenGL ผ่าน Slurm Job Scheduler บน Compute Node**  
✅ **ดาวน์โหลดผลลัพธ์กลับมาเปิดบนเครื่องส่วนตัว**  

🚀 **ตอนนี้คุณสามารถใช้ HPC เพื่อเรนเดอร์ OpenGL แบบ Off-Screen ได้แล้ว!** 🎉  

---

> 📌 **แหล่งอ้างอิง (Citations):**  
> - ThaiSC LANTA User Guide   
> - Mesa 3D Documentation: [https://docs.mesa3d.org/](https://docs.mesa3d.org/)  
> - OSMesa API Reference   

🎨 **อยากลองเพิ่มรายละเอียดในการเรนเดอร์?**  
💡 **ลองเพิ่มวัตถุ 3D หรือใช้ OpenGL Shader ดูนะคะ!** 🚀

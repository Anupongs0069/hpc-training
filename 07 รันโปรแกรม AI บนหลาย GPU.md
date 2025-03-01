### 🚀 **คู่มือฉบับที่ 7: การใช้งาน GPU บน LANTA HPC** 🚀  

---

## 📌 **บทนำ**  
**LANTA HPC** เป็นซูเปอร์คอมพิวเตอร์ที่ให้บริการการประมวลผลขั้นสูง รองรับงานที่ต้องใช้พลังของ GPU เช่น ปัญญาประดิษฐ์ (AI), Deep Learning, และ Machine Learning นักเรียนสามารถใช้งาน GPU บน LANTA เพื่อฝึกฝนการพัฒนาโมเดล AI ได้อย่างมีประสิทธิภาพ

---

## 🎯 **วัตถุประสงค์ของบทเรียน**
1. เข้าใจการทำงานของ HPC และ GPU บน LANTA  
2. เรียนรู้การใช้งาน **PyTorch** บน GPU  
3. ฝึกการรันโค้ดด้วย **SLURM Scheduler**  
4. เรียนรู้การถ่ายโอนไฟล์จากเครื่องของผู้ใช้ไปยัง HPC  
5. ทดสอบการประมวลผลแบบ Multi-GPU  

---

## 🖥 **1. เชื่อมต่อเข้าสู่ LANTA HPC**
ก่อนเริ่มต้น นักเรียนต้องเชื่อมต่อเข้าสู่ระบบ LANTA โดยใช้ SSH

1. **Windows:** ใช้ **MobaXterm** หรือ **PuTTY**
2. **macOS/Linux:** ใช้ Terminal และรันคำสั่ง:
   ```bash
   ssh [ชื่อผู้ใช้]@lanta.nstda.or.th
   ```

หลังจากนั้นระบบจะขอให้ใส่ **รหัสผ่าน** และ **Two-Factor Authentication (2FA)**

---

## 📂 **2. การถ่ายโอนไฟล์ไปยัง LANTA**
### **วิธีการใช้ SFTP ใน MobaXterm (Windows)**
1. เปิด **MobaXterm**
2. กด `Session` → เลือก `SFTP`  
3. ใส่ `Remote Host`: `transfer.lanta.nstda.or.th`  
4. กรอก `Username` และ `Password`  
5. ใส่ **2FA Code** เพื่อเข้าใช้งาน  
6. ลากไฟล์ที่ต้องการไปยังโฟลเดอร์ปลายทางบน LANTA  

### **วิธีการใช้ SCP บน macOS/Linux**
**อัปโหลดไฟล์:**  
```bash
scp myfile.py [ชื่อผู้ใช้]@transfer.lanta.nstda.or.th:/home/[ชื่อผู้ใช้]/
```
**อัปโหลดโฟลเดอร์:**  
```bash
scp -r myfolder/ [ชื่อผู้ใช้]@transfer.lanta.nstda.or.th:/home/[ชื่อผู้ใช้]/
```

---

## 🚀 **3. ติดตั้งและใช้งาน PyTorch บน GPU**
### **สร้าง Virtual Environment บน LANTA**
1. โหลดโมดูล **Mamba**:
   ```bash
   module load Mamba/23.11.0-0
   ```
2. เปิดใช้งาน environment:
   
   ```bash
   conda activate pytorch-2.2.2
   ```
---

## 🧠 **3. ทดสอบการใช้ GPU**
รันโค้ด Python เพื่อตรวจสอบว่า GPU ใช้งานได้:
```python
import torch

print("Torch version:", torch.__version__)
print("CUDA Available:", torch.cuda.is_available())
print("CUDA Version:", torch.version.cuda)
print("GPU Count:", torch.cuda.device_count())
```

ถ้าโค้ดแสดง `CUDA Available: True` และ `GPU Count` มากกว่า 0 แสดงว่า HPC ของคุณรองรับ GPU ✅

---

## 📌 **4. ฝึกฝนการประมวลผลแบบ Multi-GPU**
## เล่น Tensorflow Playground
https://playground.tensorflow.org/#activation=tanh&batchSize=10&dataset=circle&regDataset=reg-plane&learningRate=0.03&regularizationRate=0&noise=0&networkShape=4,2&seed=0.48315&showTestData=false&discretize=false&percTrainData=50&x=true&y=true&xTimesY=false&xSquared=false&ySquared=false&cosX=false&sinX=false&cosY=false&sinY=false&collectStats=false&problem=classification&initZero=false&hideText=false

### **ตัวอย่างโค้ด PyTorch สำหรับ Multi-GPU**
```python
import torch
import torch.nn as nn

# ตรวจสอบจำนวน GPU ที่มีอยู่
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
gpu_count = torch.cuda.device_count()

print(f"ใช้ {gpu_count} GPUs!")

# สร้างโมเดลอย่างง่าย
class SimpleModel(nn.Module):
    def __init__(self):
        super(SimpleModel, self).__init__()
        self.fc = nn.Linear(10, 2)

    def forward(self, x):
        return self.fc(x)

# ใช้ DataParallel เพื่อกระจายงานไปยังหลาย GPU
model = SimpleModel()
if gpu_count > 1:
    model = nn.DataParallel(model)

model.to(device)

# ทดสอบโมเดล
x = torch.randn(5, 10).to(device)
output = model(x)
print(output)
```

---

## 📜 **5. การส่งงานประมวลผลไปยัง LANTA GPU Node**
LANTA ใช้ **SLURM Scheduler** สำหรับจัดการงานประมวลผล ให้สร้างไฟล์ `job_gpu.slurm` แล้วใส่โค้ดนี้:
```bash
#!/bin/bash
#SBATCH -p gpu  # เลือกใช้งาน GPU Node
#SBATCH -N 1     # จำนวนโหนด
#SBATCH -c 16    # จำนวน CPU Core ต่อโหนด
#SBATCH --gpus-per-task=1  # จำนวน GPU ที่ใช้
#SBATCH --ntasks-per-node=4  # จำนวนงานต่อโหนด
#SBATCH -t 02:00:00  # เวลาที่ใช้ (ชั่วโมง:นาที:วินาที)
#SBATCH -A cb9009xx  # ใส่ชื่อโปรเจกต์ของคุณ
#SBATCH -J my_gpu_job  # ตั้งชื่อ job

module load Mamba/23.11.0-0
conda activate pytorch-2.2.2
export PATH=/lustrefs/disk/modules/easybuild/software/Mamba/23.11.0-0/envs/pytorch-2.2.2/bin:$PATH
python3 multi-gpu.py
```
จากนั้นใช้คำสั่ง **sbatch** เพื่อรัน:
```bash
sbatch job_gpu.slurm
```

---

## 🎯 **7. ดูผลการทำงาน**
ตรวจสอบสถานะของงาน:
```bash
myqueue
```
ถ้าต้องการดู log file:
```bash
cat slurm-[JOB_ID].out
```
ยกเลิกงาน:
```bash
scancel [JOB_ID]
```

---

## 📢 **8. สรุป**
✅ เชื่อมต่อเข้าสู่ HPC ด้วย SSH  
✅ ถ่ายโอนไฟล์โดยใช้ SFTP หรือ SCP  
✅ ตั้งค่า Virtual Environment และติดตั้ง PyTorch  
✅ ทดสอบการใช้งาน GPU  
✅ ใช้ Multi-GPU ด้วย PyTorch  
✅ ส่งงานประมวลผลผ่าน SLURM  

---

💡 **หมายเหตุ:**  
- HPC LANTA มี **NVIDIA A100 GPUs** ที่ทรงพลัง ใช้งานให้เต็มประสิทธิภาพ  
- หากมีปัญหาเกี่ยวกับ **โมดูลและแพ็กเกจ** สามารถใช้ `module list` เพื่อตรวจสอบ  
- สามารถดูข้อมูลเพิ่มเติมได้ที่: [ThaiSC LANTA](https://thaisc.io/th/thaisc-resources/lanta)  

📌 **แหล่งข้อมูลอ้างอิง**:  
- คู่มือ Multi-GPU Training  
- คู่มือการใช้คำสั่ง pip และ conda บน LANTA  
- คู่มือการถ่ายโอนไฟล์บน LANTA  

---


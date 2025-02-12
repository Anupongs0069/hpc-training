### **🚀 คู่มือการสร้างชุดข้อมูลสำหรับ Fine-Tuning โมเดล LLaMA 3.2:1B บน LANTA HPC**
**สำหรับนักศึกษาในโครงการ HPC Ignite**

---

## **📌 ทำไมต้องสร้างชุดข้อมูลสำหรับ Fine-Tuning?**
**Fine-Tuning** เป็นการปรับแต่งโมเดล AI ให้เชี่ยวชาญเฉพาะด้านมากขึ้น เช่น:
- **สอนให้โมเดลเข้าใจศัพท์เฉพาะด้าน HPC**
- **เพิ่มความแม่นยำในการตอบคำถามเกี่ยวกับการใช้ LANTA**
- **ช่วยให้ AI จดจำแนวทางการแก้ปัญหา HPC ในบริบทของประเทศไทย**

---

## **✅ 1. รูปแบบของชุดข้อมูลสำหรับ Fine-Tuning**
ชุดข้อมูลที่ใช้ Fine-Tune โมเดลจะอยู่ใน **3 รูปแบบหลัก**:
1. **JSONL (Recommended)** - เหมาะกับ **Hugging Face Transformers**
2. **CSV** - ง่ายต่อการดูและแก้ไขข้อมูล
3. **Text-Based Instruction Data** - ใช้สำหรับ Fine-Tuning แบบ Prompt Learning

---

### **📖 ตัวอย่างข้อมูลแบบ JSONL**
```json
{"input": "HPC คืออะไร?", "output": "HPC หรือ High-Performance Computing คือการใช้คอมพิวเตอร์ที่มีสมรรถนะสูงเพื่อแก้ปัญหาที่ซับซ้อน เช่น การจำลองสภาพอากาศ โมเดล AI และงานวิจัยวิทยาศาสตร์"}
{"input": "ฉันจะใช้ SLURM บน LANTA ได้อย่างไร?", "output": "SLURM เป็นระบบจัดคิวบน LANTA คุณสามารถส่งงานด้วย `sbatch myscript.sh` และตรวจสอบสถานะงานด้วย `squeue -u username`"}
{"input": "อะไรคือวิธีการเพิ่มประสิทธิภาพโค้ด HPC?", "output": "คุณสามารถเพิ่มประสิทธิภาพโค้ดโดยใช้ Parallel Computing เช่น OpenMP, MPI หรือ CUDA เพื่อใช้ประโยชน์จาก GPU และ CPU ที่มีอยู่"}
```
---

### **📖 ตัวอย่างข้อมูลแบบ CSV**
| **input** | **output** |
|-----------|-----------|
| "HPC คืออะไร?" | "HPC หรือ High-Performance Computing คือการใช้คอมพิวเตอร์ที่มีสมรรถนะสูงเพื่อแก้ปัญหาที่ซับซ้อน เช่น การจำลองสภาพอากาศ โมเดล AI และงานวิจัยวิทยาศาสตร์" |
| "ฉันจะใช้ SLURM บน LANTA ได้อย่างไร?" | "SLURM เป็นระบบจัดคิวบน LANTA คุณสามารถส่งงานด้วย `sbatch myscript.sh` และตรวจสอบสถานะงานด้วย `squeue -u username`" |
| "อะไรคือวิธีการเพิ่มประสิทธิภาพโค้ด HPC?" | "คุณสามารถเพิ่มประสิทธิภาพโค้ดโดยใช้ Parallel Computing เช่น OpenMP, MPI หรือ CUDA เพื่อใช้ประโยชน์จาก GPU และ CPU ที่มีอยู่" |

---

## **✅ 2. วิธีสร้างชุดข้อมูล JSONL สำหรับ Fine-Tuning**
### **2.1 ใช้ Python สร้างไฟล์ JSONL**
📌 **สร้างไฟล์ `create_dataset.py`**
```python
import json

# สร้างชุดข้อมูล
dataset = [
    {"input": "HPC คืออะไร?", "output": "HPC หรือ High-Performance Computing คือการใช้คอมพิวเตอร์ที่มีสมรรถนะสูงเพื่อแก้ปัญหาที่ซับซ้อน เช่น การจำลองสภาพอากาศ โมเดล AI และงานวิจัยวิทยาศาสตร์"},
    {"input": "ฉันจะใช้ SLURM บน LANTA ได้อย่างไร?", "output": "SLURM เป็นระบบจัดคิวบน LANTA คุณสามารถส่งงานด้วย `sbatch myscript.sh` และตรวจสอบสถานะงานด้วย `squeue -u username`"},
    {"input": "อะไรคือวิธีการเพิ่มประสิทธิภาพโค้ด HPC?", "output": "คุณสามารถเพิ่มประสิทธิภาพโค้ดโดยใช้ Parallel Computing เช่น OpenMP, MPI หรือ CUDA เพื่อใช้ประโยชน์จาก GPU และ CPU ที่มีอยู่"}
]

# บันทึกเป็น JSONL
with open("hpc_finetune_dataset.jsonl", "w", encoding="utf-8") as f:
    for entry in dataset:
        f.write(json.dumps(entry, ensure_ascii=False) + "\n")

print("✅ สร้างไฟล์ JSONL สำเร็จ: hpc_finetune_dataset.jsonl")
```
📌 **รันสคริปต์เพื่อสร้างไฟล์ JSONL**
```bash
python create_dataset.py
```
🔹 **ไฟล์ `hpc_finetune_dataset.jsonl` จะถูกสร้างขึ้นในโฟลเดอร์ปัจจุบัน**

---

### **2.2 ใช้ Pandas แปลงจาก CSV เป็น JSONL**
หากมีชุดข้อมูลเป็น CSV สามารถใช้ **Pandas** แปลงเป็น JSONL ได้:

📌 **สร้างไฟล์ `convert_csv_to_jsonl.py`**
```python
import pandas as pd
import json

# โหลดไฟล์ CSV
df = pd.read_csv("hpc_finetune_dataset.csv")

# แปลงเป็น JSONL
with open("hpc_finetune_dataset.jsonl", "w", encoding="utf-8") as f:
    for _, row in df.iterrows():
        json.dump({"input": row["input"], "output": row["output"]}, f, ensure_ascii=False)
        f.write("\n")

print("✅ แปลง CSV เป็น JSONL สำเร็จ: hpc_finetune_dataset.jsonl")
```
📌 **รันสคริปต์**
```bash
python convert_csv_to_jsonl.py
```
🔹 **จะได้ไฟล์ `hpc_finetune_dataset.jsonl` พร้อมใช้งาน**

---

## **✅ 3. อัปโหลดชุดข้อมูลไปที่ LANTA**
### **📌 วิธีส่งชุดข้อมูลจากเครื่องของนักศึกษาไปยัง LANTA**
📌 ใช้คำสั่ง `scp` เพื่ออัปโหลดไฟล์ JSONL ไปที่ LANTA:
```bash
scp hpc_finetune_dataset.jsonl username@lanta.thaisc.org:/project/cb900902-hpct01/datasets/
```
หรือใช้ `rsync` สำหรับไฟล์ขนาดใหญ่:
```bash
rsync -av --progress hpc_finetune_dataset.jsonl username@lanta.thaisc.org:/project/cb900902-hpct01/datasets/
```

---

## **✅ 4. ใช้ชุดข้อมูลสำหรับ Fine-Tuning บน LANTA**
เมื่อมีชุดข้อมูลแล้ว สามารถใช้ Fine-Tune โมเดลได้ โดยแก้ไข **`finetune_llama.py`**:
```python
from datasets import load_dataset

# โหลดชุดข้อมูลจากไฟล์ JSONL
dataset = load_dataset("json", data_files="/project/cb900902-hpct01/datasets/hpc_finetune_dataset.jsonl")

# ตรวจสอบข้อมูล
print(dataset)
```

---

## **🚀 สรุป**
✔ **สร้างชุดข้อมูล JSONL และ CSV สำหรับ Fine-Tuning**  
✔ **ใช้ Python สร้างและแปลงข้อมูลเป็น JSONL**  
✔ **อัปโหลดข้อมูลไปที่ LANTA ผ่าน SCP หรือ Rsync**  
✔ **ใช้ชุดข้อมูลสำหรับ Fine-Tuning โมเดล LLaMA 3.2:1B บน LANTA**  


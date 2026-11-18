<div align="center">

# 🖥️ Mini OS — Kernel Bare-Metal x86 (32-bit)

**Implementasi sistem operasi bare-metal berbasis C dan x86 Assembly dengan Multiboot v1, Preemptive Round Robin Scheduler, sinkronisasi thread, deteksi & pencegahan deadlock, alokator memori dinamis, serta CLI shell interaktif.**

![Language](https://img.shields.io/badge/Language-C%20%7C%20x86%20Assembly-00599C?style=flat-square&logo=c&logoColor=white)
![Architecture](https://img.shields.io/badge/Architecture-x86%20(i386%2032--bit)-4A5568?style=flat-square)
![Boot](https://img.shields.io/badge/Bootloader-Multiboot%20v1%20%2F%20GRUB-805AD5?style=flat-square)
![Emulator](https://img.shields.io/badge/Emulator-QEMU-E53E3E?style=flat-square)

</div>

---

## 📋 Daftar Isi

- [Tentang Proyek](#-tentang-proyek)
- [Arsitektur & Tahapan Fitur](#-arsitektur--tahapan-fitur)
- [Struktur Direktori](#-struktur-direktori)
- [Prasyarat & Instalasi](#-prasyarat--instalasi)
- [Cara Menjalankan](#-cara-menjalankan)
- [Kontributor](#-kontributor)

---

## 📖 Tentang Proyek

**Mini OS** adalah proyek pengembangan sistem operasi (*Tugas Besar Sistem Operasi*) yang dibangun dari nol (*from scratch*) tanpa ketergantungan pada *standard library* sistem operasi host. Kernel dimuat pada alamat memori fisik `1 MB` (`0x100000`) menggunakan spesifikasi **Multiboot v1** dan berjalan dalam lingkungan **x86 32-bit Protected Mode**.

---

## ⚙️ Arsitektur & Tahapan Fitur

Pengembangan sistem dibagi ke dalam beberapa tahapan inti:

| Tahap | Modul | Deskripsi Teknis |
| :--- | :--- | :--- |
| **Tahap 1** | **Boot, Drivers & Shell** | • Header **Multiboot v1**, inisialisasi stack `16 KB`, dan entry point `_start` (`boot.asm`).<br>• Driver layar **VGA Text Mode** (`80x25` pada `0xB8000`) dengan dukungan warna, kursor perangkat keras, dan *scrolling* (`vga.c`).<br>• Driver input **PS/2 Keyboard** berbasis *polling* melalui Port `0x60` & `0x64` (`keyboard.c`).<br>• **Interactive CLI Shell** lengkap dengan parser dan handler perintah (`shell.c`). |
| **Tahap 2** | **Kernel Threading** | • Pengelolaan *Kernel Thread* menggunakan **Thread Control Block (TCB)** (`thread.c`).<br>• Mekanisme *context switching* tingkat rendah di Assembly (`context_switch`).<br>• Manajemen siklus hidup thread dengan **5 status thread** (*Ready, Running, Blocked, Terminated*, dll.). |
| **Tahap 3** | **Interrupt & Preemptive Scheduler** | • Konfigurasi **GDT**, **IDT** (Exception `0–31` + `IRQ0`), *remapping* **PIC 8259**, serta timer **PIT 1000 Hz** (`idt.c`).<br>• **Preemptive Round Robin Scheduler** berbasis *Timer Interrupt* (`IRQ0`) dengan perlindungan *critical section* (`irq_save`, `irq_restore`, `cli`, `sti`) (`sched.c`). |
| **Tahap 4** | **Sinkronisasi Thread** | • Implementasi primitif sinkronisasi **Mutex** serta **Binary & Counting Semaphore** berbasis antrean tunggu **FIFO** (`waitq_t`) (`sync.c`). |
| **Tahap 5** | **Manajemen Deadlock** | • Simulasi skenario *deadlock* menggunakan penguncian mutex berlawanan (`deadlock.c`).<br>• Deteksi siklus menggunakan **Wait-For Graph**.<br>• Pencegahan *deadlock* melalui strategi **Lock Ordering**. |
| **Tahap 6** | **Manajemen Memori Dinamis** | • Alokator heap kernel (`kmalloc` / `kfree`) menggunakan algoritma **First-Fit** (`memory.c`).<br>• Mendukung pemecahan blok (*splitting*), penggabungan blok bebas bersebelahan (*coalescing*), dan penanganan aman saat kondisi *Out-of-Memory* (OOM). |

---

## 📂 Struktur Direktori

```text
mini-os/
├── Makefile                # Build system (build, run, iso, clean)
├── linker.ld               # Linker script untuk memuat kernel di memori fisik 1MB (0x100000)
├── setup.sh                # Script bash pengecekan & instalasi otomatis toolchain di Linux/WSL
├── boot/
│   └── boot.asm            # Header Multiboot v1, stack 16KB, _start, context_switch, & ISR/IRQ0 stubs
├── kernel/
│   ├── kernel.c            # Fungsi kernel_main(), verifikasi Multiboot, dan inisialisasi sistem
│   ├── shell.c             # Interactive CLI Shell (parser & handler perintah)
│   ├── thread.c            # Tahap 2: Pengelolaan Kernel Thread (TCB, context switch, 5 status thread)
│   ├── idt.c               # Tahap 3: GDT, IDT (Exception 0-31 + IRQ0), remap PIC 8259, & PIT 1000 Hz
│   ├── sched.c             # Tahap 3: Preemptive Round Robin Scheduler berbasis Timer Interrupt (IRQ0)
│   ├── sync.c              # Tahap 4: Sinkronisasi Mutex & Counting/Binary Semaphore (antrean FIFO)
│   ├── deadlock.c          # Tahap 5: Demo deadlock (mutex berlawanan), deteksi wait-for graph, & pencegahan lock ordering
│   └── memory.c            # Tahap 6: Alokator memori dinamis (kmalloc/kfree, first-fit, split & coalesce, aman saat OOM)
├── drivers/
│   ├── vga.c               # Driver layar VGA Text Mode (80x25 @ 0xB8000, scrolling, kursor, warna)
│   └── keyboard.c          # Driver input PS/2 Keyboard berbasis polling (Port 0x60 & 0x64)
├── include/
│   ├── vga.h               # Header deklarasi fungsi VGA & port I/O (inb/outb)
│   ├── keyboard.h          # Header deklarasi fungsi keyboard PS/2
│   ├── thread.h            # Header struktur data Thread Control Block (TCB) & status thread
│   ├── idt.h               # Header GDT/IDT & timer frekuensi PIT (1000 Hz)
│   ├── irq.h               # Header helper critical section interrupt (irq_save, irq_restore, cli, sti)
│   ├── sched.h             # Header API Preemptive Round Robin Scheduler
│   ├── sync.h              # Header struktur data waitq_t, mutex_t, dan semaphore_t
│   └── shell.h             # Header deklarasi fungsi CLI shell & utilitas string
└── README.md               # Dokumentasi proyek

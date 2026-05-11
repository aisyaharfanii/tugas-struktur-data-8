# 📝 Note-Taking App — Struktur Data

Implementasi struktur data untuk aplikasi **note-taking** menggunakan Python murni (tanpa library eksternal). Proyek ini adalah latihan untuk memahami konsep:

- **Multi-linked list** via tag index
- **Doubly linked list** untuk sorted views
- **Circular buffer** untuk sync tracking

---

## 🗂️ Fitur

| Fitur | Implementasi |
|---|---|
| Multiple tags per note | Tag index (`dict` → `list[NoteNode]`) |
| Chronological view | Doubly linked list (urutan insert) |
| Alphabetical view | Doubly linked list (sorted A–Z) |
| Sync status tracking | Circular buffer (recent changes) |

---

## 🏗️ Arsitektur Struktur Data

```
                    ┌─────────────────────────────────┐
                    │           NoteApp               │
                    │                                 │
   ┌──────────┐     │  _notes: { id → NoteNode }      │
   │ NoteNode │────►│  _tag_index: { tag → [nodes] }  │
   │          │     │  _chron_list: ChronologicalList  │
   │ - title  │     │  _alpha_list: AlphabeticalList   │
   │ - content│     │  _sync_buffer: CircularBuffer    │
   │ - tags   │     └─────────────────────────────────┘
   │ - synced │
   │ - prev/  │
   │   next   │   (chron & alpha pointer masing-masing)
   └──────────┘
```

### 1. `NoteNode`
Node tunggal yang menyimpan data sebuah note. Setiap node memiliki **dua pasang pointer** (`prev/next`) untuk dua doubly linked list yang berbeda (kronologis & alfabetis).

### 2. `ChronologicalList`
Doubly linked list dengan urutan **waktu penambahan** (FIFO). `head` = note terlama, `tail` = note terbaru. Mendukung traversal dua arah.

```
head                              tail
 │                                 │
[Belajar Python] ⇄ [Catatan Mtg] ⇄ [Arsitektur API]
   (terlama)                          (terbaru)
```

### 3. `AlphabeticalList`
Doubly linked list yang **selalu terurut A–Z** berdasarkan judul. Setiap insert mencari posisi yang tepat secara linear.

```
head
 │
[Arsitektur API] ⇄ [Belajar Python] ⇄ [Catatan Meeting] ⇄ [Zenius Review]
      A                   B                   C                   Z
```

### 4. `CircularBuffer`
Buffer dengan kapasitas tetap untuk menyimpan **riwayat perubahan terbaru**. Jika penuh, event paling lama otomatis ditimpa (FIFO wrap-around).

```
Kapasitas: 5 slot

[ ADD "Python" | ADD "Sorting" | SYNC "Python" | DELETE "Sorting" | ADD "API" ]
      ↑ paling lama                                                  ↑ paling baru
```

### 5. Tag Index (`dict` of lists)
Setiap tag menyimpan referensi (pointer) ke semua node yang memiliki tag tersebut. Ini membentuk struktur **multi-linked** — satu note bisa terhubung ke banyak tag, dan satu tag menghubungkan banyak note.

```
tag_index = {
  "python":    [NoteNode#1, NoteNode#4],
  "belajar":   [NoteNode#1, NoteNode#2, NoteNode#5],
  "kerja":     [NoteNode#3, NoteNode#4],
}
```

---

## 🚀 Cara Menjalankan

Tidak perlu install library apapun. Cukup Python 3.10+.

```bash
python note_app.py
```

### Contoh Output

```
✅ Note ditambahkan: [1] 'Belajar Python' | Tags: [python, belajar] | Sync: ✗

📅 Chronological View (terlama → terbaru):
  [1] 'Belajar Python'    | Tags: [python, belajar] | Sync: ✓
  [3] 'Catatan Meeting'   | Tags: [kerja]            | Sync: ✓
  [4] 'Arsitektur API'    | Tags: [python, kerja]    | Sync: ✗

🔤 Alphabetical View (A → Z):
  [4] 'Arsitektur API'    | Tags: [python, kerja]    | Sync: ✗
  [1] 'Belajar Python'    | Tags: [python, belajar]  | Sync: ✓
  [3] 'Catatan Meeting'   | Tags: [kerja]            | Sync: ✓

🏷️  Notes dengan tag 'python':
  [1] 'Belajar Python'
  [4] 'Arsitektur API'

🔁 Sync History (buffer size=5):
  [11:17:58] ADD    → 'Arsitektur API'
  [11:17:58] ADD    → 'Zenius Review'
  [11:17:58] SYNC   → 'Belajar Python'
  [11:17:58] SYNC   → 'Catatan Meeting'
  [11:17:58] DELETE → 'Algoritma Sorting'
```

---

## 📋 API / Penggunaan

```python
from note_app import NoteApp

app = NoteApp(sync_buffer_size=5)

# Tambah note
note = app.add_note("Judul", "Isi note", tags=["tag1", "tag2"])

# Tampilkan views
app.view_chronological()               # terlama → terbaru
app.view_chronological(newest_first=True)  # terbaru → terlama
app.view_alphabetical()                # A → Z

# Filter by tag
app.view_by_tag("python")

# Sync & hapus
app.mark_synced(note_id=1)
app.delete_note(note_id=2)

# Lihat riwayat perubahan
app.view_sync_history()
```

---

## 📊 Kompleksitas

| Operasi | Time Complexity |
|---|---|
| `add_note` | O(n) — insert di alpha list |
| `delete_note` | O(1) — hapus dari linked list |
| `view_chronological` | O(n) |
| `view_alphabetical` | O(n) |
| `view_by_tag` | O(k) — k = jumlah note dengan tag tsb |
| `push` circular buffer | O(1) |
| `get_all` circular buffer | O(capacity) |

---

## 📁 Struktur File

```
note_taking/
└── note_app.py    # Semua implementasi dalam satu file
└── README.md      # Dokumentasi ini
```

---

## 🎓 Konsep yang Dipelajari

- **Doubly Linked List** — navigasi dua arah, insert & delete O(1)
- **Circular Buffer** — fixed-size FIFO, cocok untuk log/history terbatas
- **Inverted Index** — pemetaan tag → list of nodes (mirip search engine)
- **Multiple Pointer per Node** — satu node bisa hidup di beberapa linked list sekaligus

---

> Dibuat sebagai latihan struktur data — Python 3.10+, tanpa dependensi eksternal.

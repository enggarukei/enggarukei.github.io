### **📌 Perbedaan Prerouting dan Forward di Mangle MikroTik**

Ketika kita bekerja dengan **Mangle di Firewall MikroTik**, kita sering menggunakan **prerouting** atau **forward** untuk menandai paket. Namun, ada perbedaan penting dalam cara keduanya bekerja.

---

## **1️⃣ Prerouting vs Forward: Perbedaan Utama**

| Chain          | Kapan Digunakan?                                            | Fungsi Utama                                                                         |
| -------------- | ----------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| **Prerouting** | Saat paket pertama kali masuk ke router                     | Digunakan untuk menandai paket sebelum diproses oleh routing                         |
| **Forward**    | Saat paket diteruskan dari satu interface ke interface lain | Digunakan untuk menandai paket yang melewati router (bukan untuk router itu sendiri) |

---

## **2️⃣ Bagaimana Cara Kerjanya?**

📍 **Prerouting:**

- **Menandai paket lebih awal**, sebelum MikroTik memutuskan apakah paket akan dikirim ke router atau diteruskan ke jaringan lain.
- Berguna jika kita ingin **menandai semua traffic lebih awal**, termasuk traffic yang akan diteruskan atau yang ditujukan ke router.

📍 **Forward:**

- Hanya berlaku untuk **paket yang melewati router**, bukan untuk paket yang berasal dari atau ditujukan ke router itu sendiri.
- Jika paket hanya menuju ke router (misalnya untuk management atau DNS resolver di MikroTik), maka **tidak akan masuk ke chain forward**.

---

## **3️⃣ Kapan Harus Menggunakan Prerouting atau Forward?**

🔹 **Gunakan `prerouting` jika:**

- Anda ingin **menandai paket lebih awal** sebelum routing.
- Anda ingin menangkap semua traffic dari VLAN tertentu (misalnya VLAN Dosen).

🔹 **Gunakan `forward` jika:**

- Anda hanya ingin **menandai paket yang diteruskan antar interface**.
- Paket harus **melewati router** (misalnya dari LAN ke internet atau antar VLAN).

### **Contoh Kasus: Limitasi Bandwidth VLAN Dosen**

Jika kita ingin membatasi **bandwidth untuk setiap perangkat di VLAN Dosen**, kita harus memilih chain yang tepat:

1️⃣ **Jika menggunakan Simple Queue** (Target Address List) → **Gunakan `prerouting`** karena paket perlu ditandai sebelum diteruskan.  
2️⃣ **Jika menggunakan Queue Tree** (Parent: global) → **Gunakan `forward`** karena paket yang dilewatkan ke queue hanya yang diteruskan oleh router.

---

## **🛠 Contoh Konfigurasi Mangle untuk Queue Tree**

Jika Anda ingin membatasi bandwidth **hanya untuk perangkat yang melewati router (bukan perangkat di router)**, gunakan `forward`:

1️⃣ **Menandai Koneksi**

```sh
/ip firewall mangle
add chain=forward src-address=192.168.20.0/24 action=mark-connection new-connection-mark=VLAN-Dosen-Conn
```

2️⃣ **Menandai Paket Berdasarkan Koneksi**

```sh
/ip firewall mangle
add chain=forward connection-mark=VLAN-Dosen-Conn action=mark-packet new-packet-mark=VLAN-Dosen-Pkt
```

3️⃣ **Membuat Queue Tree**

```sh
/queue tree
add name=VLAN-Dosen parent=global packet-mark=VLAN-Dosen-Pkt max-limit=15M
```

📝 **Kenapa pakai forward?**  
Karena kita hanya ingin membatasi paket **yang diteruskan antar interface**, bukan yang masuk ke router.

---

## **📌 Kesimpulan**

|Chain|Gunakan Jika...|
|---|---|
|**Prerouting**|Paket perlu ditandai **sebelum routing diproses** (misal untuk Simple Queue dengan Address List).|
|**Forward**|Paket perlu ditandai **saat diteruskan antar interface** (misal untuk Queue Tree).|

✅ **Kalau pakai Simple Queue → Gunakan `prerouting`**  
✅ **Kalau pakai Queue Tree → Gunakan `forward`**

---

Semoga penjelasan ini membantu! Jika ada yang kurang jelas, jangan ragu untuk bertanya! 🚀😊
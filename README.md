## POLEJ3MBUT

### **Disable Task Manager.reg**
- **Fungsi**: Menonaktifkan Task Manager dan fitur pencarian Windows
- **Target**: `HKEY_CURRENT_USER`
- **Efek**: 
  - Task Manager tidak dapat dibuka
  - Pencarian di Start Menu dinonaktifkan
  - Search Box di taskbar disembunyikan
  - Cortana dan Bing Search dinonaktifkan

### **Enable Task Manager.reg**
- **Fungsi**: Mengaktifkan kembali Task Manager dan fitur pencarian
- **Target**: `HKEY_CURRENT_USER`
- **Efek**: Mengembalikan semua fungsi ke kondisi normal

### **Hide Process.reg** ⚠️ (Butuh Admin Rights)
- **Fungsi**: Menyembunyikan proses `dwagsvc.exe` dari monitoring tools
- **Target**: `HKEY_LOCAL_MACHINE` (Memerlukan hak Administrator)
- **Proses yang disembunyikan**:
  - dwagsvc.exe (DWAgent Service)
- **Efek**:
  - `dwagsvc.exe` tersembunyi dari Task Manager (semua tab)
  - Tidak terdeteksi oleh WMI queries
  - Tidak muncul di PowerShell Get-Process
  - EventLog access dibatasi
  - Performance Counters dinonaktifkan
  - Process Creation Auditing dinonaktifkan
  - Proses tetap berjalan normal di background

### **Show Process.reg** ⚠️ (Butuh Admin Rights)
- **Fungsi**: Mengembalikan visibilitas `dwagsvc.exe` dan monitoring tools
- **Target**: `HKEY_LOCAL_MACHINE` (Memerlukan hak Administrator)
- **Efek**: Mengembalikan visibilitas proses dan mengaktifkan kembali semua monitoring tools

## Cara Penggunaan

### Metode 1: Double Click
1. **Klik kanan** pada file `.reg` yang ingin dijalankan
2. Pilih **"Merge"** atau **"Open"**
3. Klik **"Yes"** pada dialog konfirmasi
4. Klik **"OK"** setelah berhasil

### Metode 2: Command Line
```cmd
# Jalankan sebagai Administrator
reg import "Disable Task Manager.reg"
reg import "Hide Process.reg"

# Untuk mengembalikan
reg import "Enable Task Manager.reg"
reg import "Show Process.reg"
```

### Metode 3: PowerShell
```powershell
# Jalankan sebagai Administrator
Regedit.exe /s "Hide Process.reg"
Regedit.exe /s "Disable Task Manager.reg"

# Untuk mengembalikan
Regedit.exe /s "Show Process.reg"
Regedit.exe /s "Enable Task Manager.reg"
```

### Skenario 1: Disable Total
```
1. Jalankan "Disable Task Manager.reg"
2. Jalankan "Hide Process.reg" (sebagai Admin)
```
**Hasil**: Task Manager dinonaktifkan dan proses tersembunyi dari semua monitoring tools

### Skenario 2: Hide DWAgent Only
```
1. Jalankan "Hide Process.reg" (sebagai Admin)
```
**Hasil**: Task Manager masih bisa dibuka tapi `dwagsvc.exe` tersembunyi dari monitoring

### Skenario 3: Disable Search Only
```
1. Jalankan "Disable Task Manager.reg"
```
**Hasil**: Task Manager dinonaktifkan dan tidak bisa dicari, tapi `dwagsvc.exe` masih terlihat

### Skenario 4: Restore All
```
1. Jalankan "Enable Task Manager.reg"
2. Jalankan "Show Process.reg" (sebagai Admin)
```
**Hasil**: Semua kembali ke kondisi normal

## Peringatan

### Hak Akses
- **File 1 & 2**: Tidak memerlukan hak Administrator
- **File 3 & 4**: **WAJIB** dijalankan sebagai Administrator

### Keamanan
- Selalu backup registry sebelum menggunakan
- Gunakan dengan tanggung jawab
- Jangan gunakan pada sistem yang bukan milik Anda

### Kompatibilitas
- Windows 7, 8, 8.1, 10, 11
- Beberapa fitur mungkin berbeda tergantung versi Windows

## Troubleshooting

### Problem: "Access Denied"
**Solusi**: Jalankan Command Prompt atau PowerShell sebagai Administrator

### Problem: "Registry key not found"
**Solusi**: Pastikan menggunakan versi Windows yang kompatibel

### Problem: Tidak bisa mengembalikan ke normal
**Solusi**: 
1. Jalankan "Enable Task Manager.reg"
2. Jalankan "Show Process.reg" sebagai Administrator
3. Restart komputer jika diperlukan

### Emergency Restore
Jika terjadi masalah, buka Registry Editor manual:
1. Tekan `Win + R`, ketik `regedit`
2. Navigate ke:
   - `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System`
   - Hapus nilai `DisableTaskMgr`
3. Restart komputer

## Menambahkan Proses Custom

Untuk menyembunyikan proses .exe lain, tambahkan entry berikut di **Hide Process.reg**:

```registry
; Menyembunyikan proses custom (contoh: chrome.exe)
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\chrome.exe]
"Debugger"="svchost.exe"
```

Dan di **Show Process.reg** untuk mengembalikan:

```registry
; Mengembalikan visibilitas proses custom
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\chrome.exe]
"Debugger"=-
```

**Contoh proses yang bisa disembunyikan:**
- `taskmgr.exe` - Task Manager
- `procexp.exe` - Process Explorer
- `chrome.exe` - Google Chrome
- `firefox.exe` - Mozilla Firefox
- `notepad.exe` - Notepad
- `calc.exe` - Calculator
- `dwagent.exe` - DWAgent Client

**Catatan**: File saat ini hanya menyembunyikan `dwagsvc.exe`. Untuk menyembunyikan proses lain, edit file sesuai template di atas.

## Technical Details

### Registry Keys yang Dimodifikasi

#### Disable/Enable Task Manager:
- `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System`
- `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer`
- `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced`
- `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Search`

#### Hide/Show Process:
- `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options`
- `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\EventLog`
- `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\WBEM\CIMOM`
- `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\PowerShell`
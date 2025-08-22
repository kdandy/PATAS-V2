## File Registry

### Disable Task Manager.reg
Menghilangkan akses Task Manager dari sistem Windows. Efek:
- Task Manager tidak dapat dibuka melalui Ctrl+Shift+Esc
- Task Manager tidak dapat dibuka melalui Ctrl+Alt+Del
- Task Manager tidak dapat dibuka melalui Start Menu
- Task Manager tidak dapat dibuka melalui Run dialog

### Enable Task Manager.reg
Mengembalikan akses Task Manager ke sistem Windows. Efek:
- Task Manager dapat dibuka kembali melalui semua metode
- Mengembalikan fungsi normal Task Manager

### Hide Process.reg (Basic Method)
Menyembunyikan proses `dwagsvc.exe` dari monitoring tools dan Task Manager "Details" tab. Efek:
- `dwagsvc.exe` tidak muncul di tab "Details", "Processes", dan "Services"
- Proses tetap berjalan di background
- Tidak terdeteksi oleh search/filter di Task Manager
- Tidak terlihat di kolom Name, PID, Status
- Membatasi akses EventLog
- Menonaktifkan Performance Counters
- Menonaktifkan Process Creation Auditing
- Mengatur PowerShell execution policy ke "Restricted"
- Memerlukan hak administrator

### Show Process.reg (Basic Restore)
Mengembalikan visibilitas proses `dwagsvc.exe` di monitoring tools dan Task Manager. Efek:
- `dwagsvc.exe` kembali terlihat di semua tab Task Manager
- Mengaktifkan kembali EventLog access
- Mengaktifkan kembali Performance Counters
- Mengaktifkan kembali Process Creation Auditing
- Mengatur PowerShell execution policy ke "RemoteSigned"

### Hide Process Advanced.reg (Advanced Method)
Metode hiding yang lebih canggih dengan 10 teknik berbeda untuk menyembunyikan `dwagsvc.exe`. Efek:
- Semua efek dari metode basic
- Menonaktifkan Performance Counters sistem
- Menggunakan GlobalFlag dan VerifierFlags di IFEO
- Menonaktifkan WMI Process Enumeration
- Menyembunyikan dari Services.msc
- Menggunakan Registry Virtualization
- Mengaktifkan Silent Process Exit
- Lebih sulit dideteksi oleh security tools
- Efek lebih permanen dan mendalam

### Show Process Advanced.reg (Advanced Restore)
Mengembalikan semua pengaturan dari metode advanced hiding. Efek:
- Mengembalikan semua fungsi monitoring sistem
- Mengaktifkan kembali Performance Counters
- Mengembalikan WMI functionality
- Mengembalikan Services.msc visibility
- Menghapus Registry Virtualization
- Menonaktifkan Silent Process Exit



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

## Troubleshooting

### Jika Registry Entry Tidak Bekerja

Jika proses `dwagsvc.exe` masih terlihat setelah menjalankan registry, coba solusi berikut:

#### 1. Gunakan Metode Advanced
```cmd
# Jalankan sebagai Administrator
regedit /s "Hide Process Advanced.reg"
```

#### 2. Restart Komputer
Beberapa registry changes memerlukan restart untuk efek penuh.

#### 3. Cek Service Status
```cmd
# Stop service terlebih dahulu
net stop "DWAgent"
# Jalankan registry
regedit /s "Hide Process Advanced.reg"
# Start service kembali
net start "DWAgent"
```

#### 4. Manual Registry Edit
Jika masih tidak bekerja, edit registry secara manual:
1. Buka `regedit` sebagai Administrator
2. Navigate ke: `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options`
3. Buat key baru: `dwagsvc.exe`
4. Tambahkan String Value: `Debugger` = `svchost.exe -k netsvcs`

#### 5. Verifikasi Manual
```cmd
# Cek apakah registry entry sudah ada
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\dwagsvc.exe" /v Debugger

# Cek apakah proses masih terlihat
tasklist | findstr dwagsvc.exe
```

### Penyebab Umum Kegagalan
- **Tidak running sebagai Administrator**: Registry HKLM memerlukan elevated privileges
- **Antivirus blocking**: Beberapa antivirus memblokir registry changes
- **Service protection**: Windows melindungi beberapa system services
- **UAC interference**: User Account Control dapat memblokir changes
- **Timing issue**: Service mungkin perlu di-restart setelah registry change

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
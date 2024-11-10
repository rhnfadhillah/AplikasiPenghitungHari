
# Aplikasi Penghitung Hari    

Tugas 4 - Muhammad Raihan Fadhillah 2210010404

## Daftar Isi
- [Deskripsi](#deskripsi)
- [Prerequisites](#prerequisites)
- [Fitur](#fitur)
- [Cara Menjalankan](#cara-menjalankan)
- [Dokumentasi](#dokumentasi)
- [Screenshots](#screenshots)
- [Contoh Penggunaan](#contoh-penggunaan)
- [Pembuat](#pembuat)

## Deskripsi
Aplikasi Penghitungan Hari adalah aplikasi sederhana yang dibuat menggunakan GUI berbasis Java. Aplikasi ini dirancang untuk membantu pengguna menghitung jumlah hari dalam satu bulan dan tahun tertentu. Selain itu, aplikasi ini juga memberikan informasi mengenai hari pertama dan hari terakhir dalam bulan yang dipilih, serta menentukan apakah tahun tersebut adalah tahun kabisat.

## Prerequisites
- JDK versi 8 atau lebih baru.
- IDE seperti IntelliJ IDEA, Eclipse, atau NetBeans untuk menjalankan dan mengembangkan aplikasi.
- Library JCalendar

## Fitur
1. Menghitung jumlah hari dalam bulan yang dipilih.
2. Menentukan apakah tahun yang dipilih adalah tahun kabisat.
3. Menampilkan nama hari pertama dan hari terakhir dari bulan yang dipilih.
4. Menghitung dan menampilkan selisih hari antara dua tanggal yang dipilih.


## Cara Menjalankan
1. Clone atau Download Repository:
    - Clone repository ini atau download sebagai ZIP dan ekstrak.

2. Buka Proyek di IDE:
    - Buka IDE Anda dan import proyek Java yang telah diunduh.

3. Jalankan Aplikasi:
    - Jalankan kelas AplikasiPenghitungHari dari IDE Anda.

## Dokumentasi
- Method konstruktor
``` java
  public AplikasiPenghitungHari() {
        initComponents();
        setupListeners();
        // Gunakan variabel kontrol agar perubahan awal tidak memicu listener
        isUpdating = true;
        
        // Set tanggalAwalCalendar ke tanggal saat ini
        jCalendar1.setDate(new Date());
        
        // Ambil tanggal saat ini
        LocalDate currentDate = LocalDate.now();

        // Sinkronkan tahunSpinner dan bulanComboBox dengan tanggal saat ini
        spinnerTahun.setValue(currentDate.getYear());
        comboBulan.setSelectedIndex(currentDate.getMonthValue() - 1);

        // Nonaktifkan variabel kontrol setelah inisialisasi
        isUpdating = false;
    }
```

- Method setup listener
``` java
private void setupListeners() {
        btnHitung.addActionListener(e -> calculateDays());
        spinnerTahun.addChangeListener(e -> updateCalendar());
        comboBulan.addActionListener(e -> updateCalendar());
        jCalendar1.addPropertyChangeListener("calendar", evt -> {
            if (!isUpdating) {
                updateSpinnerAndComboBoxFromCalendar(); // Update spinner and combo box dari calendar
            }
        });
    }
```

- Method mencari selisih
``` java
 private void calculateTwoDate(){
        LocalDate tanggalAwal = jCalendar1.getDate().toInstant().atZone(java.time.ZoneId.systemDefault()).toLocalDate();
        LocalDate tanggalAkhir = jCalendar2.getDate().toInstant().atZone(java.time.ZoneId.systemDefault()).toLocalDate();

        // Hitung selisih hari
        long selisihHari = ChronoUnit.DAYS.between(tanggalAwal, tanggalAkhir);

        // Perbarui label selisih hari
        labelSelisih.setText("Selisih Dua Tanggal: " + selisihHari + " hari");
    }
```

- Method update jSpinner dan jComboBox
``` java
 private void updateSpinnerAndComboBoxFromCalendar() {
        if (isUpdating) return; // if updating exit the method
        isUpdating = true;

        // getting date from jCalendar
        LocalDate selectedDate = jCalendar1.getDate().toInstant().atZone(java.time.ZoneId.systemDefault()).toLocalDate();

        // set month and years based on jcalendar
        comboBulan.setSelectedIndex(selectedDate.getMonthValue() - 1); // index start from 0
        spinnerTahun.setValue(selectedDate.getYear()); // set year to spinner

        // updating using method
        calculateDays();

        isUpdating = false;
    }
```

- Method hitung berapa jumlah hari, penentuan hari awal dan akhir, dan tahun kabisat
``` java
private void calculateDays() {
        int year = (Integer) spinnerTahun.getValue();
        int monthIndex = comboBulan.getSelectedIndex(); // 0 = January, 11 = December
        Month month = Month.of(monthIndex + 1); // Month is 1-based
        // Get number of days in the month
        int daysInMonth = month.length(LocalDate.of(year, month, 1).isLeapYear());
        labelJumlahHari.setText(String.valueOf(daysInMonth));
        // Check if it's a leap year
        if (LocalDate.of(year, month, 1).isLeapYear()) {
            labelKabisat.setText("Ya");
        } else {
            labelKabisat.setText("Tidak");
        }
        LocalDate firstDay = LocalDate.of(year, month, 1);
        LocalDate lastDay = LocalDate.of(year, month, daysInMonth);


        // Get the names of the first and last days
        String firstDayName = getDayName(firstDay.getDayOfWeek());
        String lastDayName = getDayName(lastDay.getDayOfWeek());
        labelAwalAkhir.setText("Hari Pertama: " + firstDayName + ", Hari Terakhir: " + lastDayName);

    }
```

- Method update calendar
``` java
private void updateCalendar() {

        if (isUpdating) return; // if updating exit this method
        isUpdating = true;

        int bulan = comboBulan.getSelectedIndex(); // index start from 0
        int tahun = (int) spinnerTahun.getValue(); // get years

        // set jcalendar based on combo box and spinner
        LocalDate tanggalBaru = LocalDate.of(tahun, bulan + 1, 1); // +1 because index
        Date tanggal = Date.from(tanggalBaru.atStartOfDay(java.time.ZoneId.systemDefault()).toInstant());
        jCalendar1.setDate(tanggal);

        isUpdating = false;
    }
```

- Method ubah nama hari ke Bahasa Indonesia
``` java
 private String getDayName(DayOfWeek dayOfWeek) {
        switch (dayOfWeek) {
            case MONDAY: return "Senin";
            case TUESDAY: return "Selasa";
            case WEDNESDAY: return "Rabu";
            case THURSDAY: return "Kamis";
            case FRIDAY: return "Jumat";
            case SATURDAY: return "Sabtu";
            case SUNDAY: return "Minggu";
            default: return "";

        }

    }
```
## Screenshots
![Screenshot 2024-11-10 180000](https://github.com/user-attachments/assets/94666a9b-cf57-4aa8-9903-3726b0619dad)

## Contoh Penggunaan
1. Pilih bulan dan tahun yang diinginkan pada jSpinner dan jComboBox atau JCalendar1.
2. Pilih tanggal menggunakan JCalendar1 dan JCalendar2.
3. Klik tombol "Hitung" untuk melihat selisih hari antara dua tanggal yang dipilih, berapa jumlah hari dalam 1 bulan dari JCalendar1, penentuan tahun kabisat berdasarkan JCalendar1 atau melalui jSpinner, dan penentuan hari awal dan hari akhir.


## Pembuat

- Nama : Muhammad Raihan Fadhillah
- NPM : 2210010404

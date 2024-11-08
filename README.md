
# Aplikasi Penghitung Hari    

Tugas 4 - Muhammad Raihan Fadhillah 2210010404

Aplikasi Penghitungan Hari adalah aplikasi sederhana yang dibuat menggunakan GUI berbasis Java. Aplikasi ini dirancang untuk membantu pengguna menghitung jumlah hari dalam satu bulan dan tahun tertentu. Selain itu, aplikasi ini juga memberikan informasi mengenai hari pertama dan hari terakhir dalam bulan yang dipilih, serta menentukan apakah tahun tersebut adalah tahun kabisat.

## Fitur
    1. Pengecekkan berapa jumlah hari dalam bulan dan tahun yang di masukkan pengguna.
    2. Melakukan perhitungan selisih hari antara dua tanggal berdasarkan inputan pengguna.
    3. Mengetahui kapan hari pertama dan hari terakhir pada bulan dan tahun yang di input pengguna.
    4. Mengetahui apakah tahun yang di input pengguna merupakan tahun kabisat atau bukan.
## Dokumentasi
## PenghitungUmurForm.java
- Method tombol hitung (form)
```java
private void btnHitungActionPerformed(java.awt.event.ActionEvent evt) { 
    Date tanggalLahir = dateChooserTanggalLahir.getDate();
    if (tanggalLahir != null) {
        LocalDate lahir = tanggalLahir.toInstant().atZone(ZoneId.systemDefault()).toLocalDate();
        LocalDate sekarang = LocalDate.now();
        String umur = helper.hitungUmurDetail(lahir, sekarang);
        txtUmur.setText(umur);

        LocalDate ulangTahunBerikutnya = helper.hariUlangTahunBerikutnya(lahir, sekarang);
        String hariUlangTahunBerikutnya = helper.getDayOfWeekInIndonesian(ulangTahunBerikutnya);
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd-MM-yyyy");
        String tanggalUlangTahunBerikutnya = ulangTahunBerikutnya.format(formatter);
        
        txtHariUlangTahunBerikutnya.setText(hariUlangTahunBerikutnya + " (" + tanggalUlangTahunBerikutnya + ")");
        
        stopFetching = true;
        if (peristiwaThread != null && peristiwaThread.isAlive()) {
            peristiwaThread.interrupt(); // Beri sinyal ke thread untuk berhenti
        }
        stopFetching = false;

        // Mendapatkan peristiwa penting secara asinkron
        peristiwaThread = new Thread(() -> {
            try {
                txtAreaPeristiwa.setText("Tunggu, sedang mengambil data...\n");
                helper.getPeristiwaBarisPerBaris(ulangTahunBerikutnya, txtAreaPeristiwa, () -> stopFetching);
                if (!stopFetching) {
                    javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.append("Selesai mengambil data peristiwa"));
                }
            } catch (Exception e) {
                if (Thread.currentThread().isInterrupted()) {
                    javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
                }
            }
        });
        peristiwaThread.start();
    }
}

``` 
- Method saat date chooser berubah (form)

```java
private void dateChooserTanggalLahirPropertyChange(java.beans.PropertyChangeEvent evt) {                                                
        txtUmur.setText("");
        txtHariUlangTahunBerikutnya.setText("");
        stopFetching = true;
        if (peristiwaThread != null && peristiwaThread.isAlive()) {
            peristiwaThread.interrupt();
        }
        txtAreaPeristiwa.setText("");
    }     
```

- Method tombol keluar
``` java
System.exit(0);
```

## PenghitungUmurHelper.java

- Method penghitungan umur secara detail (kelas helper)
``` java
 public String hitungUmurDetail(LocalDate lahir, LocalDate sekarang){
       Period period = Period.between(lahir, sekarang);
       return period.getYears() + " tahun, " + period.getMonths() + " bulan, " + period.getDays() + " hari";
    }
```

- Method perhitungan ulang tahun berikutnya (kelas helper)
```java
 public LocalDate hariUlangTahunBerikutnya (LocalDate lahir, LocalDate sekarang) {
        LocalDate ulangTahunBerikutnya = lahir.withYear(sekarang.getYear());
        if (!ulangTahunBerikutnya.isAfter(sekarang)) {
            ulangTahunBerikutnya = ulangTahunBerikutnya.plusYears(1);
        } return ulangTahunBerikutnya;
    }
```

- Method untuk mengambil nama hari dalam Bahasa Indonesia (kelas helper)
``` java
public String getDayOfWeekInIndonesian (LocalDate date){
        switch (date.getDayOfWeek()){
            case MONDAY :
                return "Senin";
            case TUESDAY :
                return "Selasa";
            case WEDNESDAY :
                return "Rabu";
            case THURSDAY :
                return "Kamis";
            case FRIDAY :
                return "Jum'at";
            case SATURDAY :
                return "Sabtu";
            case SUNDAY :
                return "Minggu";
            default :
                return "";
        }
```

- Method untuk menampilkan peristiwa per baris (kelas helper)
``` java
 public void getPeristiwaBarisPerBaris(LocalDate tanggal, JTextArea txtAreaPeristiwa, Supplier<Boolean> shouldStop) {
        try {
                if (shouldStop.get()) {
                    return;
                }
            String urlString = "https://byabbe.se/on-this-day/" +
            tanggal.getMonthValue() + "/" + tanggal.getDayOfMonth() + "/events.json";
            URL url = new URL(urlString);
            HttpURLConnection conn = (HttpURLConnection)
            url.openConnection();
            conn.setRequestMethod("GET");
            conn.setRequestProperty("User-Agent", "Mozilla/5.0");
            int responseCode = conn.getResponseCode();
            if (responseCode != 200) {
            throw new Exception("HTTP response code: " + responseCode +
            ". Silakan coba lagi nanti atau cek koneksi internet.");
        }
        BufferedReader in = new BufferedReader(new
        InputStreamReader(conn.getInputStream()));
        String inputLine;
        StringBuilder content = new StringBuilder();
        while ((inputLine = in.readLine()) != null) {
            if (shouldStop.get()) {
                in.close();
                conn.disconnect();
                javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
                return;
            }
            content.append(inputLine);
        }
        in.close();
        conn.disconnect();
        JSONObject json = new JSONObject(content.toString());
        JSONArray events = json.getJSONArray("events");
        for (int i = 0; i < events.length(); i++) {
            if (shouldStop.get()) {
                javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
                return;
            }
        JSONObject event = events.getJSONObject(i);
        String year = event.getString("year");
        String description = event.getString("description");
        String translatedDescription = translateToIndonesian(description);
        String peristiwa = year + ": " + translatedDescription;
        javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.append(peristiwa + "\n"));
        }
        if (events.length() == 0) {
            javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Tidak ada peristiwa penting yang ditemukan pada tanggal ini."));
        }
        } catch (Exception e) {
            javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Gagal mendapatkan data peristiwa: " + e.getMessage()));
        }
    }
```

- Method terjemah peristiwa penting ke bahasa Indonesia
```java
 public void getPeristiwaBarisPerBaris(LocalDate tanggal, JTextArea txtAreaPeristiwa, Supplier<Boolean> shouldStop) {
        try {
                if (shouldStop.get()) {
                    return;
                }
            String urlString = "https://byabbe.se/on-this-day/" +
            tanggal.getMonthValue() + "/" + tanggal.getDayOfMonth() + "/events.json";
            URL url = new URL(urlString);
            HttpURLConnection conn = (HttpURLConnection)
            url.openConnection();
            conn.setRequestMethod("GET");
            conn.setRequestProperty("User-Agent", "Mozilla/5.0");
            int responseCode = conn.getResponseCode();
            if (responseCode != 200) {
            throw new Exception("HTTP response code: " + responseCode +
            ". Silakan coba lagi nanti atau cek koneksi internet.");
        }
        BufferedReader in = new BufferedReader(new
        InputStreamReader(conn.getInputStream()));
        String inputLine;
        StringBuilder content = new StringBuilder();
        while ((inputLine = in.readLine()) != null) {
            if (shouldStop.get()) {
                in.close();
                conn.disconnect();
                javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
                return;
            }
            content.append(inputLine);
        }
        in.close();
        conn.disconnect();
        JSONObject json = new JSONObject(content.toString());
        JSONArray events = json.getJSONArray("events");
        for (int i = 0; i < events.length(); i++) {
            if (shouldStop.get()) {
                javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Pengambilan data dibatalkan.\n"));
                return;
            }
        JSONObject event = events.getJSONObject(i);
        String year = event.getString("year");
        String description = event.getString("description");
        String translatedDescription = translateToIndonesian(description);
        String peristiwa = year + ": " + translatedDescription;
        javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.append(peristiwa + "\n"));
        }
        if (events.length() == 0) {
            javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Tidak ada peristiwa penting yang ditemukan pada tanggal ini."));
        }
        } catch (Exception e) {
            javax.swing.SwingUtilities.invokeLater(() -> txtAreaPeristiwa.setText("Gagal mendapatkan data peristiwa: " + e.getMessage()));
        }
    }
```
## Pembuat

- Nama : Muhammad Raihan Fadhillah
- NPM : 2210010404


## Screenshots




#Penjelasan dari API info Bed

```
VUE_APP_API_BASE_URL = https://cors.bridged.cc/https://yankes.kemkes.go.id/app/siranap/rumah_sakit?
```

```
https://yankes.kemkes.go.id/app/siranap/rumah_sakit?jenis=1&propinsi=33prop&kabkota=3310#
```

-   Jenis :
    Ada 2 jenis dalam bed-info :
    -   bed untuk dikhususkan untuk covid (jenis 1)
    -   bed untuk umum (jenis 2)
-   Propinsi :
    pada propinsi diisi dengan key tiap-tiap provinsi bersifat unik
-   Kabkota :
    Seperti halnya propinsi, kabkota berisi dari key tiap-tiap kota
-   Untuk Detail Bed berdasarkan Rumah Sakit yang di pilih
    ```
    https://yankes.kemkes.go.id/app/siranap/tempat_tidur?kode_rs=3310405&jenis=1&propinsi=33prop&kabkota=3310
    ```

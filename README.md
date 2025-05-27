
typedef struct {
    char username[20];
    char password[20];
    char kategori[20];
    char kadaluarsa[20];
} Akun;

typedef struct {
    char kategori[20];
    char nama[50];
    char tanggal[15];
    char slot[5];
    int jumlah;
    int total;
} Reservasi;

#define ADMIN "admin"
#define APASS "admin123"

void tampilkanHeader();
void login(int *role, char *username);
void tampilkanMenu(int role, char *username);
void registrasiUser();
void reservasi(char *username);
void lihatReservasi();
void cariReservasi();
void hapusReservasi();
void lihatReservasiUser(char *username);
int hitungBiaya(char kategori[], int jumlah);
void cetakStruk(char *username, Reservasi rsv);

int main() {
    
    int role = 0;
    char username[20];
    tampilkanHeader();
    login(&role, username);
        if (role != 0) {
        printf("\nBerhasil login sebagai %s\n", username);
        tampilkanMenu(role, username);
    }


    return 0;
}

void tampilkanHeader() {
    printf("\n\n");
    printf("\t\t\t============================================\n");
    printf("\t\t\t           Selamat Datang di LDR\n");
    printf("\t\t\t      Layanan Duduk Reservasi Room 19\n");
    printf("\t\t\t============================================\n");
}

int cekMasaAktif(char kadaluarsa[]) {
    struct tm tgl;
    time_t now = time(NULL);
    time_t expired;

    sscanf("%d-%d-%d", &tgl.tm_mday, &tgl.tm_mon, &tgl.tm_year);
    tgl.tm_mon -= 1;
    tgl.tm_year -= 1900;
    tgl.tm_hour = 0; tgl.tm_min = 0; tgl.tm_sec = 0;

    expired = mktime(&tgl);

    return difftime(expired, now) >= 0; // True jika belum kedaluwarsa
}

void login(int *role, char *username) {
    int pilih;
    char user[20], pass[20];
    int valid = 0;

    printf("\t\t\t1. Login\n\t\t\t2. Registrasi User Baru\n");
    printf("====================================================");
    printf("\n\t\t\tPilihan: ");
    scanf("%d", &pilih);
    if (pilih == 2) {
        registrasiUser();
    }

    while (!valid) {
        printf("\t\t\t=======================================\n");
        printf("\t\t\t                Login LDR              \n");
        printf("\t\t\t=======================================\n");
        printf("\t\t\tUsername: ");
        scanf("%s", user);
        printf("\t\t\tPassword: ");
        scanf("%s", pass);

        if (strcmp(user, ADMIN) == 0 && strcmp(pass, APASS) == 0) {
            *role = 1; // admin
            strcpy(username, "Admin"); // SIMPAN sebagai Admin
            valid = 1;
        } else {
        FILE *fp = fopen("akun.txt", "r");
        if(fp == NULL) {
            printf("File akun.txt tidak ditemukan. Silakan registrasi terlebih dahulu.\n");
            exit(1);
        }
        Akun a;
        while (fscanf(fp, "%s %s %s %s", a.username, a.password, a.kategori, a.kadaluarsa) != EOF) {
            if (strcmp(user, a.username) == 0 && strcmp(pass, a.password) == 0) {
                *role = 2; // user
                strcpy(username, a.username);
                valid = 1;
                break;
            }
         }
            fclose(fp);
            if (!valid) printf("Login gagal, coba lagi.\n");
        }
    }
}

void registrasiUser() {
    FILE *fp = fopen("akun.txt", "a");
    Akun a;
    printf("\n\t\t\t========== Registrasi User Baru ==========\n");
    printf("\t\t\tUsername: ");
    scanf("%s", a.username);
    printf("\t\t\tPassword: ");
    scanf("%s", a.password);
    
    // Tambahan input untuk kategori user
    printf("\t\t\tKategori (Umum/Mahasiswa/Pelajar): ");
    scanf("%s", a.kategori);
    
    // Jika bukan Umum, minta input masa aktif
    if (strcmp(a.kategori, "Mahasiswa") == 0 || strcmp(a.kategori, "Pelajar") == 0 || strcmp(a.kategori, "mahasiswa") == 0 || strcmp(a.kategori, "Pelajar") == 0) {
        printf("\t\t\tMasukkan tanggal masa aktif (dd-mm-yyyy): ");
        scanf("%s", a.kadaluarsa);
    } else {
        strcpy(a.kadaluarsa, "00-00-0000"); // Default untuk Umum
    }
    
    fprintf(fp, "%s %s %s %s\n", a.username, a.password, a.kategori, a.kadaluarsa);
    fclose(fp);
    printf("\t\t\tRegistrasi berhasil! Silakan login.\n");
}

void tampilkanMenu(int role, char *username) {
    int pilih;
    do {
        if (role == 1) {
            printf("\n\t\t\t========== Menu Admin ==========\n");
            printf("\t\t\t1. Lihat semua reservasi\n");
            printf("\t\t\t2. Cari reservasi\n");
            printf("\t\t\t3. Hapus reservasi\n");
            printf("\t\t\t4. Keluar\n");
            printf("\t\t\tPilihan: ");
            scanf("%d", &pilih);

            switch(pilih) {
                case 1: lihatReservasi(); break;
                case 2: cariReservasi(); break;
                case 3: hapusReservasi(); break;
                case 4: exit(0);
                default: printf("Pilihan tidak valid!\n");
            }
        } else if (role == 2) {
            printf("\n\t\t\t========== Menu User ==========\n");
            printf("\t\t\t1. Lakukan Reservasi\n");
            printf("\t\t\t2. Lihat Reservasi Saya\n");
            printf("\t\t\t3. Keluar\n");
            printf("\t\t\tPilihan: ");
            scanf("%d", &pilih);

            switch(pilih) {
                case 1: reservasi(username); break;
                case 2: lihatReservasiUser(username); break;
                case 3: exit(0);
                default: printf("\t\t\tPilihan tidak valid!\n");
            }
        }
    } while (1);
}

void reservasi(char *username) {
    Reservasi rsv;
    char bayar;
    FILE *fp = fopen("reservasi.txt", "a");
    FILE *akunFile = fopen("akun.txt", "r");
    
    if(fp == NULL || akunFile == NULL) {
        printf("Gagal membuka file!\n");
        return;
    }
    
    printf("\n\t\t\t========== Form Reservasi ==========\n");
    
    // Cari data user untuk mendapatkan kategori asli
    Akun a;
    int found = 0;
    while (fscanf(akunFile, "%s %s %s %s", a.username, a.password, a.kategori, a.kadaluarsa) != EOF) {
        if (strcmp(a.username, username) == 0) {
            found = 1;
            break;
        }
    }
    fclose(akunFile);
    
    if (!found) {
        printf("\t\t\tData user tidak ditemukan!\n");
        return;
    }
    
    // Jika user terdaftar sebagai Mahasiswa/Pelajar, cek masa aktif
    if (strcmp(a.kategori, "Umum") == 0 || strcmp(a.kadaluarsa, "00-00-0000") == 0) {
        strcpy(rsv.kategori, "Umum");
    } else {
        if (!cekMasaAktif(a.kadaluarsa)) {
            printf("\t\t\tMasa aktif kategori %s telah kedaluwarsa (berlaku hingga %s).\n", a.kategori, a.kadaluarsa);
            printf("\t\t\tReservasi akan dilakukan sebagai Umum.\n");
            strcpy(rsv.kategori, "Umum");
    } else {
        printf("\t\t\tAnda terdaftar sebagai %s (berlaku hingga %s)\n", a.kategori, a.kadaluarsa);
        strcpy(rsv.kategori, a.kategori);
    }
}
    
    strcpy(rsv.nama, username);
    
    printf("\t\t\tTanggal (dd-mm-yyyy): ");
    scanf("%s", rsv.tanggal);
    printf("\t\t\tSlot (A/B/C): ");
    scanf("%s", rsv.slot);
    printf("\t\t\tJumlah orang: ");
    scanf("%d", &rsv.jumlah);

    rsv.total = hitungBiaya(rsv.kategori, rsv.jumlah);
    printf("\t\t\tTotal pembayaran: Rp%d\n", rsv.total);

    printf("\t\t\tApakah Anda sudah melakukan pembayaran? (y/t): ");
    scanf(" %c", &bayar);

    if (bayar == 'y' || bayar == 'Y') {
        fprintf(fp, "%s \"%s\" %s %s %d %d\n", rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot, rsv.jumlah, rsv.total);
        cetakStruk(username, rsv);
        printf("\n\t\t\tReservasi berhasil!\n");
    } else {
        printf("\t\t\tReservasi dibatalkan karena belum melakukan pembayaran.\n");
    }

    fclose(fp);
}

void cetakStruk(char *username, Reservasi rsv) {
    char filename[50];
    sprintf(filename, "struk_%s.txt", username);
    FILE *fs = fopen(filename, "w");
    if(fs == NULL) {
        printf("Gagal membuat file struk\n");
        return;
    }
    fprintf(fs, "============================================\n");
    fprintf(fs, "               STRUK RESERVASI\n");
    fprintf(fs, "============================================\n");
    fprintf(fs, "Nama      : %s\n", rsv.nama);
    fprintf(fs, "Kategori  : %s\n", rsv.kategori);
    fprintf(fs, "Tanggal   : %s\n", rsv.tanggal);
    fprintf(fs, "Slot      : %s\n", rsv.slot);
    fprintf(fs, "Jumlah    : %d orang\n", rsv.jumlah);
    fprintf(fs, "Total     : Rp%d\n", rsv.total);
    fprintf(fs, "============================================\n");
    fprintf(fs, "Silakan tunjukkan struk ini kepada kasir\n");
    fprintf(fs, "Terima kasih telah menggunakan layanan kami!\n");
    fclose(fs);

    // Tampilkan juga ke layar
    printf("\n============================================\n");
    printf("               STRUK RESERVASI\n");
    printf("============================================\n");
    printf("Nama      : %s\n", rsv.nama);
    printf("Kategori  : %s\n", rsv.kategori);
    printf("Tanggal   : %s\n", rsv.tanggal);
    printf("Slot      : %s\n", rsv.slot);
    printf("Jumlah    : %d orang\n", rsv.jumlah);
    printf("Total     : Rp%d\n", rsv.total);
    printf("============================================\n");
    printf("Silakan tunjukkan struk ini kepada kasir\n");
    printf("Terima kasih telah menggunakan layanan kami!\n");
}

int hitungBiaya(char kategori[], int jumlah) {
    int harga;
    if (strcmp(kategori, "Umum") == 0) harga = 35000;
    else if (strcmp(kategori, "Pelajar") == 0) harga = 25000;
    else harga = 25000;
    return jumlah * harga;
}

void lihatReservasi() {
    FILE *fp = fopen("reservasi.txt", "r");
    Reservasi rsv;
    if(fp == NULL) {
        printf("File reservasi.txt tidak ditemukan.\n");
        return;
    }

    printf("\n\t\t\t========== Daftar Reservasi ==========\n");
    printf("\t\t\t%-10s | %-20s | %-12s | %-10s | %-6s | %-10s\n", 
           "\t\t\tKategori", "Nama", "Tanggal", "Slot", "Jumlah", "Total");
    printf("\t\t\t--------------------------------------------------------------------------\n");

    while (fscanf(fp, "%s \"%[^\"]\" %s %s %d %d", 
                  rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot, &rsv.jumlah, &rsv.total) != EOF) {
        printf("%-10s | %-20s | %-12s | %-10s | %-6d | %-10d\n", 
               rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot, rsv.jumlah, rsv.total);
    }

    fclose(fp);
}


void cariReservasi() {
    int pilih;
    char key_nama[50], key_tanggal[20];
    FILE *fp = fopen("reservasi.txt", "r");
    Reservasi rsv;
    int found = 0;

    if (fp == NULL) {
        printf("File reservasi.txt tidak ditemukan.\n");
        return;
    }

    printf("Cari berdasarkan:\n1. Nama\n2. Tanggal\nPilihan Anda: ");
    scanf("%d", &pilih);

    printf("\n%-10s | %-20s | %-12s | %-10s | %-6s | %-10s\n",
           "Kategori", "Nama", "Tanggal", "Slot", "Jumlah", "Total");
    printf("--------------------------------------------------------------------------\n");

    switch (pilih) {
        case 1:
            printf("Input nama: ");
            scanf(" %49[^\n]", key_nama);
            while (fscanf(fp, "%s \"%[^\"]\" %s %s %d %d",
                          rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot,
                          &rsv.jumlah, &rsv.total) != EOF) {
                if (strcmp(rsv.nama, key_nama) == 0) {
                    printf("%-10s | %-20s | %-12s | %-10s | %-6d | %-10d\n",
                           rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot, rsv.jumlah, rsv.total);
                    found = 1;
                }
            }
            break;

        case 2:
            printf("Input tanggal (dd-mm-yyyy): ");
            scanf(" %19s", key_tanggal);
            while (fscanf(fp, "%s \"%[^\"]\" %s %s %d %d",
                          rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot,
                          &rsv.jumlah, &rsv.total) != EOF) {
                if (strcmp(rsv.tanggal, key_tanggal) == 0) {
                    printf("%-10s | %-20s | %-12s | %-10s | %-6d | %-10d\n",
                           rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot, rsv.jumlah, rsv.total);
                    found = 1;
                }
            }
            break;

        default:
            printf("Pilihan tidak valid.\n");
            fclose(fp);
            return;
    }

    if (!found) {
        printf("Data tidak ditemukan.\n");
    }

    fclose(fp);
}


void hapusReservasi() {
    FILE *fp = fopen("reservasi.txt", "r");
    if (fp == NULL) {
        printf("File reservasi.txt tidak ditemukan.\n");
        return;
    }

    FILE *temp = fopen("temp.txt", "w");
    if (temp == NULL) {
        printf("Gagal membuat file temporary.\n");
        fclose(fp);
        return;
    }

    Reservasi rsv;
    char nama[50], tanggal[15];
    int found = 0;

    printf("\t\t\tMasukkan nama: ");
    scanf(" %49[^\n]", nama);  // Membersihkan newline dan membaca hingga 49 karakter
    printf("\t\t\tMasukkan tanggal (dd-mm-yyyy): ");
    scanf(" %14s", tanggal);   // Membaca tanggal

    while (fscanf(fp, "%s \"%[^\"]\" %s %s %d %d",
                 rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot,
                 &rsv.jumlah, &rsv.total) != EOF) {
        if (strcmp(rsv.nama, nama) == 0 && strcmp(rsv.tanggal, tanggal) == 0) {
            found = 1;
        } else {
            fprintf(temp, "%s \"%s\" %s %s %d %d\n",
                    rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot,
                    rsv.jumlah, rsv.total);
        }
    }

    fclose(fp);
    fclose(temp);

    if (found) {
        remove("reservasi.txt");
        rename("temp.txt", "reservasi.txt");
        printf("Data berhasil dihapus.\n");
    } else {
        remove("temp.txt");
        printf("Data tidak ditemukan.\n");
    }

    // Membersihkan buffer input
    while (getchar() != '\n');
}

void lihatReservasiUser(char *username) {
    FILE *fp = fopen("reservasi.txt", "r");
    Reservasi rsv;
    int found = 0;

    if (fp == NULL) {
        printf("File reservasi.txt tidak ditemukan.\n");
        return;
    }

    printf("\n\t\t\t\t========== Reservasi Anda ==========\n");
    printf("\t\t\t==========================================================\n");
    printf("\t\t\t%-10s | %-12s | %-10s | %-6s | %-10s\n",
           "\t\t\tKategori", "Tanggal", "Slot", "Jumlah", "Total");
    printf("\t\t\t==========================================================\n");

    while (fscanf(fp, "%s \"%[^\"]\" %s %s %d %d",
                  rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot,
                  &rsv.jumlah, &rsv.total) != EOF) {
        if (strcmp(rsv.nama, username) == 0) {
            printf("%-10s | %-12s | %-10s | %-6d | %-10d\n",
                   rsv.kategori, rsv.tanggal, rsv.slot, rsv.jumlah, rsv.total);
            found = 1;
        }
    }

    if (!found) {
        printf("Belum ada reservasi yang dilakukan.\n");
    }

    fclose(fp);
}

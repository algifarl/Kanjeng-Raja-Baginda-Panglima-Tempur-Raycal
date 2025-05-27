// 162024019 - Senna Almalia
// 17 Mei 2025
// Program Reservasi Cafe - CLI (Admin dan User)

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

typedef struct {
    char kategori[20];
    char nama[50];
    char tanggal[15];
    char slot[2];
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
int hitungBiaya(char kategori[], int jumlah);
int cekMasaAktif(char *tanggal);
void cetakStruk(char *username, Reservasi rsv);
int cekSlotPenuh(char *tanggal, char *slot);

int cekSlotPenuh(char *tanggal, char *slot) {
    FILE *fp = fopen("reservasi.txt", "r");
    if (fp == NULL) return 0;

    Reservasi rsv;
    int totalOrang = 0;
    while (fscanf(fp, "%s %s %s %s %d %d",
                  rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot,
                  &rsv.jumlah, &rsv.total) != EOF) {
        if (strcmp(rsv.tanggal, tanggal) == 0 && strcmp(rsv.slot, slot) == 0) {
            totalOrang += rsv.jumlah;
        }
    }
    fclose(fp);

    return totalOrang >= 5;
}

int main() {
    int role = 0;
    char username[20];
    tampilkanHeader();
    login(&role, username);
    if (role != 0) {
        printf("\nBerhasil login sebagai %s\n", role == 1 ? "Admin" : "User");
        tampilkanMenu(role, username);
    }
    return 0;
}

void tampilkanHeader() {
    printf("\n\n");
    printf("============================================\n");
    printf("       Selamat Datang di LDR\n");
    printf("  Layanan Duduk Reservasi Room 19\n");
    printf("============================================\n");
}

void login(int *role, char *username) {
    int pilih;
    char user[20], pass[20];
    int valid = 0;

    printf("\n1. Login\n2. Registrasi User Baru\nPilihan: ");
    scanf("%d", &pilih);
    if (pilih == 2) {
        registrasiUser();
    }

    while (!valid) {
        printf("\nLogin\nUsername: ");
        scanf("%s", user);
        printf("Password: ");
        scanf("%s", pass);

        if (strcmp(user, ADMIN) == 0 && strcmp(pass, APASS) == 0) {
            *role = 1;
            valid = 1;
        } else {
            FILE *fp = fopen("akun.txt", "r");
            if (fp == NULL) {
                printf("File akun.txt tidak ditemukan.\n");
                exit(1);
            }

            char uname[20], pw[20], kategori[20], nim[20], jurusan[30], kampus[30], masa_aktif[15];
            while (fscanf(fp, "%s %s %s %s %s %s %s", uname, pw, kategori, nim, jurusan, kampus, masa_aktif) != EOF) {
                if (strcmp(user, uname) == 0 && strcmp(pass, pw) == 0) {
                    *role = 2;
                    strcpy(username, uname);
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
    if (fp == NULL) {
        printf("Gagal membuka file akun.txt\n");
        return;
    }

    char username[20], password[20], kategori[10];
    char nim[20] = "-", jurusan[30] = "-", kampus[30] = "-", masa_aktif[15] = "-";

    printf("\n=== Registrasi User Baru ===\n");
    printf("Username: ");
    scanf("%s", username);
    printf("Password: ");
    scanf("%s", password);

    printf("Kategori (Umum/Pelajar): ");
    scanf("%s", kategori);

    if (strcmp(kategori, "Pelajar") == 0) {
        printf("NIM: ");
        scanf("%s", nim);
        printf("Jurusan: ");
        scanf("%s", jurusan);
        printf("Asal Kampus: ");
        scanf("%s", kampus);
        printf("Masa Aktif Kartu Pelajar (YYYY-MM-DD): ");
        scanf("%s", masa_aktif);
    }

    fprintf(fp, "%s %s %s %s %s %s %s\n", username, password, kategori, nim, jurusan, kampus, masa_aktif);
    fclose(fp);
    printf("Registrasi berhasil! Silakan login.\n");
}

void tampilkanMenu(int role, char *username) {
    int pilih;
    do {
        if (role == 1) {
            printf("\n=== Menu Admin ===\n");
            printf("1. Lihat semua reservasi\n");
            printf("2. Cari reservasi\n");
            printf("3. Hapus reservasi\n");
            printf("4. Keluar\n");
            printf("Pilihan: ");
            scanf("%d", &pilih);

            switch(pilih) {
                case 1: lihatReservasi(); break;
                case 2: cariReservasi(); break;
                case 3: hapusReservasi(); break;
                case 4: exit(0);
                default: printf("Pilihan tidak valid!\n");
            }
        } else if (role == 2) {
            printf("\n=== Menu User ===\n");
            printf("1. Lakukan Reservasi\n");
            printf("2. Keluar\n");
            printf("Pilihan: ");
            scanf("%d", &pilih);

            switch(pilih) {
                case 1: reservasi(username); break;
                case 2: exit(0);
                default: printf("Pilihan tidak valid!\n");
            }
        }
    } while (1);
}

int cekMasaAktif(char *tanggal) {
    int tahun, bulan, hari;
    sscanf(tanggal, "%d-%d-%d", &tahun, &bulan, &hari);

    time_t now = time(NULL);
    struct tm *tm_now = localtime(&now);

    if ((tahun > (tm_now->tm_year + 1900)) ||
        (tahun == (tm_now->tm_year + 1900) && bulan > (tm_now->tm_mon + 1)) ||
        (tahun == (tm_now->tm_year + 1900) && bulan == (tm_now->tm_mon + 1) && hari >= tm_now->tm_mday)) {
        return 1;
    } else {
        return 0;
    }
}

void reservasi(char *username) {
    Reservasi rsv;
    char bayar;
    FILE *fp = fopen("reservasi.txt", "a");
    if (fp == NULL) {
        printf("Gagal membuka file reservasi.txt\n");
        return;
    }

    FILE *fa = fopen("akun.txt", "r");
    if (fa == NULL) {
        printf("File akun.txt tidak ditemukan!\n");
        fclose(fp);
        return;
    }

    char uname[20], pass[20], kategori[20], nim[20], jurusan[30], kampus[30], masa_aktif[15];
    int ditemukan = 0;

    while (fscanf(fa, "%s %s %s %s %s %s %s", uname, pass, kategori, nim, jurusan, kampus, masa_aktif) != EOF) {
        if (strcmp(uname, username) == 0) {
            strcpy(rsv.kategori, kategori);
            ditemukan = 1;
            break;
        }
    }
    fclose(fa);

    if (!ditemukan) {
        printf("Data user tidak ditemukan!\n");
        fclose(fp);
        return;
    }

    if (strcmp(rsv.kategori, "Pelajar") == 0 && !cekMasaAktif(masa_aktif)) {
    printf("Masa aktif kartu pelajar sudah habis, akan dihitung sebagai kategori Umum.\n");
    strcpy(rsv.kategori, "Umum");
}


    strcpy(rsv.nama, username);
    printf("\n=== Form Reservasi ===\n");
    printf("Tanggal (dd-mm-yyyy): ");
    scanf("%s", rsv.tanggal);
    printf("Slot (A/B/C): ");
    scanf("%s", rsv.slot);

    // Cek slot penuh sebelum input jumlah orang
    if (cekSlotPenuh(rsv.tanggal, rsv.slot)) {
        printf("Maaf, slot %s pada tanggal %s sudah penuh.\n", rsv.slot, rsv.tanggal);
        fclose(fp);
        return;
    }

    printf("Jumlah orang: ");
    scanf("%d", &rsv.jumlah);

    rsv.total = hitungBiaya(rsv.kategori, rsv.jumlah);
    printf("Kategori Anda: %s\n", rsv.kategori);
    printf("Total pembayaran: Rp%d\n", rsv.total);
    printf("Apakah Anda sudah melakukan pembayaran? (y/t): ");
    scanf(" %c", &bayar);

    if (bayar == 'y' || bayar == 'Y') {
        fprintf(fp, "%s %s %s %s %d %d\n", rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot, rsv.jumlah, rsv.total);
        cetakStruk(username, rsv);
        printf("\nReservasi berhasil!\n");
    } else {
        printf("Reservasi dibatalkan.\n");
    }

    fclose(fp);
}

void cetakStruk(char *username, Reservasi rsv) {
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
    int harga = (strcmp(kategori, "Umum") == 0) ? 35000 : 25000;
    return jumlah * harga;
}

void lihatReservasi() {
    FILE *fp = fopen("reservasi.txt", "r");
    if (fp == NULL) {
        printf("Data reservasi belum ada.\n");
        return;
    }
    Reservasi rsv;
    printf("\n=== Daftar Reservasi ===\n");
    printf("Kategori | Nama | Tanggal | Slot | Jumlah | Total\n");
    printf("-------------------------------------------------\n");
    while (fscanf(fp, "%s %s %s %s %d %d",
                  rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot,
                  &rsv.jumlah, &rsv.total) != EOF) {
        printf("%-8s | %-10s | %-10s | %-4s | %-6d | Rp%-8d\n",
               rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot, rsv.jumlah, rsv.total);
    }
    fclose(fp);
}

void cariReservasi() {
    char cari[50];
    int ketemu = 0;
    FILE *fp = fopen("reservasi.txt", "r");
    if (fp == NULL) {
        printf("Data reservasi belum ada.\n");
        return;
    }
    printf("Masukkan nama untuk cari reservasi: ");
    scanf("%s", cari);
    Reservasi rsv;
    while (fscanf(fp, "%s %s %s %s %d %d",
                  rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot,
                  &rsv.jumlah, &rsv.total) != EOF) {
        if (strcmp(rsv.nama, cari) == 0) {
            printf("\nDitemukan reservasi:\n");
            printf("Kategori: %s\nNama: %s\nTanggal: %s\nSlot: %s\nJumlah: %d\nTotal: Rp%d\n",
                   rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot, rsv.jumlah, rsv.total);
            ketemu = 1;
        }
    }
    if (!ketemu) {
        printf("Reservasi dengan nama %s tidak ditemukan.\n", cari);
    }
    fclose(fp);
}

void hapusReservasi() {
    char nama[50];
    int ketemu = 0;
    FILE *fp = fopen("reservasi.txt", "r");
    FILE *temp = fopen("temp.txt", "w");
    if (fp == NULL) {
        printf("Data reservasi belum ada.\n");
        return;
    }
    printf("Masukkan nama reservasi yang ingin dihapus: ");
    scanf("%s", nama);
    Reservasi rsv;
    while (fscanf(fp, "%s %s %s %s %d %d",
                  rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot,
                  &rsv.jumlah, &rsv.total) != EOF) {
        if (strcmp(rsv.nama, nama) != 0) {
            fprintf(temp, "%s %s %s %s %d %d\n",
                    rsv.kategori, rsv.nama, rsv.tanggal, rsv.slot,
                    rsv.jumlah, rsv.total);
        } else {
            ketemu = 1;
        }
    }
    fclose(fp);
    fclose(temp);
    remove("reservasi.txt");
    rename("temp.txt", "reservasi.txt");
    if (ketemu) {
        printf("Reservasi dengan nama %s berhasil dihapus.\n", nama);
    } else {
        printf("Reservasi dengan nama %s tidak ditemukan.\n", nama);
    }
}

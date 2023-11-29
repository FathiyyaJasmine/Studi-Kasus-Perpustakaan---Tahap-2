# Studi-Kasus-Perpustakaan---Tahap-2
package perpustakaan;

import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.Random;
import java.util.Scanner;

public class Perpustakaan {
    private static final Admin admin = new Admin("adminUsername", "adminPassword");
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        Buku buku1 = new Buku("Statistika dan Probabilitas", "Dwi Handako", "ISBN-919-0-7087-2367-9", true);
        Buku buku2 = new Buku("Laskar Pelangi", "Mark Manson", "ISBN-912-1-2083-2757-3", true);
        Buku buku3 = new Buku("Kisah untuk Geri", "Erisca Febriani", "SBN-9117-2-2153-2237-1", true);
        admin.getDaftarBuku().add(buku1);
        admin.getDaftarBuku().add(buku2);
        admin.getDaftarBuku().add(buku3);
        
        while(true){
        System.out.println("Selamat Datang di Perpustakaan");
        System.out.println("-----------------------------");
        System.out.println("Pilih Login Sebagai:");
        System.out.println("1. Admin");
        System.out.println("2. Anggota Perpustakaan");
        System.out.println("-----------------------------");
        int opsi = scanner.nextInt();
        scanner.nextLine();

        switch (opsi) {
            case 1:
                handleAdmin();
                break;
            case 2:
                handleAnggotaPerpustakaan();
                break;
            default:
                System.out.println("Pilihan tidak valid");
            }
        }
    }

    private static void handleAnggotaPerpustakaan() {
        Scanner scanner = new Scanner(System.in);

        System.out.println("Masukkan ID Anggota:");
        int idAnggota = scanner.nextInt();
        scanner.nextLine(); 
        System.out.println("Masukkan Nama Anggota:");
        String namaAnggota = scanner.nextLine();
        System.out.println("Masukkan Alamat Anggota:");
        String alamat = scanner.nextLine();
        AnggotaPerpustakaan anggota = new AnggotaPerpustakaan();
        anggota.idAnggota = idAnggota;
        anggota.namaAnggota = namaAnggota;
        anggota.alamat = alamat;
        
        System.out.println("Selamat Datang, " + anggota.namaAnggota + "!");
        System.out.println("-----------------------------");
        System.out.println("Pilihan Anggota Perpustakaan:");
        System.out.println("1. Pinjam Buku");
        System.out.println("2. Kembalikan Buku");
        System.out.println("-----------------------------");
        
        int opsi;
        if (scanner.hasNextInt()) {
            opsi = scanner.nextInt();
            scanner.nextLine(); 

            switch (opsi) {
                case 1:
                    handlePeminjaman(anggota);
                    break;
                case 2:
                    handlePengembalian(anggota);
                    break;
                default:
                    System.out.println("Pilihan tidak valid");
            }
        } else {
            System.out.println("Input tidak valid. Pilihan harus berupa angka.");
        }
    }
   
    private static void handleAdmin() {
        if (admin.loginAdmin()) {
            System.out.println("Admin berhasil login.");
            admin.handleAdminActions();
        } else {
            System.out.println("Login Admin gagal. Username atau password salah.");
        }
    }

    private static void handlePeminjaman(AnggotaPerpustakaan anggota) {
        Scanner scanner = new Scanner(System.in);
        System.out.println("Masukkan judul buku yang ingin dipinjam:");
        String judulBuku = scanner.nextLine();
        
        Buku buku = cariBuku(judulBuku);
        if (buku != null) {
            anggota.peminjamanBuku(buku);
            
            TransaksiPeminjaman transaksi = new TransaksiPeminjaman();
            transaksi.anggota = anggota;
            transaksi.buku = buku;
            transaksi.tanggalPinjam = new Date();
            transaksi.durasiPinjam = 7;           
            System.out.println("INFORMASI PEMINJAMAN:");
            System.out.println("-----------------------------");
            transaksi.infoPeminjaman();
        } else {
            System.out.println("Buku tidak ditemukan.");
        }
    }    

    private static void handlePengembalian(AnggotaPerpustakaan anggota) {
        Scanner scanner = new Scanner(System.in);
        System.out.println("Masukkan judul buku yang ingin dikembalikan:");
        String judulBuku = scanner.nextLine();
        
        Buku buku = cariBuku(judulBuku);
        if (buku != null) {
            if (!buku.tersedia) {
                if (buku.pinjaman.isSameMember(anggota)) {
                    anggota.kembalikanBuku(buku);

                    TransaksiPengembalian transaksi = new TransaksiPengembalian();
                    transaksi.anggota = anggota;
                    transaksi.buku = buku;
                    transaksi.transaksiPeminjaman = new TransaksiPeminjaman();
                    transaksi.transaksiPeminjaman.tanggalPinjam = new Date(System.currentTimeMillis() - (7 * 24 * 60 * 60 * 1000));
                    transaksi.tanggalPengembalian = new Date();

                    System.out.println("INFORMASI PENGEMBALIAN:");
                    System.out.println("-----------------------------");
                    transaksi.infoPengembalian();

                    long remainingDays = (transaksi.transaksiPeminjaman.tanggalPinjam.getTime()
                            + (transaksi.transaksiPeminjaman.durasiPinjam * 24 * 60 * 60 * 1000)
                            - System.currentTimeMillis()) / (24 * 60 * 60 * 1000);

                    if (remainingDays >= 0) {
                        Notifikasi notifikasi = new Notifikasi();
                        notifikasi.idNotifikasi = "1";
                        notifikasi.anggota = anggota;
                        notifikasi.isi = "Waktu pengembalian buku anda  melebihi batas pengembalian" ;
                        notifikasi.tampilkanNotifikasi();
                    } else {
                        Notifikasi notifikasi = new Notifikasi();
                        notifikasi.idNotifikasi = "2";
                        notifikasi.anggota = anggota;
                        notifikasi.isi = "Waktu pengembalian anda bebas denda, Terima kasih telah tepat waktu mengembalikannya";
                        notifikasi.tampilkanNotifikasi();
                    }
                } else {
                    System.out.println(
                            "Maaf, Anda tidak dapat mengembalikan buku ini. Buku dipinjam oleh anggota lain.");
                }
            } else {
                System.out.println("Buku " + buku.judul + " sudah tersedia. Pengembalian tidak diperlukan.");
            }
        } else {
            System.out.println("Buku tidak ditemukan.");
        }
    }
    
    private static Buku cariBuku(String judul) {
        for (Buku buku : admin.getDaftarBuku()) {
            if (buku.judul.equalsIgnoreCase(judul)) {
                return buku;
            }
        }
        return null;
    }    
}

class Admin {
    private final String[] VALID_USERNAMES = {"nazwa", "haya"};
    private final String[] VALID_PASSWORDS = {"123456", "455678"};
    String username;
    String password;
    private final List<Buku> daftarBuku = new ArrayList<>();

    Admin(String username, String password) {
        this.username = username;
        this.password = password;
    }
    boolean isValidPassName() {
        for (int i = 0; i < VALID_USERNAMES.length; i++) {
            if (username.equals(VALID_USERNAMES[i]) && password.equals(VALID_PASSWORDS[i])) {
                return true;
            }
        }
        return false;
    }

    boolean loginAdmin() {
        Scanner scanner = new Scanner(System.in);
        
        System.out.println("Masukkan username Admin:");       
        this.username = scanner.nextLine();        
        System.out.println("Masukkan password Admin:");
        this.password = scanner.nextLine();
        return isValidPassName();
    }

    void tambahBuku() {
        Scanner scanner = new Scanner(System.in);

        System.out.println("Masukkan judul buku:");
        String judul = scanner.nextLine();
        System.out.println("Masukkan pengarang buku:");
        String pengarang = scanner.nextLine();
        System.out.println("Masukkan nomor ISBN buku:");
        String nomorISBN = scanner.nextLine();
        Buku buku = new Buku(judul, pengarang, nomorISBN, true);
        daftarBuku.add(buku);
        System.out.println("Buku berhasil ditambahkan.");
    }
    
    void handleAdminActions() {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            System.out.println("-----------------------------");
            System.out.println("Pilihan Admin:");
            System.out.println("1. Tambah Buku");
            System.out.println("2. Lihat Daftar Buku");
            System.out.println("3. Keluar");
            System.out.println("-----------------------------");
            int opsi = scanner.nextInt();
            scanner.nextLine();
            
            switch (opsi) {
                case 1:
                    tambahBuku();
                    break;
                case 2:
                    lihatDaftarBuku();
                    break;
                case 3:
                    System.out.println("Keluar dari admin mode.");
                    return;
                default:
                    System.out.println("Pilihan tidak valid");
            }
        }
    }
    void lihatDaftarBuku() {
        System.out.println("Daftar Buku Tersedia:");
        for (Buku buku : daftarBuku) {
            buku.lihatInfoBuku();
            System.out.println("-----------------------------");
        }
    }
    List<Buku> getDaftarBuku() {
        return daftarBuku;
    }
}

class AnggotaPerpustakaan {
    int idAnggota;
    String namaAnggota;
    String alamat;
    String sejarahPeminjaman;
    TransaksiPeminjaman transaksiPeminjaman;

    void peminjamanBuku(Buku buku) {
       if (buku.tersedia) {
           System.out.println("Buku " + buku.judul + " berhasil dipinjam.");
           buku.tersedia = false; 
           buku.pinjaman= this; 
       } else {
           System.out.println("Buku " + buku.judul + " tidak tersedia untuk dipinjam.");
       }
    }

    void kembalikanBuku(Buku buku) {
        if (!buku.tersedia) {
            if (buku.pinjaman.isSameMember(this)) { 
                System.out.println("Buku " + buku.judul + " berhasil dikembalikan.");
                buku.tersedia = true;
                buku.pinjaman = null; 
            } else {
                System.out.println("Maaf, Anda tidak dapat mengembalikan buku ini. Buku dipinjam oleh anggota lain.");
            }
        } else {
            System.out.println("Buku " + buku.judul + " sudah tersedia.");
        }
    }
    
    public boolean isSameMember(AnggotaPerpustakaan otherMember) {
        return this.idAnggota == otherMember.idAnggota && this.namaAnggota.equals(otherMember.namaAnggota) && this.alamat.equals(otherMember.alamat);
    }
}

class TransaksiPeminjaman {
    int idTransaksi = 0;
    AnggotaPerpustakaan anggota;
    Buku buku;
    Date tanggalPinjam;
    int durasiPinjam;

    void infoPeminjaman() {
        Random random = new Random();
        idTransaksi = random.nextInt(1000);       
        System.out.println("ID Transaksi: " + idTransaksi);
        System.out.println("Nama Anggota: " + anggota.namaAnggota);
        System.out.println("Judul Buku: " + buku.judul);
        System.out.println("Tanggal Pinjam: " + tanggalPinjam);
        System.out.println("Durasi Pinjam: " + durasiPinjam + " hari");
    }
}

class TransaksiPengembalian {
    private int idTransaksi;
    AnggotaPerpustakaan anggota;
    Buku buku; 
    TransaksiPeminjaman transaksiPeminjaman; 
    Date tanggalPengembalian;

    void infoPengembalian() { 
        Random random = new Random();
        idTransaksi = random.nextInt(1000);
        System.out.println("ID Transaksi: " + idTransaksi);
        System.out.println("Nama Anggota: " + anggota.namaAnggota);
        System.out.println("Judul Buku: " + buku.judul);
        System.out.println("Tanggal Pengembalian: " + tanggalPengembalian);
        System.out.println("Denda: " + denda());
   }
    double denda() {
        double denda = 0.0;
        if (new Date().after(transaksiPeminjaman.tanggalPinjam)) {
            long selisihHari = (tanggalPengembalian.getTime() - transaksiPeminjaman.tanggalPinjam.getTime()) / (24 * 60 * 60 * 1000)-7;
            if (selisihHari > transaksiPeminjaman.durasiPinjam) {
                denda = (selisihHari - transaksiPeminjaman.durasiPinjam) * 500;
            }
        }
        return denda; 
   }   
}

class Notifikasi {
    String idNotifikasi; 
    AnggotaPerpustakaan anggota;  
    String isi;
    void tampilkanNotifikasi() {
        System.out.println("Notifikasi untuk " + anggota.namaAnggota + ": " + isi);
    }   
}

class Buku {
    String judul;
    private String pengarang;
    private String nomorISBN;
    boolean tersedia;
    AnggotaPerpustakaan pinjaman;

    public Buku(String judul, String pengarang, String nomorISBN, boolean tersedia) {
        this.judul = judul;
        this.pengarang = pengarang;
        this.nomorISBN = nomorISBN;
        this.tersedia = tersedia;
    }

    public String getJudul() {
        return judul;
    }
    public void setJudul(String judul) {
        this.judul = judul;
    }
    public String getPengarang() {
        return pengarang;
    }
    public void setPengarang(String pengarang) {
        this.pengarang = pengarang;
    }
    public String getNomorISBN() {
        return nomorISBN;
    }
    public void setNomorISBN(String nomorISBN) {
        this.nomorISBN = nomorISBN;
    }
    public boolean isTersedia() {
        return tersedia;
    }
    public void setTersedia(boolean tersedia) {
        this.tersedia = tersedia;
    }

    void lihatInfoBuku() {
        System.out.println("Judul: " + judul);
        System.out.println("Pengarang: " + pengarang);
        System.out.println("Nomor ISBN: " + nomorISBN);
        System.out.println("Tersedia: " + (tersedia ? "Ya" : "Tidak"));
    }
}


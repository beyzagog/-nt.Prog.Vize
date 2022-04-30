# -nt.Prog.Vize
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net;
using System.Net.Http;
using System.Web.Http;
using vize01.Models;
using vize01.ViewModel;

namespace vize01.Controllers
{
    public class ServisController : ApiController
    {
        DB01Entities db = new DB01Entities();
        SonucModel sonuc = new SonucModel();

        #region Üye
        [HttpGet]
        [Route("api/uyeliste")]
        public List<UyeModel> UyeListe()
        {
            List<UyeModel> liste = db.Uye.Select(x => new UyeModel()
            {
                uyeId = x.uyeId,
                uyeAdSoyad = x.uyeAdSoyad,
                email = x.email,
                uyeKullaniciAdi = x.uyekullaniciAdi,
                sifre = x.sifre,
                cinsiyet = x.cinsiyet,
                dogumTarihi = x.dogumTarihi,
                admin = x.admin
            }).ToList();

            return liste;
        }

        [HttpGet]
        [Route("api/uyebyid/{uye_id}")]
        public UyeModel UyeById(int uye_id)
        {
            UyeModel kayit = db.Uye.Where(s => s.uyeId == uye_id).Select(x => new UyeModel()
            {
                uyeId = x.uyeId,
                uyeAdSoyad = x.uyeAdSoyad,
                email = x.email,
                uyeKullaniciAdi = x.uyekullaniciAdi,
                sifre = x.sifre,
                cinsiyet = x.cinsiyet,
                dogumTarihi = x.dogumTarihi,
                admin = x.admin
            }).SingleOrDefault();

            return kayit;
        }

        [HttpPost]
        [Route("api/uyeekle")]
        public SonucModel UyeEkle(UyeModel model)
        {
            if (db.Uye.Count(s => s.uyekullaniciAdi == model.uyeKullaniciAdi || s.email == model.email) > 0)
            {
                sonuc.islem = false;
                sonuc.mesaj = "Girilen Kullanıcı Adı veya E-Posta Adresi Kayıtlıdır!";
                return sonuc;
            }

            Uye yeni = new Uye();
            yeni.uyeAdSoyad = model.uyeAdSoyad;
            yeni.email = model.email;
            yeni.uyekullaniciAdi = model.uyeKullaniciAdi;
            yeni.sifre = model.sifre;
            yeni.cinsiyet = model.cinsiyet;
            yeni.dogumTarihi = model.dogumTarihi;
            yeni.admin = model.admin;


            db.Uye.Add(yeni);
            db.SaveChanges();
            sonuc.islem = true;
            sonuc.mesaj = "Üye Eklendi";
            return sonuc;
        }

        [HttpPut]
        [Route("api/uyeduzenle")]
        public SonucModel UyeDuzenle(UyeModel model)
        {
            Uye kayit = db.Uye.Where(s => s.uyeId == model.uyeId).SingleOrDefault();

            if (kayit == null)
            {
                sonuc.islem = false;
                sonuc.mesaj = "Kayıt Bulunamadı";
                return sonuc;
            }
            kayit.uyeAdSoyad = model.uyeAdSoyad;
            kayit.email = model.email;
            kayit.uyekullaniciAdi = model.uyeKullaniciAdi;
            kayit.sifre = model.sifre;
            kayit.cinsiyet = model.cinsiyet;
            kayit.dogumTarihi = model.dogumTarihi;
            kayit.admin = model.admin;


            db.SaveChanges();
            sonuc.islem = true;
            sonuc.mesaj = "Üye Düzenlendi";

            return sonuc;
        }

        [HttpDelete]
        [Route("api/uyesil/{uye_id}")]
        public SonucModel UyeSil(int uye_id)
        {
            Uye kayit = db.Uye.Where(s => s.uyeId == uye_id).SingleOrDefault();

            if (kayit == null)
            {
                sonuc.islem = false;
                sonuc.mesaj = "Kayıt Bulunamadı";
                return sonuc;
            }

            if (db.Soru.Count(s => s.uyeId == uye_id) > 0)
            {
                sonuc.islem = false;
                sonuc.mesaj = "Üzerinde Makale Kaydı Olan Üye Silinemez!";
                return sonuc;
            }

            if (db.Cevap.Count(s => s.uyeId == uye_id) > 0)
            {
                sonuc.islem = false;
                sonuc.mesaj = "Üzerinde Yorum Kaydı Olan Üye Silinemez!";
                return sonuc;
            }

            db.Uye.Remove(kayit);
            db.SaveChanges();
            sonuc.islem = true;
            sonuc.mesaj = "Üye Silindi";
            return sonuc;
        }
        #endregion

        #region kategori


        [HttpGet]
        [Route("api/kategoriliste")]

        public List<KategoriModel> KategoriListe()
        {
            List<KategoriModel> liste = db.Kategori.Select(x => new KategoriModel()
            {
                kategori_id = x.kategoriId,
                kategori_adi = x.kategoriAdi,
                kat_soru_say = x.soru.Count
            }).ToList();

            return liste;
        }

        [HttpGet]
        [Route("api/kategoribyid/{kategori_id}")]
        public KategoriModel KategoriById(int kategoriId)
        {
            KategoriModel kayit = db.Kategori.Where(s => s.kategoriId == kategoriId).Select(x => new KategoriModel()
            {
                kategori_id = x.kategoriId,
                kategori_adi = x.kategoriAdi,
                kat_soru_say = x.soru.Count
            }).SingleOrDefault();
            return kayit;
        }

        [HttpPost]
        [Route("api/kategoriekle")]
        public SonucModel KategoriEkle(KategoriModel model)
        {
            if (db.Kategori.Count(s => s.kategoriAdi == model.kategori_adi) > 0)
            {
                sonuc.islem = false;
                sonuc.mesaj = "Girilen Kategori Adı Kayıtlıdır!";
                return sonuc;
            }

            Kategori yeni = new Kategori();
            yeni.kategoriAdi = model.kategori_adi;
            db.Kategori.Add(yeni);
            db.SaveChanges();

            sonuc.islem = true;
            sonuc.mesaj = "Kategori Eklendi";
            return sonuc;
        }

        [HttpPut]
        [Route("api/kategoriduzenle")]
        public SonucModel KategoriDuzenle(KategoriModel model)
        {
            Kategori kayit = db.Kategori.Where(s => s.kategoriId == model.kategori_id).FirstOrDefault();

            if (kayit == null)
            {
                sonuc.islem = false;
                sonuc.mesaj = "Kayıt Bulunamadı!";
                return sonuc;
            }

            kayit.kategoriAdi = model.kategori_adi;
            db.SaveChanges();

            sonuc.islem = true;
            sonuc.mesaj = "Kategori Düzenlendi";
            return sonuc;
        }

        [HttpDelete]
        [Route("api/kategorisil/{kategoriId}")]
        public SonucModel KategoriSil(int kategoriId)
        {
            Kategori kayit = db.Kategori.Where(s => s.kategoriId == kategoriId).FirstOrDefault();
            if (kayit == null)
            {
                sonuc.islem = false;
                sonuc.mesaj = "Kayıt Bulunamadı!";
                return sonuc;
            }

            if (db.Kategori.Count(s => s.kategoriId == kategoriId) > 0)
            {
                sonuc.islem = false;
                sonuc.mesaj = "Üzerinde Soru Kayıtlı Kategori Silinemez!";
                return sonuc;
            }

            db.Kategori.Remove(kayit);
            db.SaveChanges();

            sonuc.islem = true;
            sonuc.mesaj = "Kategori Silindi";
            return sonuc;
        }


        #endregion

        #region soru

        [HttpGet]
        [Route("api/soruliste")]
        public List<SoruModel> MakaleListe()
        {
            List<SoruModel> liste = db.Soru.Select(x => new SoruModel()
            {
                soru_id = x.soruId,
                Soru = x.soru,
                kategoriId = x.kategoriId,
                kategori_adi = x.Kategori.kategori_adi,
                uye_id = x.uyeId,
                uye_kullanici_adi = x.Uye.uye_kullanici_adi

            }).ToList();

            return liste;
        }
        [HttpGet]
        [Route("api/sorulistesoneklenenler/{s}")]
        public List<SoruModel> SoruListeSonEklenenler(int s)
        {
            List<SoruModel> liste = db.Soru.OrderByDescending(o => o.soru_id).Take(s).Select(x => new SoruModel()
            {
                soru_id = x.soru_id,
                Soru = x.Soru,
                kategori_id = x.kategori_id,
                kategori_adi = x.Kategori.kategori_adi,
                uye_id = x.uye_id,
                uye_kullanici_adi = x.Uye.uye_kullanici_adi


            }).ToList();

            return liste;
        }
        [HttpGet]
        [Route("api/sorulistebykatid/{kategori_id}")]
        public List<SoruModel> SoruListeBykategori_id(int kategori_id)
        {
            List<SoruModel> liste = db.Soru.Where(s => s.kategoriId == kategori_id).Select(x => new SoruModel()
            {
                soru_id = x.soru_id,
                Soru = x.Soru,
                kategori_id = x.kategori_id,
                kategori_adi = x.Kategori.kategori_adi,
                uye_id = x.uye_id,
                uye_kullanici_adi = x.Uye.uye_kullanici_adi

            }).ToList();

            return liste;
        }
        [HttpGet]
        [Route("api/sorulistebyuyeid/{uye_id}")]
        public List<SoruModel> SoruListeByUyeId(int uye_id)
        {
            List<SoruModel> liste = db.Soru.Where(s => s.uyeId == uye_id).Select(x => new SoruModel()
            {
                soru_id = x.soruId,
                Soru = x.Soru,
                kategori_id = x.kategoriId,
                kategori_adi = x.Kategori.kategori_adi,
                uye_id = x.uyeId,
                uye_kullanici_adi = x.Uye.uye_kullanici_adi

            }).ToList();

            return liste;
        }

        [HttpGet]
        [Route("api/sorubyid/{soru_id}")]
        public SoruModel SoruById(int soru_id)
        {
            SoruModel kayit = db.Soru.Where(s => s.soru_id == soru_id).Select(x => new SoruModel()
            {
                soru_id = x.soru_id,
                Soru = x.Soru,
                kategori_id = x.kategori_id,
                kategori_adi = x.Kategori.kategori_adi,
                uye_id = x.uye_id,
                uye_kullanici_adi = x.Uye.uye_kullanici_adi
            }).SingleOrDefault();
            return kayit;
        }

        [HttpPost]
        [Route("api/soruekle")]
        public SonucModel MakaleEkle(SoruModel model)
        {
            if (db.Soru.Count(s => s.soru_id == model.soru_id) > 0)
            {
                sonuc.islem = false;
                sonuc.mesaj = "Girilen Makale Başlığı Kayıtlıdır!";
                return sonuc;
            }

            Soru yeni = new Soru();
            yeni.Soru = model.Soru;
            yeni.kategori_id = model.kategori_id;
            yeni.uyeId = model.uye_id;
            db.Soru.Add(yeni);
            db.SaveChanges();

            sonuc.islem = true;
            sonuc.mesaj = "Soru Eklendi";
            return sonuc;
        }

        [HttpPut]
        [Route("api/soruduzenle")]
        public SonucModel MakaleDuzenle(SoruModel model)
        {
            Soru kayit = db.Soru.Where(s => s.soru_id == model.soru_id).SingleOrDefault();
            if (kayit == null)
            {
                sonuc.islem = false;
                sonuc.mesaj = "Kayıt Bulunamadı!";
                return sonuc;
            }
            kayit.soru = model.soru;
            kayit.kategori_id = model.kategori_id;
            kayit.uyeId = model.uye_id;
            db.SaveChanges();

            sonuc.islem = true;
            sonuc.mesaj = "Soru Düzenlendi";
            return sonuc;

        }

        [HttpDelete]
        [Route("api/soruesil/{soru_id}")]
        public SonucModel MakaleSil(int soru_id)
        {
            Soru kayit = db.Soru.Where(s => s.soru_id == soru_id).SingleOrDefault();
            if (kayit == null)
            {
                sonuc.islem = false;
                sonuc.mesaj = "Kayıt Bulunamadı!";
                return sonuc;
            }

            if (db.Cevap.Count(s => s.soru_id == soru_id) > 0)
            {
                sonuc.islem = false;
                sonuc.mesaj = "Üzerinde Cevap Kayıtlı Soru Silinemez!";
                return sonuc;
            }

            db.Soru.Remove(kayit);
            db.SaveChanges();

            sonuc.islem = true;
            sonuc.mesaj = "Soru Silindi";
            return sonuc;
        }


        #endregion

        #region cevap

        [HttpGet]
        [Route("api/cevapliste")]
        public List<CevapModel> YorumListe()
        {
            List<CevapModel> liste = db.Cevap.Select(x => new CevapModel()
            {
                cevap_id = x.cevap_id,
                Cevap = x.Cevap,
                soru_id = x.soru_id,
                uye_id = x.uyeId,
                uye_kullanici_adi = x.Uye.uye_kullanici_adi,
                soruİcerik = x.sorular.soru,

            }).ToList();
            return liste;
        }
        [HttpGet]
        [Route("api/sorulistebyuyeid/{uye_id}")]
        public List<CevapModel> CevapListeByUyeId(int uye_id)
        {
            List<CevapModel> liste = db.Cevap.Where(s => s.uyeId == uye_id).Select(x => new CevapModel()
            {
                cevap_id = x.cevap_id,
                Cevap = x.cevap,
                soru_id = x.soru_id,
                uye_id = x.uye_id,
                uye_kullanici_adi = x.Uye.uye_kullanici_adi,
                soruİcerik = x.sorular.soru,

            }).ToList();
            return liste;
        }
        [HttpGet]
        [Route("api/cevaplistebysoruid/{soru_id}")]
        public List<CevapModel> CevapListeBymakaleId(int soru_id)
        {
            List<CevapModel> liste = db.Cevap.Where(s => s.soru_id == soru_id).Select(x => new CevapModel()
            {
                cevap_id = x.cevap_id,
                Cevap = x.cevap,
                soru_id = x.soru_id,
                uye_id = x.uye_id,
                uye_kullanici_adi = x.Uye.uye_kullanici_adi,
                soruİcerik = x.sorular.soru,

            }).ToList();
            return liste;
        }
        [HttpGet]
        [Route("api/cevaplistesoneklenenler/{s}")]
        public List<CevapModel> CevapListeSonEklenenler(int s)
        {
            List<CevapModel> liste = db.Cevap.OrderByDescending(o => o.soru_id).Take(s).Select(x => new CevapModel()
            {
                cevap_id = x.cevap_id,
                Cevap = x.Cevap,
                soru_id = x.soru_id,
                uye_id = x.uye_id,
                uye_kullanici_adi = x.Uye.uye_kullanici_adi,
                soruİcerik = x.sorular.soru,

            }).ToList();
            return liste;
        }


        [HttpGet]
        [Route("api/cevapbyid/{cevap_id}")]

        public CevapModel YorumById(int cevap_id)
        {
            CevapModel kayit = db.Cevap.Where(s => s.cevap_id == cevap_id).Select(x => new CevapModel()
            {
                cevap_id = x.cevap_id,
                Cevap = x.Cevap,
                soru_id = x.soru_id,
                uye_id = x.uye_id,
                uye_kullanici_adi = x.uye_kullanici_adi,
                soruİcerik = x.sorular.soru,
            }).SingleOrDefault();

            return kayit;
        }

        [HttpPost]
        [Route("api/cevapekle")]
        public SonucModel YorumEkle(CevapModel model)
        {
            if (db.Cevap.Count(s => s.uyeId == model.uye_id && s.Soru_id == model.soru_id && s.Cevap == model.Cevap) > 0)
            {
                sonuc.islem = false;
                sonuc.mesaj = "Aynı Kişi, Aynı Soruya Aynı Cevabı Yapamaz!";
                return sonuc;
            }

            Cevap yeni = new Cevap();
            yeni.cevap_id = model.cevap_id;
            yeni.Cevap = model.;Cevap;
            yeni.soru_id = model.soru_id;
            yeni.uyeId = model.uye_id;
            db.Cevap.Add(yeni);
            db.SaveChanges();

            sonuc.islem = true;
            sonuc.mesaj = "Cevap Eklendi";

            return sonuc;
        }
        [HttpPut]
        [Route("api/cevapduzenle")]
        public SonucModel CevapDuzenle(CevapModel model)
        {

            Cevap kayit = db.Cevap.Where(s => s.cevap_id == model.cevap_id).SingleOrDefault();

            if (kayit == null)
            {
                sonuc.islem = false;
                sonuc.mesaj = "Kayıt Bulunamadı!";
                return sonuc;
            }

            kayit.cevap_id = model.cevap_id;
            kayit.Cevap = model.Cevap;
            kayit.soru_id = model.soru_id;
            kayit.uyeId = model.uye_id;

            db.SaveChanges();

            sonuc.islem = true;
            sonuc.mesaj = "Cevap Düzenlendi";

            return sonuc;
        }

        [HttpDelete]
        [Route("api/cevapsil/{cevap_id}")]
        public SonucModel YorumSil(int cevap_id)
        {
            Cevap kayit = db.Cevap.Where(s => s.cevap_id == cevap_id).SingleOrDefault();

            if (kayit == null)
            {
                sonuc.islem = false;
                sonuc.mesaj = "Kayıt Bulunamadı!";
                return sonuc;
            }

            db.Cevap.Remove(kayit);
            db.SaveChanges();

            sonuc.islem = true;
            sonuc.mesaj = "Cevap Silindi";

            return sonuc;
        }


        #endregion
    }
}

    }
}

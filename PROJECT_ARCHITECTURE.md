# Fitness Projesi Mimari Dokümantasyonu

## Genel Bakış
**Fitness Program Oluşturucu (v5)**, kullanıcıların antrenman programları oluşturmasına, yönetmesine ve yazdırmasına olanak tanıyan sunucusuz (serverless), tek sayfalık bir web uygulamasıdır (SPA). **HTML5**, **Tailwind CSS** ve **Alpine.js** kullanılarak modern bir arayüzle oluşturulmuş olup, veri kalıcılığı için tarayıcı `localStorage` özelliğini kullanır.

## Teknoloji Yığını
-   **Çekirdek**: HTML5, JavaScript (ES6+)
-   **Reaktivite & Durum Yönetimi**: [Alpine.js](https://alpinejs.dev/) (v3.13.3)
-   **Stil**: [Tailwind CSS](https://tailwindcss.com/) (CDN)
-   **İkonlar**: [Phosphor Icons](https://phosphoricons.com/)
-   **Depolama**: Tarayıcı `localStorage`
-   **Dağıtım**: Gerektirmez (yerel olarak dosya protokolü üzerinden çalışır)

## Veri Mimarisi
Uygulama durumu, `localStorage` içinde saklanan ve Alpine.js `x-data` içinde yönetilen birkaç ana nesne aracılığıyla işlenir.

### 1. Veri Modelleri

#### Kategori (Category)
Egzersiz gruplarını temsil eder.
```json
{
  "id": "string (zaman damgası)",
  "name": "string",
  "coverImage": "string (URL veya Base64)",
  "exercises": "Exercise[]"
}
```

#### Egzersiz (Exercise - Kütüphane)
Bir kategori içindeki egzersiz şablonudur.
```json
{
  "id": "string (zaman damgası)",
  "name": "string",
  "imageUrl": "string (URL veya Base64)",
  "reps": "number (varsayılan: 10)",
  "sets": "number (varsayılan: 3)"
}
```

#### Program Öğesi (Program Item)
Aktif antrenman programına eklenen bir egzersiz veya başlık örneğidir.
```json
{
  "id": "string (zaman damgası)",
  "libraryId": "string (opsiyonel, Exercise.id referansı)",
  "categoryId": "string (opsiyonel, Category.id referansı)",
  "name": "string",
  "imageUrl": "string",
  "reps": "number",
  "sets": "number",
  "type": "string ('exercise' | 'title')"
}
```

#### Kayıtlı Program (Saved Program)
Kullanıcının daha sonra kullanmak üzere kaydettiği tam antrenman programı.
```json
{
  "id": "string (zaman damgası)",
  "name": "string",
  "createdAt": "string (Tarih)",
  "exercises": "ProgramItem[]"
}
```

### 2. Durum Yönetimi (`localStorage` Anahtarları)
-   `categories`: Kütüphanedeki tüm kategoriler ve egzersizler.
-   `programExercises`: Mevcut oluşturulan programdaki öğeler listesi.
-   `savedPrograms`: Kaydedilmiş hazır programlar listesi.
-   `programName`: Mevcut programın adı.
-   `darkMode`: `'true'` veya `'false'`. Tema tercihi.
-   `pageState`: UI durumunu korumak için (örn. hangi kategori açık).

## Temel Fonksiyonel Bileşenler

### 1. UI ve Etkileşim (Alpine.js)
-   **Reaktivite**: Veri değişiklikleri otomatik olarak UI'a yansır (`x-model`, `x-text`).
-   **Görünüm Yönetimi**: `view` değişkeni ile 'Kategoriler' ve 'Kategori Detay' görünümleri arasında geçiş yapılır.
-   **Modallar**: `modals` nesnesi üzerinden yönetilen (Category Ekle, Exercise Ekle, Program Kaydet, Hazır Programlar) popup pencereler.
-   **Sürükle ve Bırak**: Native HTML5 Drag & Drop API kullanılarak program listesindeki öğelerin sıralanması sağlanır.

### 2. Yazdırma Sistemi (Print System)
-   **Sayfalama**: `printProgram()` fonksiyonu, egzersizleri 5'erli gruplara ayırarak (chunking) her biri A4 boyutunda sayfalar oluşturur.
-   **Dinamik Oluşturma**: Yazdırma işlemi öncesinde gizli bir `div` (`#print-container`) içine HTML enjekte edilir.
-   **Görsel Yükleme**: Yazdırma penceresi açılmadan önce tüm görsellerin yüklenmesi beklenir (`Promise.all`).

### 3. İçe/Dışa Aktarma (Import/Export)
-   **JSON Tabanlı**: Tüm veriler (`categories`, `savedPrograms`) JSON formatında dışa aktarılır ve geri yüklenebilir.
-   **Base64 Desteği**: Görseller Base64 formatında saklandığı için JSON dosyası içinde taşınabilir.

## Dosya Yapısı
Uygulama tek bir HTML dosyasından oluşur: `Egzersiz Programı v5.html`.
-   **Head**: CDN kütüphaneleri (Tailwind, Alpine, Phosphor) ve Yazdırma CSS'i.
-   **Body**:
    -   **Header**: Logo ve araç çubuğu.
    -   **Main**: Sol panel (Kütüphane) ve Sağ panel (Program Oluşturucu).
    -   **Modallar**: Gizli katmanlar halinde bekleyen formlar.
    -   **Script**: `fitnessApp()` fonksiyonu içinde tüm uygulama mantığı.

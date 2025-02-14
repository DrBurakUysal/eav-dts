# 🎯 Entity Attribute Value (EAV) / Dynamic Table Structure (DTS)

## 📚 İçindekiler
- [Genel Bakış](#-genel-bakış)
- [Ne İşe Yarar? — Neden Kullanmalıyım?](#-ne-işe-yarar-—-neden-kullanmalıyım)
- [Mimari Yapı](#-mimari-yapı)
- [Kurulum](#-kurulum)
- [Kullanım Örnekleri](#-kullanım-örnekleri)
- [Örnek Senaryolar](#örnek-senaryolar)
- [API Referansı](#api-referansı)
- [Performans Optimizasyonları](#performans-optimizasyonları)
- [Lisans](#lisans)

## 🎯 Genel Bakış

DynamicTable yapısı, geleneksel veritabanı tasarımının esnek olmayan yapısını aşmak için geliştirilmiş bir modeldir. 
Bu yapı, dinamik olarak yeni özellikler ekleyebilmenize ve farklı veri tiplerini yönetebilmenize olanak sağlar.

Az miktarda veri saklayacağınız tüm tablolar için DynamicTable yapısını kullanabilirsiniz. Veri tabanınızda çok sayıda tablo olmasındansa, DynamicTable yapısını kullanarak daha az tablo sayısı ile çalışabilirsiniz. Yönetim kolaylığı ve performans için DynamicTable yapısı, mimarisi doğru kurulduğu sürece başarılı çalışmaktadır.

## ❓ Ne İşe Yarar? — Neden Kullanmalıyım?

Az miktarda veri tutacak tablolara ihtiyacınız vardır ve bu veriler için devamlı küçük küçük tablolar oluşturmak, veritabanı tarafında dağıtık ve yönetilemez durumlar oluşturabilir.

DynamicTable yapısı, 3 tablo ile verilerin tutulabilmesini sağlamaktadır.

Dinamik bir şekilde tablo ve tablolara kolon eklememizi sağlar. Bu da client tarafından geliştirmeye ihtiyaç duymadan Tablo ve Kolonlarının eklenmesini sağlatır.

### 📊 Geleneksel vs DynamicTable Karşılaştırması

![Karşılaştırma](docs/images/comparison.png)

### Geleneksel vs DynamicTable Karşılaştırması

#### Geleneksel Yaklaşım
- ✅ Basit ve anlaşılır yapı
- ✅ Doğrudan SQL sorguları
- ❌ Schema değişiklikleri zor
- ❌ Yeni özellikler için ALTER TABLE gerekir

#### DynamicTable Yaklaşım
- ✅ Esnek ve genişletilebilir yapı
- ✅ Yeni özellikler dinamik olarak eklenebilir
- ✅ Schema değişikliği gerektirmez
- ❌ Daha karmaşık sorgular
- ❌ Performans optimizasyonu gerektirir

## 🏗 Mimari Yapı

### Tablolar ve İlişkiler:

![Mimari](docs/images/eav_schema.png)

#### 1. DynamicTable
- TableId (PK)
- TableName
- IsActive
- CreatedDate
- ModifiedDate

#### 2. DynamicColumn
- ColumnId (PK)
- ColumnName
- ColumnType
- IsRequired
- Description

#### 3. DynamicValue
- ValueId (PK)
- TableId (FK)
- ColumnId (FK)
- StringValue
- IntegerValue
- DecimalValue
- DateValue

## 🚀 Kurulum

1. DB ve Tabloların oluşturulması:
- `scripts/create_database.sql`

2. Prosedürlerin oluşturulması:
- Tablo içindeki verileri getiren Prosedür: `/procedures/sp_GetDynamicData.sql`
- Tablo oluşturan Prosedür: `/procedures/sp_CreateDynamicTable.sql`
- Kolon oluşturan Prosedür: `/procedures/sp_AddDynamicColumn.sql`
- Veri ekleme Prosedürü: `/procedures/sp_AddDynamicValue.sql`
- Table Design Prosedürü: `/procedures/sp_GetTableDesign.sql`

3. (Opsiyonel) Örnek verileri yükleyin:
- `sample_data/sample_data.sql`
- `sample_data/02_categories.sql`
- `sample_data/03_customers.sql`
- `sample_data/04_orders.sql`
- `sample_data/05_additional_products.sql`

## 📝 Kullanım Örnekleri

### 1. Yeni Bir Tablo Oluşturma
```sql
EXEC sp_CreateDynamicTable 'Products'
```

### 2. Kolon Ekleme
```sql
EXEC sp_AddDynamicColumn 
    @TableName = 'Products',
    @ColumnName = 'ProductName',
    @ColumnType = 'STRING',
    @IsRequired = 1,
    @Description = 'Ürün adı'
```

### 3. Veri Ekleme
```sql
EXEC sp_AddDynamicValue 
    @TableName = 'Products',
    @ColumnName = 'ProductName',
    @RowId = 1,
    @Value = 'MacBook Pro'
```

### 4. Veri Sorgulama
```sql
EXEC sp_GetDynamicData 'Products'
EXEC sp_GetDynamicData @TableName = 'Products', @Where = "Price > 5000"
EXEC sp_GetDynamicData @TableName = 'Products', @Where = "Stock < 50", @OrderBy = "ProductName ASC"
EXEC sp_GetDynamicData @TableName = 'Products', @Where = "Brand = 'Apple' AND Price > 30000 AND Stock > 0", @OrderBy = "Price DESC, ProductName ASC"
```
![Veriler](docs/images/data.png)

### 5. Tablo Yapısını Görüntüleme
```sql
-- Products tablosunun yapısını getir
EXEC sp_GetTableDesign 'Products'

-- Customers tablosunun yapısını getir
EXEC sp_GetTableDesign 'Customers'
```
![Tablo Yapısı](docs/images/table_design.png)
## Örnek Senaryolar

### Ürünler Tablosu
```sql
-- Ürünler tablosunu oluştur
EXEC sp_CreateDynamicTable 'Products'

-- Kolonları ekle
EXEC sp_AddDynamicColumn 'Products', 'ProductName', 'STRING', 1, 'Ürün adı'
EXEC sp_AddDynamicColumn 'Products', 'Price', 'DECIMAL', 1, 'Fiyat'
EXEC sp_AddDynamicColumn 'Products', 'Stock', 'INTEGER', 1, 'Stok'

-- Veri ekle
EXEC sp_AddDynamicValue 'Products', 'ProductName', 1, 'iPhone 15 Pro'
EXEC sp_AddDynamicValue 'Products', 'Price', 1, 64999.99
EXEC sp_AddDynamicValue 'Products', 'Stock', 1, 100
```

### Müşteriler Tablosu
```sql
-- Müşteriler tablosunu oluştur
EXEC sp_CreateDynamicTable 'Customers'

-- Kolonları ekle
EXEC sp_AddDynamicColumn 'Customers', 'FirstName', 'STRING', 1, 'Ad'
EXEC sp_AddDynamicColumn 'Customers', 'LastName', 'STRING', 1, 'Soyad'
EXEC sp_AddDynamicColumn 'Customers', 'Email', 'STRING', 1, 'E-posta'
```

## Prosedürler

| Prosedür | Açıklama | Örnek Kullanım |
|----------|----------|----------------|
| sp_CreateDynamicTable | Yeni tablo oluşturur | `EXEC sp_CreateDynamicTable 'Products'` |
| sp_AddDynamicColumn | Kolonu ekler | `EXEC sp_AddDynamicColumn 'Products', 'Price', 'DECIMAL', 1` |
| sp_AddDynamicValue | Veri ekler | `EXEC sp_AddDynamicValue 'Products', 'Price', 1, 999.99` |
| sp_GetDynamicData | Verileri sorgular | `EXEC sp_GetDynamicData 'Products'` |
| sp_GetTableDesign | Tablo yapısını gösterir | `EXEC sp_GetTableDesign 'Products'` |

## Performans Optimizasyonları

1. **İndeksler**
- Her tabloda filtered index'ler kullanılmıştır
- IsActive = 1 koşulu için optimize edilmiştir
- Sık kullanılan sorgular için covering index'ler eklenmiştir

2. **Trigger'lar**
- UpdateDate alanları otomatik güncellenir
- SET NOCOUNT ON ile performans iyileştirilmiştir

3. **Constraints**
- Veri bütünlüğü için foreign key'ler eklenmiştir
- UNIQUE constraint'ler ile tekrar eden kayıtlar engellenmiştir

## Lisans
MIT License - Detaylar için [LICENSE](LICENSE) dosyasına bakın.
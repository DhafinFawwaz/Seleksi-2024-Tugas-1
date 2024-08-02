<h1 align="center"><br>
  Seleksi Laboratorium Basis Data (Tugas 1)<br>
  ETL Project<br>
</h1>

## Author
|          Nama             |      NIM      |
| ------------------------- | ------------- |
| Dhafin Fawwaz Ikramullah  |    13522084   |

## Deskripsi
<!-- - Deskripsi singkat mengenai data dan DBMS yang telah dibuat + mengapa kalian memilih topik tersebut -->
DBMS yang dibuat merupakan data dari game [Super Mario Bros](https://www.mariowiki.com/Super_Mario_Bros.) yang data-datanya merupakan hasil web scrapping. Topik ini dipilih karena akan cocok untuk dijadikan sebuah sistem basis data relasional dengan adanya hal-hal seperti character yang memiliki beberapa tipe, level yang berisikan item-item dengan jumlah tertentu, hubungan antara power up, playable character, dan form power up, dan sebagainya. Dengan adanya DBMS ini kita dapat dengan mudah memperoleh sebuah insight untuk game ini.

## Cara Menggunakan
<!-- - Cara menggunakan scraper yang telah dibuat dan menggunakan hasil output-nya -->
Pastikan untuk membuka folder `DataScraping/src` di terminal.

### Environtment Variables
Buat file `.env` di dalam folder `DataScraping/src` dengan isi dapat mengikuti examplenya pada file `DataScraping/src/.env.example.docker` atau `DataScraping/src/.env.example.localhost`. Atau langsung saja rename yang `DataScraping/src/.env.example.localhost` menjadi `DataScraping/src/.env`.


### Docker
Jika ingin menggunakan docker, jalankan ini
```
docker-compose up
```
Program akan secara otomatis melakukan web scraping dan menyimpannya di `json/data.json` serta memasukkan data-data ke basis data.

### Dump
jalankan command berikut untuk melakukan dump database
```
docker exec -it mariodb pg_dump mariodb > dump/dump.sql
```

### Restore Dump
jalankan command berikut untuk melakukan restore database
```
docker exec -i mariodb psql -U user -d mariodb < dump/dump.sql
```

### Manual
Jika tidak menggunakan docker, anda bisa mengikuti ini. Pastikan Python dan PostgreSQL terinstall.
Jalankan
```
python main.py
```
Dan akan otomatis web scraping dan memasukkan data ke basis data.

Jika hanya ingin web scraping saja maka jalankan
```
python scrape.py
```

Jika hanya ingin memasukkan ke database saja maka jalankan
```
python db.py
```

Contoh jika ingin menjalankan sebuah sql query dari code adalah sebagai berikut.
```
python db.py "SELECT * FROM item"
```


## Struktur File Hasil Scrapping
<!-- - Penjelasan struktur dari file JSON yang dihasilkan scraper -->

```
{
  character: {

    // array of playable characters, keunikannya punya relasi dengan form (misal mario menjadi fiery mario)
    pc: {
      name: string,
      description: string,
      image: {
        name: string,
        url: string,
        width: number,
        height: number
      },
      detail_url: string
    }[],


    // array of non playable characters
    npc: {
      name: string,
      description: string,
      image: {
        name: string,
        url: string,
        width: number,
        height: number
      },
      detail_url: string,
    }[],


    // array of enemies, keunikannya punya points
    enemy: {
      name: string,
      description: string,
      image: {
        name: string,
        url: string,
        width: number,
        height: number
      },
      detail_url: string,
      points: number,
    }[]
  },

  // array of obstacles yang ada di game
  obstacle: {
    name: string,
    description: string,
    image: {
      name: string,
      url: string,
      width: number,
      height: number
    },
    detail_url: string,
  }[],

  // array of item yang ada di game
  item: {
    name: string,
    description: string,
    image: {
      name: string,
      url: string,
      width: number,
      height: number
    },
    detail_url: string,
  }[],

  // array of power up yang ada di game, dapat membuat playable character berubah form
  power_up: {

    // ada yang N/A karena state saat playable character sedang tidak menggunakan power up
    name: null || string,
    description: string,

    // ada yang N/A karena state saat playable character sedang tidak menggunakan power up
    image: null || {
      name: string,
      url: string,
      width: number,
      height: number
    },

    // ada yang N/A karena state saat playable character sedang tidak menggunakan power up
    detail_url: null || string,

    // khusus untuk playable character bernama 'mario'
    mario_form: {
      name: string,
      image: {
        name: string,
        url: string,
        width: number,
        height: number
      },
      detail_url: string
    },

    // khusus untuk playable character bernama 'luigi'
    luigi_form: {
      name: string,
      image: {
        name: string,
        url: string,
        width: number,
        height: number
      },
      detail_url: string
    },
  }[],

  // array of object yang ada di game. tiap object memiliki typenya masing-masing, namun tidak ada keunikan dari tiap type, hanya sebagai penamaan saja. Lalu tiap 4 level memiliki image yang sama.
  object: {
    name: string,
    description: string,
    image: Image,
    detail_url: string,
    type: string
  }[],

  // array of reference yang ada pada game
  reference: {
    name: string,
    description: string,
    detail_url: string,
    type: string
  }[],

  // array of version yang ada pada game
  version: {
    year: string,
    description: string,
  }[],

  // array of level yang ada pada game. Tiap level memiliki item, power_up, enemies, dan obstacle dengan jumlah tertentu. Namun website sering tidak konsisten. Misal dalam item ada power_up. Kadang enemies dicampur dengan obstacle.
  level: {
    name: string,
    image: {
      name: string,
      url: string,
      width: number,
      height: number
    },
    detail_url: string,
    setting: string,
    course_map_image: {
      name: string,
      url: string,
      width: number,
      height: number
    },

    // Website sering tidak konsisten. Di dalam ini bisa ada power_up
    item: { 
      name: string;
      count: number;
      detail_url: string;
      description: string;
    }[],

    // Website sering tidak konsisten. Kadang enemies, kadang campuran enemies dan obstacle, kadang tidak ada sama sekali
    enemies: null || {
      name: string,
      count: number,
      detail_url: string
    }[],
    enemies_and_obstacle: null || {
      name: string,
      count: number,
      detail_url: string
    }[],
  }[]
}
```

## Struktur ERD dan diagram relasional RDBMS 
<!-- - Struktur ERD dan diagram relasional RDBMS -->
### Entity Relationship Diagram
<img src="Data Storing/design/Mario-DB-ER.drawio.png"/>

### Relational Diagram
<img src="Data Storing/design/Mario-DB-R.drawio.png"/>

## Translasi ERD Menjadi diagram relasional
<!-- - Penjelasan mengenai proses translasi ERD menjadi diagram relasional -->

## Screenshot
<!-- - Beberapa screenshot dari program yang dijalankan (image di-upload sesuai folder-folder yang tersedia, di README tinggal ditampilkan) -->


## Referensi
- Bahasa Pemrograman: Python
- Library:
    - asyncio: web request secara konkuren
    - BeautifulSoup: scrapping html website
    - psycopg2: driver database
- DBMS: PostgreSQL 
- URL Website yang discrape: 
    - https://www.mariowiki.com/Super_Mario_Bros
    - https://www.mariowiki.com/World_1-1_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_1-2_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_1-3_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_1-4_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_2-1_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_2-2_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_2-3_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_2-4_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_3-1_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_3-2_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_3-3_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_3-4_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_4-1_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_4-2_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_4-3_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_4-4_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_5-1_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_5-2_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_5-3_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_5-4_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_6-1_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_6-2_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_6-3_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_6-4_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_7-1_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_7-2_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_7-3_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_7-4_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_8-1_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_8-2_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_8-3_(Super_Mario_Bros.)
    - https://www.mariowiki.com/World_8-4_(Super_Mario_Bros.)
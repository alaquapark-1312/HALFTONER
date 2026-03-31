# HALFTONER

> Denbora errealeko halftone trama prozesadorea — WebGL + SVG parametrikoa, dependentziarik gabe, HTML fitxategi bakarra.

---

## Deskribapena

**HALFTONER** web aplikazio autonomoa da, halftone trama efektuak irudiei aplikatzeko. Prozesatzeak GPU-an egiten du lan WebGL bidez (aurrebistara 60fps-tan) eta exportatzeak SVG geometria parametrikoa sortzen du hasieratik, pantailaren bereizmenarekin loturarik ez duen fideltasun bektoriala bermatuz.

Instalaziorik gabe. Zerbitzaririk gabe. Dependentzia kanporik gabe. `.html` fitxategi bat.

---

## Efektuak

### DOTS — Trama klasikoa
Puntu zirkularrez osatutako sareta biratua. Puntuaren tamaina zelda bakoitzaren argitasunarekin proportzionala da.

| Parametroa | Tartea | Deskribapena |
|------------|--------|--------------|
| Zelda tamaina | 4–40 px | Tramaren maiztasuna |
| Angelua | 0–90° | Saretaren biraketa |
| Gamma | 0.3–3.0 | Kontrastea zuzentzea |
| Alderantzikatu | on/off | Irudia alderantzikatu tramatu aurretik |
| Tinta / Papera | kolorea | Tinta eta atzealdearen koloreak |

### CMYK — Kanalen banaketa
Lau trama gainjarrita inprimaketa-angelu estandarrekin: C 105°, M 75°, Y 90°, K 45°. Aurrebista moduan kanalak RGB-n konposatzen dira. Exportatzeak 4 SVG fitxategi bereizi sortzen ditu (puntu beltzak zuri gainean), ofseta edo serigrafia prestatzeko.

| Parametroa | Tartea | Deskribapena |
|------------|--------|--------------|
| Tamaina | 4–30 px | 4 banaketetan partekatutako maiztasuna |
| Dot Gain | 0–1 | Inprimatzerakoan puntuaren loditzea simulatzea |

### LINE — Lineala / Crosshatch
Zabal aldakorreko stripe horizontalak. Crosshatch aktibatuz bigarren trama gehitzen da angelu independentean.

| Parametroa | Tartea | Deskribapena |
|------------|--------|--------------|
| Maiztasuna | 3–30 px | Lerroen arteko tartea |
| 1. Angelua | 0–180° | Lerro nagusien norabidea |
| Crosshatch | on/off | Bigarren lerro-geruza |
| 2. Angelua | 0–180° | Crosshatch-en norabidea |
| Tinta / Papera | kolorea | Koloreak |

### FORM — Forma SDF-ak
DOTS berdina baina Signed Distance Field bidez definitutako forma geometrikoekin. Eskala uniformea argitasunarekin proportzionala.

| Parametroa | Tartea | Deskribapena |
|------------|--------|--------------|
| Tamaina | 4–44 px | Tramaren maiztasuna |
| Angelua | 0–90° | Saretaren biraketa |
| Forma | hautatzailea | hex · gurutze · diamante · izar |
| Tinta / Papera | kolorea | Koloreak |

### RISO — Risograph / Serigrafia
Risograph inprimagailuaren estetika simulatzen du: 2 edo 3 kolore-geruza misregistrazioarekin eta grainarekin. Koloreak askeak dira (color picker-ak). Konposaketa biderkatzailea da, inprimatze errealean bezala.

| Parametroa | Tartea | Deskribapena |
|------------|--------|--------------|
| Tamaina | 4–44 px | Puntuen maiztasuna |
| Koloreak | 2–3 | Kolore-geruzen kopurua |
| Desplazamendua px | 0–14 | Geruzen arteko misregistrazioa |
| Graina | 0–1 | Zarata grainaren intentsitatea |
| Kolorea 1/2/3 | kolorea | Kolore askea geruza bakoitzeko |

---

## Exportazioa

| Formatua | Deskribapena |
|----------|--------------|
| **PNG** | WebGL canvas-aren zuzeneko kaptura, kargatutako irudiaren bereizmenean |
| **SVG** | JS-n birkalkulatutako geometria bektorial parametrikoa (GLSL-arekin bat dator). Edozein tamainara eskalagarria |
| **CMYK SVG** | 4 SVG fitxategi independenteren deskarga: `C.svg`, `M.svg`, `Y.svg`, `K.svg`. Puntu beltzak fondo zurian, inprimatze-prestatzeko |

SVG exportazioak GLSL shader-en logika matematiko bera erabiltzen du (angelu berdinak, zelda berdinak, sampling bera), ez canvas-aren tracing-a.

---

## Erabilera

```
1. Ireki halftoner.html WebGL euskarria duen edozein nabigatzailetan
2. Irudi bat canvas-ean arrastatu, edo FITXATEGIA HAUTATU sakatu
3. Hautatu efektua fitxetan: DOTS · CMYK · LERROA · FORMA · RISO
4. Doitu parametroak denbora errealean slider-ekin
5. (Aukerakoa) ZATIKETA aktibatu originala eta prozesatua konparatzeko
6. Exportatu EXPORTATU botoiarekin
```

Onartutako irudi-formatuak: JPG, PNG, WEBP, GIF, nabigatzaileak onartzen duen edozein formatu. 2048px baino gehiagoko irudiak automatikoki tamainaz aldatzen dira kargatzean, GPU errendimendua mantentzeko.

---

## Arkitektura teknikoa

```
halftoner.html
├── WebGL pipeline-a
│   ├── Vertex shader (quad pantaila osoa)
│   └── Fragment shader-ak (efektu bakoitzeko bat)
│       ├── fs_dots.glsl
│       ├── fs_cmyk.glsl
│       ├── fs_linear.glsl
│       ├── fs_shapes.glsl
│       └── fs_riso.glsl
├── SVG sortzailea (JS hutsa)
│   ├── svgDots()      → <circle> elementuak
│   ├── svgCMYK()      → 4 × <circle> geruzak, beltz zurian
│   ├── svgLinear()    → <polygon> stripe-lerroak
│   ├── svgShapes()    → <polygon> / <path> SDF baliokideak
│   └── svgRiso()      → geruza bakoitzeko <g>, mix-blend-mode: multiply
└── Interfazea
    ├── Goibarra: igo · zatiketa · exportatu
    ├── Albobarra: efektu-fitxak + kontrol testuingurualak
    └── Infobar: neurriak · efektu-izena
```

**Stack:** WebGL 1.0, GLSL ES 1.0, JS vanilla ES6+. Tipografia Google Fonts bidez (Barlow Condensed + Azeret Mono). Framework-ik gabe, bundler-ik gabe, npm-ik gabe.

---

## Eskakizunak

- WebGL euskarria duen nabigatzailea (Chrome, Firefox, Safari, Edge — azken 5 urteetako edozein bertsio)
- Gidari eguneratuak dituen GPU (aplikazio osoa GPU-an exekutatzen da)
- Ez du Interneteko konexiorik behar fitxategia behin deskargatuta (tipografia-iturrien salbu)

---

## Diseinu-oharrak

- Pipeline-a **duala** da: WebGL elkarreraginerako 60fps-tan, JS parametrikoa kalitate handiko exportaziorako. Horrek canvas-aren tracing-a saihesten du eta bektore-irteera garbiak bermatzen ditu.
- **CMYK angeluak** ofseta inprimatzearen estandarra jarraitzen du (Cyan 105°, Magenta 75°, Yellow 90°, Black 45°) moiré minimizatzeko.
- **Dot gain**-ak `v += dg * v * (1 - v)` formularen bidez ofset-paperean puntuaren benetako loditzea modelatzen du.
- **Forma SDF-ak** distantzia-funtzio bera erabiltzen dute GLSL-n (aurrebista) eta JS poligono-sorkuntzan (exportazioa), koherentzia bisuala bermatuz.
- **Riso**-ak biderkatzeko konposaketa aplikatzen du inprimatze errealean bezala: `rgb *= mix(white, color, dot)`.

---

## Kredituak

**B_RR_K_** — AI garatzailea eta teknologista sortzailea, Euskal Herria.

---

*HALFTONER · fitxategi bakarra · dependentziarik gabe · WebGL + SVG*

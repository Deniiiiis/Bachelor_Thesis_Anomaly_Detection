============================================================
NOTEBOOK 01 – Preprocessing CIC-IDS2017
============================================================

Notebook 01 slúži na úvodné spracovanie datasetu CIC-IDS2017. Jeho cieľom je získať základný prehľad o dátach, kategóriách útokov, numerických atribútoch a koreláciách. Notebook neobsahuje žiadne modely – slúži výlučne na prípravu dát a vizuálnu analýzu, podobne ako pri datasete UNSW-NB15.

BUNKA 1 – Import knižníc
Načítava pandas, numpy, matplotlib a seaborn. Nastavuje úplné zobrazenie stĺpcov v DataFrame, aby boli všetky atribúty viditeľné.

BUNKA 2 – Načítanie dát
Načítava všetky CSV súbory z priečinka data/ (pondelok až piatok, jednotlivé scenáre útokov). Všetky súbory sú spojené pomocou pandas.concat do jedného DataFrame df. Názvy stĺpcov sa orezávajú od medzier. Výsledkom je jednotný dataset CIC-IDS2017.

BUNKA 3 – Zobrazenie základných informácií o datasete
Pomocou df.info() zobrazuje počet riadkov, počet stĺpcov, dátové typy atribútov a veľkosť datasetu v pamäti. Súčasne sa vypíše tvar (shape) datasetu.

BUNKA 4 – Štatistiky cieľových premenných
Identifikuje stĺpec Label, ktorý obsahuje hodnoty BENIGN a jednotlivé typy útokov. Zobrazuje rozdelenie Label (počet záznamov pre každú triedu). Zároveň vytvára pomocný binárny stĺpec is_attack, kde:
 - BENIGN → 0
 - iná hodnota → 1
a zobrazuje jeho distribúciu.

BUNKA 5 – Tabuľka počtov a percent podľa typu útoku
Vytvára prehľadnú tabuľku počtu záznamov pre každú kategóriu Label spolu s percentuálnym zastúpením v celom datasete. Táto tabuľka slúži ako základ pre porovnanie s UNSW-NB15 (rozdelenie typov útokov a benigných záznamov).

BUNKA 6 – Graf rozdelenia kategórií útokov
Zobrazuje barplot s počtom záznamov pre každú kategóriu Label (BENIGN + všetky útoky). Pomáha vizuálne pochopiť výraznú nerovnováhu medzi jednotlivými typmi útokov a veľkosť BENIGN triedy.

BUNKA 7 – Výber numerických atribútov
Identifikuje všetky numerické stĺpce pomocou select_dtypes(). Tento výber slúži pre výpočet korelácie a neskôr ako základ feature space pre modely v ďalších notebookoch.

BUNKA 8 – Korelačná matica numerických atribútov
Vytvára heatmapu korelačných koeficientov medzi numerickými atribútmi pomocou seaborn. Zobrazenie pomáha identifikovať redundantné a silno prepojené premenné, podobne ako pri datasete UNSW-NB15.

BUNKA 9 – Rozdelenie datasetu podľa typu útoku
Pomocou groupby(Label) vytvára mapovanie názvu útoku na príslušné riadky datasetu. Pre každú kategóriu sa vypisuje tvar (počet riadkov a stĺpcov). Táto časť pripravuje pôdu pre separáciu dát podľa tried útokov.

BUNKA 10 – Export subsetov dát podľa typu útoku
Ukladá každý typ útoku (vrátane BENIGN) do samostatného CSV súboru v priečinku data/attacks/. Každá kategória Label tak má svoj vlastný dataset pripravený pre detailnejšiu analýzu alebo experimenty s modelmi.

============================================================
NOTEBOOK 02 – Isolation Forest (detekcia anomálií) – CIC-IDS2017
============================================================

Notebook 02 je venovaný modelu Isolation Forest na datasete CIC-IDS2017. Umožňuje pracovať s celým zjednoteným datasetom (MODE="ALL") alebo len s vybranými kategóriami útokov (MODE="SELECTED") uloženými v data/attacks/. Trénovanie prebieha výlučne na normálnych (BENIGN) dátach, testovať sa môže na celom zvolenom datasete.

BUNKA 1 – Import knižníc
Importuje balíky potrebné pre načítanie dát, škálovanie, tréning Isolation Forestu a výpočet metrík: pandas, numpy, glob, sklearn, matplotlib a seaborn. Nastavuje základné prostredie pre analýzu.

BUNKA 2 – Konfigurácia režimu výberu dát
Definuje:
- RANDOM_STATE – náhodné semeno (reprodukovateľnosť),
- MODE – "ALL" alebo "SELECTED",
- PERCENTILE – percentil anomáliového skóre na trénovacích dátach pre určenie prahu,
- DATA_DIR – priečinok s pôvodnými CSV súbormi (data/),
- ATTACKS_DIR – priečinok so subsetmi podľa Label (data/attacks/),
- SELECTED_ATTACKS – zoznam názvov útokov pre MODE="SELECTED" (napr. ["DDoS", "PortScan"]).
Týmto sa dá flexibilne prepínať medzi plným datasetom a experimentmi s vybranými typmi útokov.

BUNKA 3 – Načítanie dát podľa MODE
MODE="ALL":
  - Načíta všetky CSV súbory v data/ a spojí ich do jedného DataFrame.
MODE="SELECTED":
  - Načíta BENIGN a vybrané kategórie útokov zo súborov v data/attacks/ (napr. BENIGN.csv, DDoS.csv, PortScan.csv).
Následne:
  - ošetrí názvy stĺpcov (strip),
  - identifikuje stĺpec Label,
  - vypíše tvar datasetu a distribúciu tried podľa Label.

BUNKA 4 – Výber numerických atribútov a binárneho labelu
- Vyberá numerické stĺpce pomocou select_dtypes(),
- nahrádza inf/-inf hodnoty NaN a odstraňuje riadky s NaN v numerických atribútoch,
- vytvára binárnu cieľovú premennú label_binary:
    BENIGN → 0 (normálna prevádzka)
    iné hodnoty Label → 1 (útok),
- tvorí maticu príznakov X (iba numerické atribúty) a vektor y (label_binary),
- vypisuje tvar dát a distribúciu tried po čistení.

BUNKA 5 – Rozdelenie na tréning a test + škálovanie
- Trénovacia množina:
    X_train_raw = všetky vzorky s label_binary = 0 (BENIGN),
- Testovacia množina:
    X_test_raw = celý zvolený dataset (normálne + útoky),
- Vykoná škálovanie pomocou StandardScaler:
    - scaler sa fituje na X_train_raw,
    - transformuje X_train_raw a X_test_raw na X_train a X_test,
- vypíše tvary matíc a distribúciu y_test (0 vs 1).

BUNKA 6 – Tréning Isolation Forest modelu
- Vytvára a trénuje IsolationForest s parametrami:
    n_estimators=200,
    contamination="auto",
    random_state=RANDOM_STATE,
    n_jobs=-1,
- Model sa učí normálne správanie (BENIGN) v priestore numerických príznakov.

BUNKA 7 – Výpočet skóre anomálií a určenie prahu
- Po trénovaní vypočíta anomáliové skóre pre trénovacie aj testovacie dáta pomocou -score_samples(),
- Určí prahovú hodnotu threshold ako PERCENTILE-percentil z anomáliových skóre trénovacích dát,
- Vypíše tvary skóre a zvolený prah.

BUNKA 8 – Výpočet metrík (precision, recall, F1)
- Prevádza skóre na binárne predikcie:
    skóre >= threshold → predikcia 1 (útok),
    skóre < threshold  → predikcia 0 (normál),
- Vypočíta metriky:
    Precision, Recall, F1-score (pozitívna trieda = útok, label 1),
- Vytvorí a vypíše confusion matrix (2x2 pre triedy [0, 1]).

BUNKA 9 – Confusion Matrix (vizualizácia)
- Pomocou seaborn.heatmap vizualizuje confusion matrix,
- Osi sú označené ako "Normal" a "Attack",
- Graf umožňuje intuitívne vnímať počty TP, FP, FN a TN pre nastavený prah.

BUNKA 10 – Precision-Recall krivka
- Vypočíta Precision-Recall krivku z y_test a skóre (scores_test),
- Vypočíta Average Precision (AP),
- Vykreslí PR-krivku (Recall na osi X, Precision na osi Y),
- PR-krivka je vhodná pre silne nevyvážené datasety, akým je CIC-IDS2017 (veľa BENIGN vs. menší počet útokov).

============================================================
NOTEBOOK 03 – One-Class SVM (detekcia anomálií) – CIC-IDS2017
============================================================

Notebook 03 implementuje model One-Class SVM nad datasetom CIC-IDS2017 s cieľom detegovať anomálie v sieťových reláciách. Je navrhnutý ako priamy ekvivalent notebooku 02 (Isolation Forest), aby sa výsledky jednotlivých modelov dali porovnávať jednotným spôsobom. Umožňuje pracovať s celým datasetom (MODE="ALL") alebo len s vybranými kategóriami útokov z priečinka data/attacks/ (MODE="SELECTED"). One-Class SVM je citlivý na škálovanie a veľkosť datasetu, preto notebook používa pipeline: výber numerických atribútov, škálovanie dát, tréning len na normálnych záznamoch, percentilové prahovanie, výpočet metrík a vizualizácie.

BUNKA 1 – Import knižníc
Importuje pandas, numpy, matplotlib, seaborn a knižnice zo sklearn potrebné pre One-Class SVM, škálovanie dát, výpočet metrík a vizualizácie. Zabezpečuje stabilné a jednotné prostredie na spracovanie dát.

BUNKA 2 – Konfigurácia režimu výberu dát (ALL / SELECTED)
Definuje:
 - RANDOM_STATE – náhodné semeno (reprodukovateľnosť),
 - MODE – "ALL" alebo "SELECTED",
 - PERCENTILE – percentil pre určenie prahu z anomáliových skóre,
 - DATA_DIR – priečinok s pôvodnými CSV súbormi (data/),
 - ATTACKS_DIR – priečinok so subsetmi podľa Label (data/attacks/),
 - SELECTED_ATTACKS – zoznam názvov útokov pre MODE="SELECTED" (napr. ["DDoS", "PortScan"]).
Pri MODE="ALL" sa použije celý zjednotený dataset, pri MODE="SELECTED" sa načítajú subsety z priečinka data/attacks/. Ide o rovnakú filozofiu ako v notebooku 02 (Isolation Forest).

BUNKA 3 – Načítanie dát podľa zvoleného režimu
MODE="ALL":
  - Načíta všetky CSV súbory v data/ (pondelok až piatok) a spojí ich do jedného DataFrame.
MODE="SELECTED":
  - Načíta BENIGN a vybrané kategórie útokov zo súborov v data/attacks/ (napr. BENIGN.csv, DDoS.csv, PortScan.csv).
Následne:
  - orezáva názvy stĺpcov od medzier,
  - identifikuje stĺpec Label,
  - vypíše tvar datasetu a distribúciu Label.

BUNKA 4 – Výber numerických atribútov a binárneho labelu
- Vyberie numerické stĺpce pomocou select_dtypes(),
- nahradí nekonečné hodnoty (inf, -inf) NaN a odstráni riadky s NaN v numerických atribútoch,
- vytvorí binárnu premennú label_binary:
    BENIGN → 0 (normálna prevádzka)
    iná hodnota Label → 1 (útok),
- vytvorí maticu príznakov X (numerické atribúty) a vektor y (label_binary),
- vypíše tvar dát a distribúciu tried po čistení.

BUNKA 5 – Rozdelenie dát na tréningové (len normálne) a testovacie (všetko) + škálovanie
- Trénovacia množina:
    X_train_raw = všetky vzorky s label_binary = 0 (BENIGN),
- Testovacia množina:
    X_test_raw = celý zvolený dataset (normálne + útoky),
- Vykoná škálovanie pomocou StandardScaler:
    - scaler sa fituje na X_train_raw,
    - transformuje X_train_raw a X_test_raw na X_train a X_test,
- vypisuje tvary trénovacích a testovacích dát a distribúciu y_test.

BUNKA 6 – Tréning One-Class SVM
- Vytvorí a trénuje model One-Class SVM s parametrami:
    kernel="rbf",
    gamma="scale",
    nu=0.05,
- Model sa učí hranicu normálneho správania iba z BENIGN dát a očakáva, že odchýlky od tejto hranice predstavujú potenciálne útoky.

BUNKA 7 – Výpočet skóre a percentilové prahovanie
- Pomocou decision_function vypočíta skóre pre všetky tréningové a testovacie vzorky,
- Skóre sa invertuje na anomáliové skóre tak, aby vyššia hodnota znamenala vyššiu pravdepodobnosť útoku (scores = -decision_function),
- Z anomáliových skóre trénovacích dát sa určí prah threshold ako PERCENTILE-percentil,
- vypisuje tvary skóre a zvolený prah.

BUNKA 8 – Výpočet metrík
- Prevedie skóre na binárnu predikciu:
    scores_test >= threshold → 1 (útok),
    scores_test < threshold  → 0 (normál),
- Vypočíta Precision, Recall a F1-score s pozitívnou triedou nastavenou na útok (label 1),
- Vytvorí a vytlačí confusion matrix (2x2 pre triedy [0, 1]).

BUNKA 9 – Confusion Matrix
- Vizualizuje confusion matrix pomocou seaborn.heatmap,
- Osi sú označené ako "Normal" a "Attack",
- Graf má rovnaký štýl ako v notebooku 02 (Isolation Forest), čo umožňuje priame vizuálne porovnanie medzi modelmi.

BUNKA 10 – Precision–Recall krivka
- Vypočíta Precision-Recall krivku zo skóre scores_test a skutočných y_test,
- Vypočíta Average Precision (AP),
- Vykreslí PR-krivku (Recall na osi X, Precision na osi Y),
- Graf je vhodný pre veľmi nevyvážené datasety, ako je CIC-IDS2017, a formátom aj interpretáciou je zladený s notebookom 02.

============================================================
NOTEBOOK 04 – Autoencoder (detekcia anomálií) – CIC-IDS2017
============================================================

Notebook 04 implementuje hlboký autoencoder nad datasetom CIC-IDS2017 s cieľom detegovať anomálie v sieťových reláciách. Je navrhnutý ako ekvivalent notebooku 04 pre UNSW-NB15, aby bolo možné priamo porovnať výsledky medzi datasetmi. Tréning prebieha výlučne na normálnych (BENIGN) záznamoch, testovanie na celom datasete. Detekcia anomálií je založená na rekonstrukčnej chybe a percentilovom prahovaní.

BUNKA 1 – Import knižníc
Importuje os, glob, numpy, pandas, matplotlib, seaborn, StandardScaler, metriky zo sklearn (precision_recall_fscore_support, precision_recall_curve, confusion_matrix) a TensorFlow/Keras (layers, Model). Nastavuje štýl grafov a vypíše verziu TensorFlow.

BUNKA 2 – Konfigurácia režimu a načítanie dát
Definuje:
- MODE – "ALL" alebo "SELECTED",
- SELECTED_GROUPS – zoznam kategórií Label pre MODE="SELECTED",
- DATA_DIR – priečinok s pôvodnými CSV súbormi (data/),
- ATTACKS_DIR – priečinok s subsetmi podľa Label (data/attacks/).

Pri MODE="ALL" sa načítajú všetky CSV súbory z data/ a spoja do jedného DataFrame. Pri MODE="SELECTED" sa načítajú konkrétne subsety z data/attacks/ podľa SELECTED_GROUPS (napr. BENIGN, DDoS, PortScan). Následne sa ošetria názvy stĺpcov, identifikuje sa stĺpec Label a vypíše sa tvar datasetu a rozdelenie tried.

BUNKA 3 – Príprava X a y, výber numerických atribútov
Vytvorí binárny vektor y, kde:
- BENIGN → 0 (normálna prevádzka),
- ostatné hodnoty Label → 1 (útok).

Vyberie všetky numerické atribúty pomocou select_dtypes() a z nich vytvorí maticu príznakov X. Počet numerických atribútov sa vypíše ako kontrola.

BUNKA 4 – Rozdelenie na trénovacie a testovacie dáta + škálovanie
Rozdelí dáta na:
- X_train – všetky normálne záznamy (y == 0),
- X_test – celý dataset (normálne + útoky),
- y_test – binárne labely pre celý dataset.

Následne použije StandardScaler:
- fit na X_train,
- transform X_train a X_test.

Škálované matice X_train a X_test slúžia ako vstup pre autoencoder. Vypíšu sa tvary a počet útokov v testovacej množine.

BUNKA 5 – Definícia architektúry autoencodera
Vytvorí hlboký plne prepojený autoencoder:
- vstupná dimenzia = počet numerických atribútov,
- encoder: Dense(64, relu) → Dense(32, relu) → Dense(16, relu),
- decoder: Dense(32, relu) → Dense(64, relu) → Dense(input_dim, linear),
- optimizer "adam", loss "mse".

Autoencoder sa skompiluje a vypíše sa model summary (počet parametrov a vrstiev).

BUNKA 6 – Tréning autoencodera
Trénuje autoencoder na X_train, pričom cieľ sú rovnaké dáta (rekonštrukcia vstupu). Parametre:
- EPOCHS = 25,
- BATCH_SIZE = 256,
- shuffle=True,
- validation_split=0.1.

Výstupom je objekt history s priebehom trénovacej a validačnej loss.

BUNKA 7 – Výpočet rekonstrukčnej chyby
Autoencoder predikuje rekonštruované vektory X_test_pred pre všetky testovacie vzorky. Rekonstrukčná chyba je definovaná ako priemerná kvadratická odchýlka na atribút:
  recon_error = mean((X_test - X_test_pred)^2, axis=1)
Vypíše sa tvar vektora chýb a základné minimum/maximum.

BUNKA 8 – Percentilové prahovanie
Určí sa percentil PERCENTILE (napr. 98) z rekonstrukčnej chyby:
  thr = percentile(recon_error, PERCENTILE)
Vzorky s chybou väčšou ako thr sú označené ako anomálie:
  y_pred_ae = (recon_error > thr).astype(int)

Vypíše sa hodnota PERCENTILE, prah thr a počet predpovedaných anomálií.

BUNKA 9 – Výpočet metrík
Pomocou precision_recall_fscore_support vypočíta:
- Precision,
- Recall,
- F1-score,
s pozitívnou triedou = útok (label 1). Následne sa vypočíta confusion matrix (2x2) a obe informácie sa vypíšu. Výsledky sú priamo porovnateľné s Isolation Forest a One-Class SVM.

BUNKA 10 – Confusion Matrix a Precision–Recall krivka
Najprv sa vizualizuje confusion matrix pomocou imshow:
- osi: True vs Predicted,
- popisy tried: "Normal" a "Attack",
- čísla v jednotlivých bunkách,
- farebná mapa "Blues".

============================================================
NOTEBOOK 05 – Variational Autoencoder (VAE) – CIC-IDS2017
============================================================

Notebook 05 implementuje Variational Autoencoder (VAE) nad datasetom CIC-IDS2017 s cieľom detegovať anomálie v sieťových reláciách. Je koncipovaný ako ekvivalent notebooku 05 pre UNSW-NB15, aby bolo možné porovnať výsledky VAE na oboch datasetoch. Tréning prebieha výlučne na normálnych (BENIGN) záznamoch, testovanie na celom datasete. Detekcia anomálií je založená na rekonstrukčnej chybe a percentilovom prahovaní.

BUNKA 1 – Import knižníc
Importuje os, glob, numpy, pandas, matplotlib, seaborn, StandardScaler, metriky zo sklearn (precision_recall_fscore_support, precision_recall_curve, confusion_matrix) a TensorFlow/Keras (layers, Model). Nastavuje štýl grafov a vypíše verziu TensorFlow.

BUNKA 2 – Konfigurácia režimu a načítanie dát
Definuje:
- MODE – "ALL" alebo "SELECTED",
- SELECTED_GROUPS – zoznam kategórií Label pre MODE="SELECTED",
- DATA_DIR – priečinok s pôvodnými CSV súbormi (data/),
- ATTACKS_DIR – priečinok s subsetmi podľa Label (data/attacks/).

Pri MODE="ALL" sa načítajú všetky CSV súbory z data/ a spoja do jedného DataFrame. Pri MODE="SELECTED" sa načítajú konkrétne subsety z data/attacks/ podľa SELECTED_GROUPS (napr. BENIGN, DDoS, PortScan). Po načítaní sa ošetria názvy stĺpcov a identifikuje sa stĺpec Label. Vypíše sa tvar datasetu a rozdelenie Label.

BUNKA 3 – Výber numerických atribútov a čistenie
Vytvorí binárny vektor y, kde:
- BENIGN → 0,
- ostatné Label → 1.

Vyberie všetky numerické stĺpce, voliteľne odstráni prípadný numerický stĺpec "label", nahradí nekonečné hodnoty (inf, -inf) hodnotami NaN a odstráni riadky s NaN v numerických atribútoch. Následne znovu vytvorí vektor y podľa Label a z numerických stĺpcov zostaví maticu príznakov X. Vypíše počet numerických atribútov, tvar dát a distribúciu tried.

BUNKA 4 – Rozdelenie na tréningové a testovacie dáta + škálovanie
Rozdelí dáta na:
- X_train – všetky normálne záznamy (y == 0),
- X_test – celý dataset (normálne + útoky),
- y_test – binárne labely pre celý dataset.

Pomocou StandardScaler sa fituje na X_train a transformuje X_train aj X_test. Výsledné matice X_train_scaled a X_test_scaled slúžia ako vstup pre VAE. Vypíšu sa tvary oboch množín a počet útokov v teste.

BUNKA 5 – Definícia Sampling vrstvy, encoder a decoder
Definuje vrstvu Sampling, ktorá realizuje reparametrizačný trik (z_mean, z_log_var → z). Encoder pozostáva z:
- vstupnej vrstvy (input_dim),
- Dense(64, relu),
- Dense(32, relu),
- Dense(latent_dim) pre z_mean,
- Dense(latent_dim) pre z_log_var,
- Sampling vrstvy.

Decoder:
- vstup latent_dim,
- Dense(32, relu),
- Dense(64, relu),
- Dense(input_dim, linear).

Vytvorí sa encoder aj decoder model a vypíše sa summary decodera.

BUNKA 6 – Trieda VAE a kompilácia
Definuje sa trieda VAE ako potomok keras.Model s:
- encoderom a decoderom,
- metódou train_step, ktorá počíta rekonstrukčnú stratu (MSE medzi vstupom a výstupom) a KL divergenciu medzi zakódovaným rozdelením a štandardným normálnym rozdelením.
Celková strata je súčet rekonstrukčnej straty a KL straty. Model sa kompiluje s Adam optimizerom.

BUNKA 7 – Tréning VAE
Model VAE sa trénuje na X_train_scaled s parametrami:
- EPOCHS = 30,
- BATCH_SIZE = 256,
- shuffle=True.
Výstupom je history s vývojom celkovej straty.

BUNKA 8 – Rekonštrukcia a výpočet rekonstrukčnej chyby
Encoder predikuje z_mean, z_log_var a z pre X_test_scaled, decoder z nich vytvorí rekonštruované vektory X_test_pred. Rekonstrukčná chyba je definovaná ako priemerná kvadratická odchýlka na príznak:
  recon_error = mean((X_test_scaled - X_test_pred)^2, axis=1)
Vypíše sa tvar vektora chýb a min/max hodnota.

BUNKA 9 – Percentilové prahovanie a metriky
Z rekonstrukčnej chyby sa určí PERCENTILE-percentil (napr. 98) ako prah:
  thr = percentile(recon_error, PERCENTILE)
Vzorky s chybou väčšou ako thr sú označené ako anomálie:
  y_pred_vae = (recon_error > thr).astype(int)

Pomocou precision_recall_fscore_support sa vypočítajú:
- Precision,
- Recall,
- F1-score
s pozitívnou triedou = útok (label 1). Všetky hodnoty sa vypíšu.

BUNKA 10 – Confusion Matrix a PR krivka
Vytvorí sa confusion matrix (y_test vs y_pred_vae) a vizualizuje sa pomocou seaborn.heatmap s popisom osí ("True", "Predicted") a tried ("Normal", "Attack"). Následne sa pomocou precision_recall_curve zrekonštruuje Precision–Recall krivka zo (y_test, recon_error) a vykreslí sa graf (Recall na osi X, Precision na osi Y) s mriežkou a názvom "PR Curve – VAE".



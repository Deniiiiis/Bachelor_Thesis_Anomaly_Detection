============================================================
README – UNSW-NB15 NOTEBOOKS
============================================================

Tento priečinok obsahuje sériu Jupyter notebookov, ktoré postupne spracúvajú, analyzujú a modelujú dataset UNSW-NB15. Každý notebook má presne definovanú úlohu a nadväzuje na predchádzajúci. Notebooky sú rozdelené tak, aby boli čo najprehľadnejšie, opakovane použiteľné a aby umožňovali testovať modely izolovane.

Nižšie sa nachádza stručný, široko formátovaný prehľad jednotlivých notebookov.

============================================================
NOTEBOOK 01 – Preprocessing UNSW-NB15
============================================================

Notebook 01 slúži na úvodné spracovanie datasetu UNSW-NB15. Jeho cieľom je získať základný prehľad o dátach, kategóriách útokov, numerických atribútoch a koreláciách. Notebook neobsahuje žiadne modely – slúži výlučne na prípravu dát pre ďalšie časti projektu.

BUNKA 1 – Import knižníc
Načítava pandas, numpy, matplotlib a seaborn. Nastavuje úplné zobrazenie stĺpcov v DataFrame, aby boli všetky atribúty viditeľné.

BUNKA 2 – Načítanie dát
Načítava hlavné dátové súbory z priečinka data/: UNSW_NB15_training-set.csv (hlavný dataset) a NUSW-NB15_features.csv (popis atribútov, encoding='latin1'). Výsledkom sú DataFrame df a DataFrame features.

BUNKA 3 – Zobrazenie základných informácií o datasete
Pomocou df.info() zobrazuje počet riadkov, počet stĺpcov, dátové typy atribútov a veľkosť datasetu v pamäti.

BUNKA 4 – Štatistiky cieľových premenných
Zobrazuje rozdelenie stĺpcov label (normál vs. útok) a attack_cat (kategórie útokov). Táto časť ukazuje nerovnováhu datasetu a dominanciu niektorých útokov.

BUNKA 5 – Tabuľka počtov a percent podľa typu útoku
Vytvára prehľadnú tabuľku počtu záznamov pre každú kategóriu attack_cat spolu s percentuálnym zastúpením v celom datasete.

BUNKA 6 – Graf rozdelenia kategórií útokov
Zobrazuje barplot s počtom záznamov pre každú kategóriu útoku. Pomáha vizuálne pochopiť výraznú nerovnováhu medzi jednotlivými typmi útokov.

BUNKA 7 – Výber numerických atribútov
Identifikuje všetky numerické stĺpce pomocou select_dtypes(). Tento výber slúži pre výpočet korelácie a prípravu vstupov pre modely.

BUNKA 8 – Korelačná matica numerických atribútov
Vytvára heatmapu korelačných koeficientov medzi numerickými atribútmi. Zobrazenie pomáha identifikovať redundantné a silno prepojené premenné.

BUNKA 9 – Rozdelenie datasetu podľa typu útoku
Pomocou groupby('attack_cat') zobrazuje počty záznamov pre každú kategóriu a pripravuje pôdu pre separáciu dát podľa tried útokov.

BUNKA 10 – Export subsetov dát podľa typu útoku
Ukladá každý typ útoku do samostatného CSV súboru v priečinku data/attacks/. Každá kategória tak má svoj vlastný dataset pripravený pre analýzu alebo tréning modelov.

============================================================
NOTEBOOK 02 – Isolation Forest (detekcia anomálií)
============================================================

Notebook 02 je venovaný modelu Isolation Forest, ktorý sa používa na detekciu anomálií. Notebook umožňuje pracovať s celým datasetom (MODE="ALL") alebo len s vybranými kategóriami útokov (MODE="SELECTED"). Tento notebook je prvým modulom projektu.

BUNKA 1 – Import knižníc
Importuje balíky potrebné pre načítanie dát, škálovanie, tréning Isolation Forestu a výpočet metrík: pandas, numpy, sklearn, matplotlib a seaborn.

BUNKA 2 – Konfigurácia režimu výberu dát
MODE = "ALL" načíta celý dataset pre tréning aj test. MODE = "SELECTED" umožní načítať len špecifické kategórie z data/attacks/. Poskytuje flexibilitu pri experimentovaní s izolovanými typmi útokov.

BUNKA 3 – Príprava X a y
Extrahuje numerické atribúty do X a cieľovú premennú (label) do y. Odstraňuje nenumerické stĺpce, aby model dostal konzistentný vstup.

BUNKA 4 – Rozdelenie dát na trénovacie a testovacie časti
Trénovanie prebieha iba na normálnych dátach (label=0). Testovací dataset pozostáva z celého zvoleného datasetu. Týmto spôsobom Isolation Forest trénuje iba normálne správanie.

BUNKA 5 – Škálovanie dát pomocou StandardScaler
Normalizuje numerické vstupy pre stabilnejší tréning modelu. Skalér sa trénuje na X_train a transformuje X_test.

BUNKA 6 – Tréning Isolation Forest modelu
Trénuje model s parametrami: n_estimators=200, contamination='auto', random_state=42, n_jobs=-1. Model sa učí identifikovať odchýlky v normálnom správaní.

BUNKA 7 – Výpočet skóre anomálií a určenie prahu
Získané skóre sa použije na výpočet percentilovej hranice (napr. 98%), ktorá definuje prah pre označenie vzorky ako anomálie.

BUNKA 8 – Výpočet metrík (precision, recall, F1)
Vyhodnocuje presnosť modelu porovnaním y_test a predikcií. Poskytuje presné meranie kvality anomálnej detekcie.

BUNKA 9 – Confusion Matrix
Zobrazuje maticu zámien pre normálne a anomálne záznamy. Vizualizácia pomáha odhaliť štruktúru chýb modelu.

BUNKA 10 – Precision-Recall krivka
Vytvára PR-krivku pre rôzne prahové hodnoty, čo je obzvlášť dôležité pri silne nevyvážených datasetoch, ako je UNSW-NB15.

============================================================
NOTEBOOK 03 – One-Class SVM (detekcia anomálií)
============================================================

Notebook 03 implementuje model One-Class SVM nad datasetom UNSW-NB15 s cieľom detegovať anomálie v sieťových reláciách. Tento notebook je navrhnutý ako priamy ekvivalent notebooku 02 (Isolation Forest), aby sa výsledky jednotlivých modelov dali porovnávať jednotným spôsobom. Notebook umožňuje pracovať s celým datasetom (MODE="ALL") alebo len s vybranými kategóriami útokov z priečinka data/attacks/ (MODE="SELECTED"). Model One-Class SVM je citlivý na škálovanie a veľkosť datasetu, preto notebook používa precízny pipeline: výber numerických atribútov, škálovanie dát, trénovanie len na normálnych záznamoch, percentilové prahovanie, výpočet metrík a vizualizácie.

BUNKA 1 – Import knižníc
Importuje pandas, numpy, matplotlib, seaborn a knižnice zo sklearn potrebné pre One-Class SVM, škálovanie dát, výpočet metrík a vykresľovanie. Zabezpečuje stabilné a jednotné prostredie na spracovanie dát.

BUNKA 2 – Konfigurácia režimu výberu dát (ALL / SELECTED)
Umožňuje vybrať, či sa model trénuje na celom datasete (MODE="ALL") alebo len na vybraných typoch útokov. Pri MODE="SELECTED" sa načítajú všetky CSV súbory zo zvolených kategórií z priečinka data/attacks/. Rovnaké nastavenie sa používa aj v notebooku 02, aby boli experimenty medzi modelmi porovnateľné.

BUNKA 3 – Načítanie dát podľa zvoleného režimu
Načítava buď celý dataset UNSW_NB15_training-set.csv alebo zvolené subsety. Z dát sa extrahujú iba numerické atribúty, pretože One-Class SVM nepracuje s kategóriami a je citlivý na rozsah vstupov.

BUNKA 4 – Rozdelenie dát na tréningové (len normálne) a testovacie (všetko)
Model One-Class SVM sa trénuje výlučne na záznamoch so štítkom label=0 (normálne správanie). Testovanie prebieha nad celým datasetom vrátane útokov. Týmto sa dosahuje realistická validácia detekcie anomálií.

BUNKA 5 – Škálovanie dát pomocou StandardScaler
Všetky numerické atribúty sa normalizujú. Skalér sa fituje iba na tréningových dátach a následne sa aplikuje na testovacie dáta. Je to kritické pre stabilitu a presnosť One-Class SVM.

BUNKA 6 – Tréning One-Class SVM
Trénuje sa model One-Class SVM s kernely RBF, gamma="scale" a nu=0.05 (odhad maximálneho podielu anomálií v tréningovej množine). Model sa učí hranicu normálneho správania a označuje odchýlky ako anomálie.

BUNKA 7 – Výpočet skóre a percentilové prahovanie
Pomocou decision_function sa vypočíta anomálne skóre pre všetky vzorky. Skóre sa premení tak, aby vyššie hodnoty znamenali vyššiu pravdepodobnosť útoku. Percentil (napr. 98 %) definuje prahovú hodnotu, nad ktorou je vzorka označená ako útok. Toto zjednocuje správanie notebooku 03 s notebookom 02.

BUNKA 8 – Výpočet metrík
Počíta sa Precision, Recall a F1-score pomocou y_test a y_pred. Model je hodnotený identicky ako v notebooku 02, čo umožňuje ich priame porovnanie.

BUNKA 9 – Confusion Matrix
Zobrazuje maticu zámien (TP, FP, FN, TN) pre normálne a anomálne záznamy. Vizualizácia je spracovaná rovnakým spôsobom ako v notebooku 02, aby sa zachovala konzistentnosť naprieč modelmi.

BUNKA 10 – Precision–Recall krivka
Vykresľuje PR-krivku na základe anomálneho skóre zo SVM. PR krivka je vhodná pre veľmi nevyvážené datasety, čo UNSW-NB15 bezpochyby je. Graf je identický formátom aj filozofiou ako v notebooku 02.

============================================================
NOTEBOOK 04 – Autoencoder (neurónová sieť pre detekciu anomálií)
============================================================

Notebook 04 implementuje Autoencoder – neurónovú sieť určenú na rekonštrukciu normálneho správania sieťovej prevádzky. Model sa trénuje iba na normálnych záznamoch (label=0) a zvyšok dát používa na detekciu anomálií na základe rekonštrukčnej chyby. Štruktúra notebooku je zladená s notebookmi 02 (Isolation Forest) a 03 (One-Class SVM), aby bolo možné výsledky modelov priamo porovnávať.

BUNKA 1 – Import knižníc
Importuje numpy, pandas, matplotlib, seaborn, nástroje zo sklearn na škálovanie a výpočet metrík, ako aj TensorFlow/Keras pre definíciu a trénovanie neurónovej siete.

BUNKA 2 – Výber režimu dát (ALL / SELECTED)
Umožňuje pracovať buď s celým datasetom (MODE="ALL"), alebo len s vybranými kategóriami útokov z priečinka data/attacks/ (MODE="SELECTED"). Režim je zdieľaný s ostatnými notebookmi, aby boli experimenty konzistentné.

BUNKA 3 – Príprava numerických atribútov a cieľovej premennej
Vyberajú sa numerické atribúty ako vstup X a cieľová premenná y zodpovedá stĺpcu label (0 = normálne, 1 = útok). Model pracuje čisto s numerickými dátami.

BUNKA 4 – Rozdelenie dát na tréningové a testovacie + škálovanie
Trénovacia množina X_train obsahuje iba normálne záznamy (label=0), testovacia množina X_test obsahuje všetky záznamy podľa zvoleného režimu. Všetky numerické atribúty sa škálujú pomocou StandardScaler, čo je dôležité pre stabilitu učenia neurónovej siete.

BUNKA 5 – Definícia architektúry Autoencoderu
Definuje sa plne prepojená sieť s niekoľkými vrstvami: vstupná vrstva, postupné zúženie (64 → 32 → latentných 16 neurónov) a následne rozšírenie späť na pôvodnú dimenziu. Výstup má lineárnu aktiváciu a model je trénovaný s MSE (mean squared error) ako stratovou funkciou.

BUNKA 6 – Tréning modelu
Autoencoder sa trénuje len na X_train_scaled, kde sa snaží rekonštruovať normálne dáta. Používa sa nastavený počet epoch a batch size, pričom validačná časť (validation_split) monitoruje priebeh učenia.

BUNKA 7 – Výpočet rekonštrukčnej chyby a prahovanie podľa percentilu
Model predikuje rekonštrukciu X_test_scaled. Pre každú vzorku sa spočíta rekonštrukčná chyba (MSE). Na základe zvoleného percentilu (napr. 98 %) sa určí prah, nad ktorým sú vzorky považované za anomálie. To je analogické s prahovaním skóre pri Isolation Foreste a One-Class SVM.

BUNKA 8 – Výpočet metrík
Na základe y_test a y_pred_ae sa počítajú metriky Precision, Recall a F1-score. Všetky modely v projekte sú hodnotené rovnakým spôsobom, čo umožňuje priamu komparáciu.

BUNKA 9 – Confusion Matrix
Zobrazuje maticu zámien medzi normálnymi a anomálnymi záznamami, v rovnakej podobe ako notebooky 02 a 03. To uľahčuje vizuálne porovnanie modelov.

BUNKA 10 – Precision–Recall krivka
Vykresľuje PR-krivku na základe rekonštrukčnej chyby. PR-krivka je vhodná pre nevyvážené datasety a umožňuje hodnotiť Autoencoder pri rôznych nastaveniach prahu.

============================================================
NOTEBOOK 05 – Variational Autoencoder (VAE)
============================================================

Notebook 05 implementuje Variational Autoencoder (VAE), pokročilú neurónovú sieť určenú na modelovanie pravdepodobnostného latentného priestoru. Tento prístup umožňuje zachytiť komplexnejšie vzorce normálneho správania sieťovej prevádzky. Model sa trénuje výhradne na normálnych záznamoch (label=0) a rekonštrukčná chyba sa následne využíva na detekciu anomálií. Štruktúra notebooku je zladená s predchádzajúcimi modelmi, aby boli výsledky priamo porovnateľné.

BUNKA 1 – Import knižníc  
Importuje numpy, pandas, matplotlib, seaborn, škálovacie nástroje zo sklearn a TensorFlow/Keras na definíciu architektúry VAE a jeho tréning.

BUNKA 2 – Výber režimu dát (ALL / SELECTED)  
Rovnaký mechanizmus ako v predchádzajúcich notebookoch. Režim MODE="ALL" načíta celý dataset UNSW_NB15_training-set, režim MODE="SELECTED" načíta len vybrané kategórie útokov z priečinka data/attacks/. Umožňuje experimentovať so špecifickými typmi anomálií.

BUNKA 3 – Príprava numerických atribútov a cieľovej premennej  
Vyberajú sa numerické stĺpce datasetu ako vstup X. Premenná y sa načítava zo stĺpca label (0 = normálne, 1 = útok). VAE pracuje výhradne s numerickými atribútmi, ktoré sú vhodné pre rekonštrukčné učenie.

BUNKA 4 – Rozdelenie dát na tréningové a testovacie + škálovanie  
Trénovacia množina X_train obsahuje len normálne dáta. Testovacia množina X_test obsahuje všetky vzorky podľa režimu. Numerické atribúty sa škálujú pomocou StandardScaler, čo zabezpečuje stabilný tréning neurónovej siete aj v latentnom priestore.

BUNKA 5 – Definícia architektúry VAE  
Definuje sa encoder, sampling vrstva a decoder. Encoder produkuje parametre latentného priestoru (z_mean a z_log_var). Sampling vrstva pomocou reparametrizačného triku vytvorí latentný vektor z. Decoder rekonštruuje pôvodný vstup z latentného priestoru. Architektúra je plne prepojená pomocou Dense vrstiev.

BUNKA 6 – Implementácia VAE modelu  
VAE sa implementuje ako vlastná trieda s prepisom metódy train_step. Celková strata pozostáva z rekonstrukčnej chyby (MSE) a KL divergencie, ktorá regularizuje latentný priestor. Tento prístup umožňuje VAE generovať realistické latentné reprezentácie normálnej prevádzky.

BUNKA 7 – Tréning modelu  
Model sa trénuje nesupervidovaným spôsobom na X_train_scaled. Používa sa Adam optimizér a vhodný počet epoch. Počas tréningu sa optimalizuje rekonštrukcia aj štruktúra latentného priestoru.

BUNKA 8 – Výpočet rekonštrukčnej chyby a určenie prahovej hodnoty  
VAE generuje rekonštrukciu pre všetky prvky X_test_scaled. Rekonštrukčná chyba (MSE) sa využíva na detekciu anomálií. Prahová hodnota sa určuje pomocou percentilu (napr. 98 %), rovnako ako v predchádzajúcich notebookoch. Vzorky s vyššou chybou sú označené ako anomálne.

BUNKA 9 – Výpočet metrík  
Na základe porovnania y_test a predikcií modelu (y_pred_vae) sa počítajú základné metriky: Precision, Recall a F1-score. Hodnotenie je konzistentné so všetkými ostatnými modelmi v projekte.

BUNKA 10 – Confusion Matrix  
Notebook vykresľuje maticu zámien na porovnanie správnosti klasifikácie normálnych a anomálnych vzoriek. Štýl zobrazenia je zhodný s predchádzajúcimi notebookmi.

BUNKA 11 – Precision–Recall krivka  
Vykresľuje PR-krivku na základe rekonštrukčnej chyby, čo umožňuje analyzovať správanie VAE pri rôznych prahoch. PR-krivka je vhodná pri práci s nevyváženým datasetom, typickým pre úlohy detekcie útokov.

============================================================
KONIEC README
============================================================

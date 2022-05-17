# Introducere
Acest git-repository contine solutia propusa pentru rezolvarea problemei segmentarii semantice folosind imagini CT. 

Pasii urmariti pentru rezolvarea problemei sunt urmatorii:
- procesarea imaginilor CT
- construirea si antrenarea modelului Deep Learning
- testarea modelului

# Procesarea imaginilor CT
Procesarea imaginilor reprezinta primul pas catre rezolvarea problemei. Setul de date contine volumele CT si segmentarile ground-truth corespunzatoare. 
Fiecare volum de imagine CT are o dimensiune diferita, dar fiecare imagine din volum are o dimensiune de 512x512. 
Notebook-ul "data_processing.ipynb" contine etapele de procesare a setului de date.
Procesarea volumelor si segmentarilor a fost realizata intr-un mod putin diferit, dupa cum urmeaza:
- fiecare imagine din volumele CT (atat din setul de training cat si din cel de test) a fost convertit intr-o variabila de tip numpy.array folosind libraria nibabel. Matricile numpy au fost apoi normalizate, rotite cu 90 de grade, si redimensionate la dimensiunea de 128x128. Deoarece fiecare volum continea un numar diferit de imagini CT, numarul imaginilor a fost redus la 64. Dupa acest prim pas de procesare, valorile fiecarui pixel din "imaginile" din matricile rezultate au fost impartite pe 3 canale (i.e. RGB). Noile matrici rezultate au astfel dimensiunile de (6592, 128, 128, 3), pentru setul de train, si (1792, 128, 128, 3), pentru setul de test.
- fiecare imagine din seturile de ground-truth au trecut printr-un pipeline de procesare similar. Fiecare imagine a fost convertita intr-o variabila de tip numpy.array, matricile in schimb nu au mai fost normalizate, ci doar rotite si redimensionate la 128x128. Volumele de ground-truth au fost micsorate la 64 de imagini/volum, iar matricile nu au fost "sparte" pe 3 canale, ci doar s-a mai adaugat o dimensiune in plus. Noile dimensiuni ale matricilor sunt (6592, 128, 128, 1), pentru setul de train, si (1792, 128, 128, 1), pentru setul de test. Deoarece redimensionarea imaginilor a produs schimbari / aproximari in randul valorilor de ground truth (i.e. 0, 1, 2), noile valori din matrici au fost aproximate.

Variabilele rezultate (i.e. X_train, y_train, X_test, y_test) au fost apoi salvate local pentru a evita repetarea procesarii imaginilor.

# Construirea si antrenarea modelului
Modelul ales pentru rezolvarea problemei este U-Net. Arhitectura U-Net a castigat detasat "Cell Tracking Challenge" in cadrul International Symposium on Biomedical Imaging (ISBI), in anul 2015. Mai mult de atat, arhitectura este indeajuns de bine realizata incat rularea acesteia pe niste placi GPU de comert se poate efectua rapid. Lucrarea stiintifica originala care prezinta aceasta arhitectura poate fi gasita aici: https://doi.org/10.48550/arXiv.1505.04597

Modelul a fost antrenat folosind urmatorii parametri:
- epochs = 35
- validation_split = 0.1
- optimizer = 'adam'
- loss = 'sparse_categorical_crossentropy'

Dupa finalizarea procesului de antrenare, modelul obtine:
- loss = 0.0183
- accuracy = 0.9943

Modelul reuseste sa identifice ficatul corect in fiecare intre imaginile care alcatuiesc volumele din setul de test. Acesta in schimb are dificultati in conturarea completa a ficatului si in identificarea tumorilor prezente pe ficat.
Acuratetea foarte ridicata a modelului este inselatoare. Matricile numpy pe care trebuie sa le produca modelul contin, majoritar, valori egale cu 0. Valorile diferite de 0 sunt reprezentate de zonele unde se afla ficatul/tumorile, acestea avand un numar relativ mic. 
Prin urmare, modelul ar fi reusit sa obtina o acuratete ridicata chiar si in cazul in care ar fi produs matrici care sa contina doar valori egale cu 0.

# Propuneri pentru imbunatatirea modelului
Arhitectura U-Net a fost initial conceputa pentru a fi folosita pe imagini de dimensiuni 512x512. 
Decizia de micsorare a calitatii imaginilor prin redimensionarea la 128x128 si de folosire doar a 64 imagini/volum a fost luata din considerente de spatiu. 
Modelul a fost antrenat folosind Google Colab, ceea ce a insemnat ca variabilele procesate au trebuit incarcate pe Google Drive, acesta avand o limita de spatiu de 15 GB.


Una dintre propunerile imbunatatirii modelului este folosirea imaginilor la dimensiunile originale si folosirea tuturor imaginilor din volumelele initiale. 
Unul dintre motivele pentru care modelul are dificultati in identificarea tumorilor este ca acestea apar in foarte putine imagini. Folosirea tuturor imaginilor din volum va ajuta la rezolvarea acestei probleme.

O alta propunere ar fi folosirea unei variatii ale Arhitecturii U-Net. 
Arhitectura a fost publicata initial in 2015, iar dupa publicarea acesteia au aparut numeroase variatii si implementari care imbunatateau performanta acesteia (e.g. Cu-Net, TransResUnet, HistNet etc.).


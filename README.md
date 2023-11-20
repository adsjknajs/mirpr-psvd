# Recunoasterea pieselor si tablelor de sah

Aceasta aplicatie are scopul de a oferi posibilitatea de a recunoaste configuratia unei table de sah dintr-o poza.

Pentru a face acest lucru, trebuie rezolvate urmatoarele doua probleme:
1) Gasirea fiecarui patrat de pe tabla de sah
2) Clasificarea piesei care se afla pe fiecare patrat detectat la pasul anterior

Initial, folosind informatii dintr-un articol ce abordeaza aceasta tema [1], exista o prima metoda de a rezolva aceasta problema:
1) Folosind algoritmi de Computer Vision, detectam liniile din imagine
2) Avand liniile verticale si orizontale, calculam coordonatele intersectiilor
3) Avand intersectiile, putem gasi fiecare patrat din imagine, si putem imparti imaginea originale in sub-imagini mai mici, care contin un anumit patrat si piesa de pe acest patrat

O poza din acest articol ce prezinta acest proces:

![image description](https://miro.medium.com/v2/resize:fit:720/format:webp/1*srfAGMD1Mk_HsoLzKVCDhg.png)

Imaginea initiala trebuie transformata in alb-negru, blurata, iar apoi se aplica algoritmul 'Canny Edge Detection', pentru ca imaginea sa contina doar linii in format alb-negru.
Dupa obtinerea acestei imagini, se foloseste algoritmul 'Hough Transform' pentru a detecta coordonatele liniilor.

Pentru aplicatia aceasta s-a incercat folosirea acestui proces, dar cu unele schimbari:
1) Blurarea imaginilor cu diferiti parametri, deoarece nu exista un singur set de valori care va facilita detectarea liniilor intr-un mod corect
2) Filtrarea liniilor care sunt prea aproape una de alta

O problema intampinata a fost detectoarea imprecisa a liniilor, deoarece caracteristicile imaginilor erau foarte diferite pentru a obtine rezultate consistente.

Asadar, trebuie o noua metoda de a detecta patratele, cu informatiile dintr-un alt articol [2], a fost gasita o noua abordare:
1) Gasirea colturilor ale tablei de sah folosind retele neuronale
2) Transformarea perspectivei, folsind coordonate colturilor

Pentru a rezolva aceasta problema, a fost folosit modelul preantrenat 'YOLOv8s', antrenat apoi cu noi poze ce contin table de sah.

![image](https://github.com/adsjknajs/mirpr-psvd/assets/151509326/5d146925-363a-4295-b534-5abedeedecba)

Folosind platforma Roboflow, au fost colectate 107 imagini, si au fost notate pentru fiecare poza coordontele folosind o interfata grafica:
Exemplu:

![Screenshot_8](https://github.com/adsjknajs/mirpr-psvd/assets/151509326/cc8bffb9-fd37-46d2-903d-de0a7354c907)

Platforma ofera posibilitatea de a augmenta imaginile existente, asadar reies un numar mai mari de imagini la final. Acestea sunt datele training set-ului folosit pentru gasirea colturilor:

![asa](https://github.com/adsjknajs/mirpr-psvd/assets/151509326/ca48c61f-afca-4405-b4f4-8a28304daa82)

Modelul folosit pentru gasirea colturilor a fost apoi antrenat folosind acest set de date, cu batch_size = 8 si nr_epoci = 50.

Rezultate dupa antrenare:

![results](https://github.com/adsjknajs/mirpr-psvd/assets/151509326/dfce668a-4fe0-42a5-ad92-e505a5d623c3)

(Clasele ar fi trebuit sa fie doar '1' si 'background', dintr-o eroare apar si alte 3 clase ce au fost folosite initial dar scoase, dar acestea nu afecteaza cu nimic)

![confusion_matrix](https://github.com/adsjknajs/mirpr-psvd/assets/151509326/b6f2f7df-9ca7-447e-8aef-782943408457)

Dupa gasirea coordonatelor colturilor, tot in acest articol sunt oferite informatii despre cum am putea ordona aceste coordonate, asadar prima sa fie coltul din stanga sus, al doilea cel din dreapta sus, etc. folosind niste observatii cum ar fi: primul colt are suma minima dintre coordonate, ultimul are suma maxima, al doilea are diferenta minima, al treilea diferenta maxima.

Asadar, dupa acest intreg proces, avem coordonatele colturilor, din care putem transforma perspectiva imaginii pentru a avea doar tabla de sah in poza.
Exemplu:

Inainte de transformare:

![aa1](https://github.com/adsjknajs/mirpr-psvd/assets/151509326/1552f94e-a1c3-4b16-a601-5e8bc6d43b04)

Dupa transformare:

![aa2](https://github.com/adsjknajs/mirpr-psvd/assets/151509326/1c199c44-bc33-4781-82f2-38ef4fcb3ad1)

Avand aceasta poza noua, putem imparti inaltimea si latimea pozei la 8, din care rezulta fiecare patrat. De acolo, rezulta sub-imaginea ce contine piesa de pe patratul respectiv.

Problema 1) a fost rezolvata. Acum problema 2) trebuie rezolvata, in care clasificam piesele din fiecare sub-imagine.
A fost folosit modelul pre-antrenat 'YOLOv8n', si reantrenat cu o arhiva de poze [3] de pe platforma Roboflow:

-1500 imagini training

-336 imagini test

-740 imagini validare

Pentru acest antrenament, au fost folosit batch_size = 16 si nr_epoci = 80. Exista 12 clase, fiecare reprezentand tipul de piesa + culoarea.

Un exemplu:

![dsadsa](https://github.com/adsjknajs/mirpr-psvd/assets/151509326/ed9d5af3-8edc-448b-824b-245dc59a7eae)

Rezultate dupa antrenare:

![results](https://github.com/adsjknajs/mirpr-psvd/assets/151509326/b16c564b-b9d4-4b1e-98fc-f8c69ee08e0d)

![confusion_matrix](https://github.com/adsjknajs/mirpr-psvd/assets/151509326/6d170454-cde2-475d-bc64-96a52119c77d)

Referinte:

[1] https://towardsdatascience.com/board-game-image-recognition-using-neural-networks-116fc876dafa

[2] https://blog.roboflow.com/chess-boards/

[3] https://universe.roboflow.com/teste-6c5pa/fd-bc9g6

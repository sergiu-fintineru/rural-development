Am folosit datele libere puse la dispoziție de către AFIR (https://www.afir.ro/rapoarte/beneficiari-de-fonduri-europene/date-deschise/, https://opendata.afir.info) în ceea ce privește proiectele din programarea din perioada 2007-2013 a Programului Național Dezvoltare Rurală, cât și pe cele din perioada următoare, respectiv 2014-2020.

Rolul articolului este de a arată utilitatea datelor deschise în domeniul agriculturii și al dezvoltării rurale și modalitățile prin care acestea pot genera cunoștințe noi care pot fi utilizate în luarea deciziilor.

De asemenea o cerere crescută pentru acest tip de date libere de către personalul academic, dar nu numai, poate duce la o suplimentare a acestor din partea instituțiilor de stat, APIA, AFIR, Ministerul Agriculturii și Dezvoltării Rurale.

Fișierul csv. pentru prima programare, 2007-2013, are 16 coloane ['entityid', 'nume', 'tip_organizare', 'cod_fiscal', 'roapia', 'siruta', 'localitate', 'adresa', 'codproiect', 'valoare_contractata', 'valoare_platita', 'oj', 'cr', 'longitude', 'latitude', 'amplasarelocalitate']și 86.751 de rânduri care conțin datele despre fiecare proiect în parte, cele mai pertinente pentru analiză fiind coloanele cu numele persoanelor sau societăților comerciale, al tipului de organizare (AG, PFA)

Pentru procesarea fișierului csv. am folosit Jupyter Notebook local, aplicația web interactivă și open-source, care îmi permite să manipulez și să prelucrez fișierul cu peste 80.000 foarte rapid prin intermediul limbajului Python.

Deoarece din datele AFIR nu includeau și o coloană cu județele din care făceau parte proiectele m-am folosit de codul SIRUTA și de baza de date de aici https://github.com/strainu/SIRUTA/blob/master/siruta.csv; iar codul folosit este următorul.

import matplotlib.pyplot as plt
import numpy as np

# 1. Ne asigurăm că valorile sunt numerice (opțional, dar previne erorile dacă sumele sunt citite ca text)
df['valoare_contractata'] = pd.to_numeric(df['valoare_contractata'], errors='coerce').fillna(0)
df['valoare_platita'] = pd.to_numeric(df['valoare_platita'], errors='coerce').fillna(0)

# 2. Grupăm datele pe județe (numărăm rândurile pentru numărul de proiecte și însumăm banii)
df_judete = df.groupby('judet').agg(
    numar_proiecte=('judet', 'size'),
    total_contractat=('valoare_contractata', 'sum'),
    total_platit=('valoare_platita', 'sum')
).reset_index()

# Sortăm descrescător ca să arate bine vizual (de la județul cu cei mai mulți bani la cel cu cei mai puțini)
df_judete = df_judete.sort_values(by='total_contractat', ascending=False)

# 3. Desenăm graficul
fig, ax1 = plt.subplots(figsize=(18, 8)) # O lățime mai mare ca să încapă toate cele 42 de județe

# Setăm lățimea barelor și poziția lor pe axa X
x = np.arange(len(df_judete['judet']))
width = 0.35

# Desenăm barele pentru valorile financiare (pe axa din Stânga)
bara_contractat = ax1.bar(x - width/2, df_judete['total_contractat'], width, label='Valoare Contractată', color='#1f77b4')
bara_platit = ax1.bar(x + width/2, df_judete['total_platit'], width, label='Valoare Plătită', color='#2ca02c')

ax1.set_xlabel('Județ', fontsize=12)
ax1.set_ylabel('Sume (RON)', fontsize=12)
ax1.set_title('Proiecte pe Județe: Valori financiare vs. Număr de proiecte', fontsize=16)
ax1.set_xticks(x)
ax1.set_xticklabels(df_judete['judet'], rotation=45, ha='right')
ax1.legend(loc='upper left')

# 4. Adăugăm a doua axă Y (pe Dreapta) pentru numărul de proiecte
ax2 = ax1.twinx()
linie_proiecte = ax2.plot(x, df_judete['numar_proiecte'], color='red', marker='o', linewidth=2, label='Număr Proiecte')

ax2.set_ylabel('Număr de Proiecte', fontsize=12, color='red')
ax2.tick_params(axis='y', labelcolor='red')
ax2.legend(loc='upper right')

# Adăugăm un grid ușor de citit
ax1.grid(axis='y', linestyle='--', alpha=0.4)

# Afișăm graficul curat
plt.tight_layout()
plt.show()

Graficul obținut este acesta.
Conform datelor din tabel un total al valorii contractate este de 5,012,452,124.39 EUR, iar totalul valorii plătite este de  4,758,639,831.71 EUR; alocarea totală pentru PNDR din perioada 2007-2013 a fost de aproximativ 9,3 miliarde EUR, din care aproximativ 8,1 miliarde fonduri europene FEADR, iar restul cofinanțare națională.

Tabelul de mai sus însumează aproximativ 5 miliarde EUR ceea ce indică faptul că baza de date conține strict măsurile de investiții, adică proiecte propriu-zise cu contract de finanțare: precum modernizări de ferme, instalarea tinerilor fermieri, pensiuni agroturistice, renovarea satelor, prin Măsura 322.

Diferența dintre programarea totală și ce avem în tabel, aproximativ cei 4 miliarde EUR, sunt banii care au mers către subvenții anuale și plăți compensatorii, precum Măsura 214, plăți de agromediu, zone montane defavorizate. Plata pentru acestea se face la hectar, nu se semnează ca proiecte individuale de investiții, și de aceea nu vor apărea în seturi de date.

Raportul dintre valoarea plătită, 4,75 miliarde, și cea contractată 5,01 miliarde, este de 95%, ceea ce corespunde datelor pe care le avem despre PNDR 2007-2013, campionul absolut al României la fonduri europene, singurul program operațional major din acel exercițiu financiar ce a depășit rata de absorbție de 90%.


Ambele seturi de date
Analiză pe județe și apoi la nivelul localităților. 
Venitul mediu per locuitor îl comparăm cu venitul la nivelul județului și venitul generat de PNDR
Axa 1 – agricultura, axa 2 – mediu, axa 3 – non agricol, axa 4 – leader


Metodologia detaliată de curățare a datelor AFIR poate fi consultată în [scriptul de procesare a datelor](notebooks/data_pndr.ipynb).
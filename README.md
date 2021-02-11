# Qualité des données
## Notre équipe
Notre équipe est composé des collaborateurs suivants :
- CHESNOUARD Dylan
- MARTINEAU Adrien
- MORILLE Manon
- TOUSSAINT Angéline

## Contexte
Travaillant sur la qualité des données, nous prenons un échantillon de données erronées et devons corriger ses erreurs pour que les données soient les plus fiables possibles. Pour cela nous travaillerons avec du Python. 

## Outil utilisés
Afin de répondre à la problématique, nous avons utilisé les outils suivants :
- Python (pandas, numpy, matplotlib, openpyxl)
- Jupyter sous Docker
- Github
- Discord

## Analyse du sujet
Nous disposons d’un échantillon de données climatique. Nous avons créé un dataFrame afin de récupérer les informations pertinentes. Après les avoir corrigés et analysés, nous essaierons de déterminer le climat ainsi que la capitale européenne dont sont issues ces données. 

## Notre démarche
### Importation des données
```python
%matplotlib notebook

import sys
import pandas as pd
import numpy as np
from matplotlib import pyplot as plt
!{sys.executable} -m pip install openpyxl
from openpyxl import load_workbook
from openpyxl import Workbook
```

### Création du dataframe
```python
df = pd.read_csv('clean_climat.csv', dtype={'month': np.str}, delimiter=';')
df.index += 1
df = df.drop(df.columns[[0]], axis=1)
```

### Récupération des informations principales du dataframe
```python
print(df.head(10))
print("Moyenne par mois :")
print(np.mean(df, axis=0))
print("Ecart type :")
print(np.std(df, axis=0))
print("Min :")
print(np.min(df, axis=0))
print("Max :")
print(np.max(df, axis=0))
```

### Génération du graphique par mois
```python
x = np.array(df.index)

fig = plt.figure()
ax = fig.add_subplot(111)
for month in df.columns:
    y = np.array(df[month])
    ax.plot(x, y, label=month)
plt.title('Températures par mois')
plt.xlabel('Jours')
plt.ylabel('Températures')
ax.legend(loc='right', bbox_to_anchor=(1.3, 0.5), frameon=True)
fig.savefig('month_graph.png', bbox_inches='tight')
```
![Month graph](https://github.com/WaZeR-Adrien/qualite-des-donnees/blob/main/month_graph.png)


### Génération du graphique étalé sur l’année
```python
temperatures = []
for month in df.columns:
    temperatures.extend(df[month])

y = np.array(temperatures)
y = y[~np.isnan(y)]
x = np.array(range(1, (len(y) + 1)))

fig = plt.figure()
ax = fig.add_subplot(111)
ax.plot(x, y)
plt.title('Températures de l\'année')
plt.xlabel('Jours')
plt.ylabel('Températures')
fig.savefig('year_graph.png')
```
![Year graph](https://github.com/WaZeR-Adrien/qualite-des-donnees/blob/main/year_graph.png)


### Lecture du fichier .XLSX contenant des erreurs
```python
wb = load_workbook(filename='Climat.xlsx')
sheet = wb[wb.sheetnames[1]]

months = []

df_bis = pd.DataFrame()

for i in range(4, 16, 1):
    temp = []
    for j in range(5, 36, 1):
        temp.append(sheet.cell(row=j, column=i).value)
    df_bis[sheet.cell(row=3, column=i).value] = temp

df_bis.index += 1

print(df_bis)
```

### Modification du fichier pour ne plus contenir d’erreur
```python
months = list(df_bis)
for i in range(len(months)):
    month = months[i]
```
    
#### Remplacer les erreurs par la valeur précédente (sans affecter les NaN car jours non existants)
```python
df_bis[month] = df_bis[month].replace(np.nan, 500)
df_bis[month] = pd.to_numeric(df_bis[month], errors='coerce').interpolate()
df_bis[month][df_bis[month] == 500] = np.nan
```
    
#### Remplacer les trop grands écarts par la moyenne des valeurs précédentes et suivantes
```python
prevVal = df_bis[month].shift(1)
nextVal = df_bis[month].shift(-1)
df_bis[month] = np.where((df_bis[month] - prevVal) > 20, (prevVal + nextVal) / 2, df_bis[month])

print(df_bis)
```

### Méthode utilisée
Pour toutes valeurs non numériques, on essaie dans un premier temps de les transformer en numérique (exemple: "=-1" en -1).

Dans un second temps, pour les valeurs qui n'ont pas pu être transformées, nous définissons la température à la moyenne de la température du jour précédent et suivant.
Les données corrigées s'approchent des données correctes. La différence la plus élevée est de 1.5°C.

### La capital dont on a eu les données
En comparant les écarts, nous nous apercevons que les périodes chaudes et froides correspondent à notre jeux de données. Nous pouvons en conclure qu'il s'agit des températures d'une même zone géographique. Au vu de la localisation de la station, il doit d'agir de la capitale Helsinki.

## Conclusion
Après la modification des données erronées, les nouvelles données paraissent le plus fiable possible. Ce qui nous a permis de localiser la station la plus proche du jeu de données.

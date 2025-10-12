---
jupytext:
  cell_metadata_json: true
  encoding: '# -*- coding: utf-8 -*-'
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
language_info:
  name: python
  pygments_lexer: ipython3
  nbconvert_exporter: python
---

# TP on the moon

+++

**Notions intervenant dans ce TP**

* suppression de colonnes avec `drop` sur une `DataFrame`
* suppression de colonne entièrement vide avec `dropna` sur une `DataFrame`
* accès aux informations sur la dataframe avec `info`
* valeur contenues dans une `Series` avec `unique` et `value_counts` 
* conversion d'une colonne en type numérique avec `to_numeric` et `astype` 
* accès et modification des chaînes de caractères contenues dans une colonne avec l'accesseur `str` des `Series`
* génération de la liste Python des valeurs d'une série avec `tolist`
   
**N'oubliez pas d'utiliser le help en cas de problème.**

**Répartissez votre code sur plusieurs cellules**

+++

1. importez les librairies `pandas` et `numpy`

```{code-cell} ipython3
import pandas as pd
import numpy as np
```

2. 1. lisez le fichier de données `data/objects-on-the-moon.csv`
   2.  affichez sa taille et regardez quelques premières lignes

```{code-cell} ipython3
df = pd.read_csv('data/objects-on-the-moon.csv')

df
```

```{code-cell} ipython3
df.shape
```

3. 1. vous remarquez une première colonne franchement inutile  
     utiliser la méthode `drop` des dataframes pour supprimer cette colonne de votre dataframe  
     `pd.DataFrame.drop?` pour obtenir de l'aide

```{code-cell} ipython3
df.columns[0]
df1=df.drop(df.columns[0], axis=1)
df1
```

4. 1. appelez la méthode `info` des dataframes (`non-null` signifie `non-nan` i.e. non manquant)
   1. remarquez une colonne entièrement vide

```{code-cell} ipython3
df1.info()

#on remarque un 0 dans la colonne Non null ppour le numero de colonne Size
```

5. 1. utilisez la méthode `dropna` des dataframes pour supprimer *en place* les colonnes qui ont toutes leurs valeurs manquantes  
     (on s'interdit un code qui ferait explicitement référence à la colonne `'Size'`)
   2. vérifiez que vous avez bien enlevé la colonne `'Size'`

```{code-cell} ipython3

```

```{code-cell} ipython3
#df.dropna? en regardant la documentation, dropna consider par defaut les lignes donc il faut changer les axes et on
df2=df1.dropna(axis='columns', how='all')
df2
```

6. 1. affichez la ligne d'`index` $88$, que remarquez-vous ?
   2. utilisez la méthode `dropna` des dataframes pour supprimer
      *en place* les lignes qui ont toutes leurs valeurs manquantes
      (et de nouveau sans faire référence à une ligne en particulier)

```{code-cell} ipython3
df2.iloc[88]
#on remarque qu'il n'y a pas de valeurs dans cette ligne
df2.dropna(how='all')
```

```{code-cell} ipython3
df3=df2.dropna(how='all')
df3
```

7. 1. utilisez l'attribut `dtypes` des dataframes pour voir le type de vos colonnes
   2. que remarquez vous sur la colonne des masses ?

```{code-cell} ipython3
df3.dtypes
#la colonne des masses est de type "obejct"
```

8. 1. utilisez la méthode `unique` des `Series`pour en regarder le contenu de la colonne des masses
   2. que remarquez vous ?

```{code-cell} ipython3
df3['Mass (lb)'].unique()
# certaines valurs sont sous la forme d'inferieur/ superieur surement lorsqu'il manquait de précision
```

9. 1. conservez la colonne `'Mass (lb)'` d'origine  
      (par exemple dans une colonne de nom `'Mass (lb) orig'`)  
   1. utilisez la fonction `pd.to_numeric` pour convertir  la colonne `'Mass (lb)'` en numérique  
      en remplaçant les valeurs invalides par la valeur manquante (NaN)
   1. naturellement vous vérifiez votre travail en affichant le type de la série `df['Mass (lb)']`
   1. combien y a-t-il de données manquantes dans cette colonne ?

```{code-cell} ipython3
df4=df3.copy()
df4['Mass (lb) orig'] = df3['Mass (lb)']
df5=pd.to_numeric(df3['Mass (lb)'], errors='coerce')

#on a mis coerce pour transformer les valeurs invalides pas NaN
```

```{code-cell} ipython3
df5.dtype
```

```{code-cell} ipython3
len(df4['Mass (lb) orig'])
```

```{code-cell} ipython3
df5.isna().sum()
```

```{code-cell} ipython3
#pour avoir le nombre de données manquantes on soustrait les deux 
```

10. 1. cette solution ne vous satisfait pas, vous ne voulez perdre aucune valeur  
       (même au prix de valeurs approchées)  
    1. vous décidez vaillamment de modifier les `str` en leur enlevant les caractères `<` et `>`  
       afin de pouvoir en faire des entiers
    - *hint:*  
       les `pandas.Series` formées de chaînes de caractères sont du type `pandas` `object`  
       mais elle possèdent un accesseur `str` qui permet de leur appliquer les méthodes python des `str`  
       (comme par exemple `replace`)
        ```python
        df['Mass (lb) orig'].str
        ```
        remplacer les `<` et les `>` par des '' (chaîne vide)
     3. utilisez la méthode `astype` des `Series` pour la convertir finalement en `int`

```{code-cell} ipython3
df4['Mass (lb) orig']=df4['Mass (lb) orig'].str.replace('>', ' ')
df4['Mass (lb) orig']=df4['Mass (lb) orig'].str.replace('<', ' ')
df4['Mass (lb) orig'][80] # je teste avec une des valeurs qui etait invalide 
```

```{code-cell} ipython3
df4['Mass (lb) orig'].astype(int)
```

11. 1. sachant `1 kg = 2.205 lb`  
   créez une nouvelle colonne `'Mass (kg)'` en convertissant les lb en kg  
   arrondissez les flottants en entiers en utilisant `astype`

```{code-cell} ipython3
df4['Mass (kg)'] = df4['Mass (lb) orig']* (1/2.205)
df4['Mass (kg)']=df4['Mass (kg)'].astype(int)
df4
```

12. 1. Quels sont les pays qui ont laissé des objets sur la lune ?
    2. Combien en ont-ils laissé en pourcentage (pas en nombre) ?  
     *hint:* regardez les paramètres de `value_counts`

```{code-cell} ipython3
df.value_counts?
```

```{code-cell} ipython3
df4['Country'].value_counts()
#on ne s'interesse qu'aux proportions donc on peut utiliser (normalize=True) et multiplier par 100 pour avoir le pourcentage 
df4['Country'].value_counts(normalize=True)*100
```

13. 1. quel est le poids total des objets sur la lune en kg ?
    2. quel est le poids total des objets laissés par les `United States`  ?

```{code-cell} ipython3
df4['Mass (kg)'].sum()
```

```{code-cell} ipython3
poids_US = df4.loc[df4['Country'] == 'United States', 'Mass (kg)'].sum()
poids_US
```

14. 1. quel pays a laissé l'objet le plus léger ?  

````{admonition} tip
:class: dropdown tip

voyez les méthodes `Series.idxmin()` et `Series.argmin()`
````

```{code-cell} ipython3
df4['Mass (kg)'].idxmin() # nous donne l'index du minimum de cette colonne , on accede ensuite au pays correspondant 
df['Country'][df4['Mass (kg)'].idxmin()] 
#c'est la Japon
```

15. 1. y-a-t-il un Memorial sur la lune ?  
     *hint:*  
     en utilisant l'accesseur `str` de la colonne `'Artificial object'`  
     regardez si une des descriptions contient le terme `'Memorial'`
    2. quel est le pays qui a mis ce mémorial ?

```{code-cell} ipython3
df4['Artificial object'].str.contains('Memorial') #Nous renvoie True or False pour chaque ligne
df4['Artificial object'].str.contains('Memorial').any() #renvoie true meme si il n'y en a qu'un
#on veut savoir lequel
df4.loc[df4['Artificial object'].str.contains('Memorial')] 
#on fait correspondre ca aux pays
df4.loc[df4['Artificial object'].str.contains('Memorial'), 'Country'] 
```

16. 1. faites la liste Python des objets sur la lune  
     *hint:* voyez la méthode `tolist()` des séries

```{code-cell} ipython3
df4['Artificial object'].unique().tolist()
```

***

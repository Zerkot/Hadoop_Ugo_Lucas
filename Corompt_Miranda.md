## Configuration par défaut de Hadoop

### 1. Combien de tags chaque film possède-t-il ?

On crée le script `NBR_tags_film.py` :

```python
from mrjob.job import MRJob

class TagsParFilm(MRJob):

    def mapper(self, _, line):
        if line.startswith("userId"):
            return
        champs = line.split(",")
        movieId = champs[1]
        yield movieId, 1

    def reducer(self, movieId, counts):
        yield movieId, sum(counts)

if __name__ == '__main__':
    TagsParFilm.run()
```

On déplace le fichier de données dans HDFS :

```bash
hdfs dfs -put ml-25m/tags.csv /user/maria_dev/tags.csv
```

On exécute le job :

```bash
python NBR_tags_film.py -r hadoop \
  --hadoop-streaming-jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar \
  hdfs:///user/maria_dev/ml-25m/tags.csv \
  -o hdfs:///user/maria_dev/output/NBR_tags_film
```

📄 [Voir l'output](https://github.com/Zerkot/Hadoop_Ugo_Lucas/blob/main/tp_hadoop_quest_1.txt) | [Voir le résultat](tp_hadoop_quest_1_rep.txt)


> **Résultat :** 1 093 360 tags répartis sur **45 251 films distincts**.

---

### 2. Combien de tags chaque utilisateur a-t-il ajoutés ?

On crée le script `NBR_tags_user.py` :

```python
from mrjob.job import MRJob

class TagsParUser(MRJob):

    def mapper(self, _, line):
        try:
            if line.startswith("userId"):
                return
            champs = line.split(",")
            userId = champs[0]
            yield userId, 1
        except Exception:
            pass

    def reducer(self, userId, counts):
        yield userId, sum(counts)

if __name__ == '__main__':
    TagsParUser.run()
```

On exécute le job :

```bash
python NBR_tags_user.py -r hadoop \
  --hadoop-streaming-jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar \
  hdfs:///user/maria_dev/ml-25m/tags.csv \
  -o hdfs:///user/maria_dev/output/NBR_tags_user
```

📄 [Voir l'output](tp_hadoop_quest_2.txt) | [Voir le résultat](tp_hadoop_quest_2_rep.txt)

> **Résultat :** 1 093 360 tags ajoutés par **14 592 utilisateurs distincts**.

---

## Configuration avec taille de bloc = 64 Mo

### 3. Combien de blocs le fichier occupe-t-il dans chacune des configurations ?

```bash
hdfs dfs -ls /user/maria_dev/ml-25m/tags.csv
# -rw-r--r--   1 maria_dev hdfs   38810332 2026-04-01 18:46 /user/maria_dev/ml-25m/tags.csv

hdfs getconf -confkey dfs.blocksize
# 134217728
```

| Configuration | Taille du bloc | Taille du fichier | Nombre de blocs |
|---|---|---|---|
| Par défaut | 128 Mo | ≈ 37 Mo | **1 bloc** |
| Modifiée | 64 Mo | ≈ 37 Mo | **1 bloc** |

> Dans les deux cas, le fichier (37 Mo) est inférieur à la taille du bloc, il occupe donc **1 seul bloc**.

---

### 4. Combien de fois chaque tag a-t-il été utilisé pour taguer un film ?

On crée le script `NBR_tag_utilise.py` :

```python
from mrjob.job import MRJob

class TagUtilise(MRJob):

    def mapper(self, _, line):
        try:
            if line.startswith("userId"):
                return
            champs = line.split(",")
            tag = champs[2]
            yield tag, 1
        except Exception:
            pass

    def reducer(self, tag, counts):
        yield tag, sum(counts)

if __name__ == '__main__':
    TagUtilise.run()
```

**Configuration par défaut :**

```bash
python NBR_tag_utilise.py -r hadoop \
  --hadoop-streaming-jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar \
  hdfs:///user/maria_dev/ml-25m/tags.csv \
  -o hdfs:///user/maria_dev/output/NBR_tag_utilise
```

📄 [Voir l'output](tp_hadoop_quest_4.txt) | [Voir le résultat](tp_hadoop_quest_4_rep.txt)

**Configuration avec bloc = 64 Mo :**

Le résultat est identique, seule la configuration du bloc change :

```bash
python NBR_tag_utilise.py -r hadoop \
  --hadoop-streaming-jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar \
  --jobconf dfs.blocksize=67108864 \
  hdfs:///user/maria_dev/ml-25m/tags.csv \
  -o hdfs:///user/maria_dev/output/NBR_tag_utilise_64
```

📄 [Voir l'output](tp_hadoop_quest_4_64.txt)

---

### 5. Pour chaque film, combien de tags le même utilisateur a-t-il introduits ?

On crée le script `NBR_tags_user_film.py` :

```python
from mrjob.job import MRJob

class TagsUserFilm(MRJob):

    def mapper(self, _, line):
        try:
            if line.startswith("userId"):
                return
            champs = line.split(",")
            userId = champs[0]
            movieId = champs[1]
            yield (userId, movieId), 1
        except Exception:
            pass

    def reducer(self, key, counts):
        yield key, sum(counts)

if __name__ == '__main__':
    TagsUserFilm.run()
```

**Configuration par défaut :**

```bash
python NBR_tags_user_film.py -r hadoop \
  --hadoop-streaming-jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar \
  hdfs:///user/maria_dev/ml-25m/tags.csv \
  -o hdfs:///user/maria_dev/output/NBR_tags_user_film
```

📄 [Voir l'output](tp_hadoop_quest_5.txt) | [Voir le résultat](tp_hadoop_quest_5_rep.txt)

**Configuration avec bloc = 64 Mo :**

Le résultat est identique, seule la configuration du bloc change :

```bash
python NBR_tags_user_film.py -r hadoop \
  --hadoop-streaming-jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar \
  --jobconf dfs.blocksize=67108864 \
  hdfs:///user/maria_dev/ml-25m/tags.csv \
  -o hdfs:///user/maria_dev/output/NBR_tags_user_film_64
```

📄 [Voir l'output](tp_hadoop_quest_5_64.txt)

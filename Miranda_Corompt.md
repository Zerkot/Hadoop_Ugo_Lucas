# CC2 Pratique
## Avec la configuration par défaut de Hadoop
1. Combien de tags chaque film possède-t-il ?
On fait d'abord "nano NBR_tags.py"
avec le code suivant : 

from mrjob.job import MRJob

class TagsParFilm(MRJob):

    def mapper(self, _, line):
        # Ignorer l'en-tête
        if line.startswith("userId"):
            return
        champs = line.split(",")
        movieId = champs[1]
        yield movieId, 1

    def reducer(self, movieId, counts):
        yield movieId, sum(counts)

if __name__ == '__main__':
    TagsParFilm.run()

Ensuite on déplace le code python qui est en local dans le hdfs avec la commande : hdfs dfs -put ml-25m/tags.csv /user/maria_dev/tags.csv
Ensuite on execute dans le hdfs : python NBR_tags_film.py -r hadoop --hadoop-streaming-jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar hdfs:///user/maria_dev/ml-25m/tags.csv -o hdfs:///user/maria_dev/output/NBR_tags_film
[Voir l'output](tp_hadoop_quest_1.txt)

[Voir le résultat](tp_hadoop_quest_1_rep.txt)
il y a 1093360 tags réparti sur 45 251 films distincts

2. Combien de tags chaque utilisateur a-t-il ajoutés ?
On fait donc la même chose que la question précédente avec ce script python :
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

avec cette commande : python NBR_tags_user.py -r hadoop --hadoop-streaming-jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar hdfs:///user/maria_dev/ml-25m/tags.csv -o hdfs:///user/maria_dev/output/NBR_tags_user
[Voir l'output](tp_hadoop_quest_2.txt)

[Voir le résultat](tp_hadoop_quest_2_rep.txt)

il y a 1 093 360 tags ajoutés pour 14 592 utilisateurs distincts.

## Avec la configuration de Hadoop suivante (taille du bloc par défaut et taille du bloc = 64 Mo)

3. Combien de blocs le fichier occupe-t-il dans HDFS dans chacune des configurations ?

[maria_dev@sandbox-hdp ~]$ hdfs dfs -ls /user/maria_dev/ml-25m/tags.csv
-rw-r--r--   1 maria_dev hdfs   38810332 2026-04-01 18:46 /user/maria_dev/ml-25m/tags.csv
[maria_dev@sandbox-hdp ~]$ hdfs getconf -confkey dfs.blocksize
134217728

Taille du fichier : 38 810 332 octets ≈ 37 Mo
Configuration par défaut — bloc de 128 Mo :

37 Mo < 128 Mo → 1 seul bloc

Configuration bloc = 64 Mo :

37 Mo < 64 Mo → 1 seul bloc aussi


4. Combien de fois chaque tag a-t-il été utilisé pour taguer un film ?
On fait un script python de la même manière que les deux premières questions :
[maria_dev@sandbox-hdp ~]$ nano NBR_tag_utilise.py

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
    
[maria_dev@sandbox-hdp ~]$ hdfs dfs -put NBR_tag_utilise.py /user/maria_dev/ml-25m
[maria_dev@sandbox-hdp ~]$ python NBR_tag_utilise.py -r hadoop --hadoop-streaming-jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar hdfs:///user/maria_dev/ml-25m/tags.csv -o hdfs:///user/maria_dev/output/NBR_tag_utilise

[Voir l'output](tp_hadoop_quest_3.txt)

[Voir le résultat](tp_hadoop_quest_3_rep.txt)

5. Pour chaque film, combien de tags le même utilisateur a-t-il introduits ?

[maria_dev@sandbox-hdp ~]$ nano NBR_tags_user_film.py

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
    
[maria_dev@sandbox-hdp ~]$ hdfs dfs -put NBR_tags_user_film.py /user/maria_dev/ml-25m
[maria_dev@sandbox-hdp ~]$ python NBR_tags_user_film.py -r hadoop --hadoop-streaming-jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar hdfs:///user/maria_dev/ml-25m/tags.csv -o hdfs:///user/maria_dev/output/NBR_tags_user_film

[Voir l'output](tp_hadoop_quest_5.txt)

[Voir le résultat](tp_hadoop_quest_5_rep.txt)



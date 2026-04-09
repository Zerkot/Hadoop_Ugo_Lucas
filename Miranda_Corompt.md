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

il y a 1 093 360 tags ajoutés pour 14 592 utilisateurs distincts.

## Avec la configuration de Hadoop suivante (taille du bloc par défaut et taille du bloc = 64 Mo)

3. Combien de blocs le fichier occupe-t-il dans HDFS dans chacune des configurations ?

[maria_dev@sandbox-hdp ~]$ hdfs dfs -ls /user/maria_dev/ml-25m/tags.csv
-rw-r--r--   1 maria_dev hdfs   38810332 2026-04-01 18:46 /user/maria_dev/ml-25m/tags.csv
[maria_dev@sandbox-hdp ~]$ hdfs getconf -confkey dfs.blocksize
134217728
5. 

# CC2 Pratique
## Avec la configuration par défaut de Hadoop
Combien de tags chaque film possède-t-il ?
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
voici l'output : tp_hadoop_quest_1
lien vers l'output
il y a 1093360 tags réparti sur 45 251 films distincts
---
## 2. Accès au cluster
Accès via navigateur :
- http://localhost:1080
- Ambari : http://localhost:8080
Identifiants :
- maria_dev / maria_dev
On peut aussi se connecter en SSH :
ssh maria_dev@localhost -p 2222
---
## 3. HDFS
### Commandes de base
- hdfs dfs -ls : liste les fichiers
- hdfs dfs -ls / : racine HDFS
- hdfs dfs -cat fichier : afficher contenu
- hdfs dfs -put fichier : envoyer vers HDFS
- hdfs dfs -rm fichier : supprimer
n hdfs dfs et hadoop fs sont équivalents.
---
### Version Hadoop
Commande :
hadoop version
La sandbox HDP 2.6.5 utilise Hadoop 2.x.
---
### Import de fichier
Création locale :
touch monfichier.txt
echo "test" > monfichier.txt
Envoi sur HDFS :
hdfs dfs -put monfichier.txt
---
### Manipulation HDFS
- mkdir : créer dossier
- cp : copier
- mv : déplacer
- rm -r : supprimer dossier
Commande pour voir l’état :
hdfs fsck /user
---
## 4. Dataset MovieLens
Téléchargement avec wget :
wget https://files.grouplens.org/datasets/movielens/ml-1m.zip
Problème rencontré :
erreur DNS (VM mal configurée)
Solution :
- modifier resolv.conf
- ou télécharger depuis le PC
---
## 5. Analyse HDFS (MES NOTES)
### dfs.replication
Valeur : 1
Cela signifie qu’un bloc n’est stocké qu’une seule fois (sandbox mono-noeud).
---
### Taille des blocs
Commande :
hdfs getconf -confKey dfs.blocksize
Résultat : 134217728 (~128 Mo)
---
### Analyse des blocs
Commande :
hdfs fsck dataset/ratings.csv -files -blocks
Résultat :
- ~678 Mo
- 11 blocs
- réplication = 1
Le découpage est cohérent.
## 6. MapReduce
### Installation
- yum install nano
- pip install mrjob
---
### Test local
python filmEvaluation.py evaluation.data
Résultat : comptage des notes (MapReduce local)
---
### Exécution Hadoop
python filmEvaluation.py -r hadoop --hadoop-streaming-jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-Analyse :
- traitement distribué
- stockage HDFS
---
## Conclusion
- HDFS découpe en blocs
- réplication des données
- MapReduce distribue le calcul
- environnement sandbox simplifié mais représentatif

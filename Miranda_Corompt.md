# TP Hadoop - HDFS & MapReduce
## 1. Installation VirtualBox et Sandbox HDP
VirtualBox permet de créer une machine virtuelle (VM) sur un ordinateur physique (machine hôte).
Dans notre cas, on utilise la Sandbox HDP qui simule un cluster Hadoop mono-noeud.
Configuration recommandée :
- 4 à 8 Go de RAM
- 4 CPU minimum
- Virtualisation activée
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

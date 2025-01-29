Cette erreur suggère un problème avec l'installation de SQLAlchemy. Nous allons réinstaller les dépendances correctement.

1. D'abord, désinstallons SQLAlchemy et Flask-SQLAlchemy :
```bash
pip uninstall sqlalchemy flask-sqlalchemy
```

2. Ensuite, réinstallons-les dans le bon ordre :
```bash
pip install SQLAlchemy==1.4.31
pip install Flask-SQLAlchemy==2.5.1
pip install python-dotenv
```

3. Vérifions les installations :
```bash
pip list
```

Vous devriez voir quelque chose comme :
- SQLAlchemy 1.4.31
- Flask-SQLAlchemy 2.5.1
- Flask
- python-dotenv

Une fois ces installations faites, essayez à nouveau de lancer :
```bash
python init_db.py
```

Si vous avez toujours une erreur, pouvez-vous me montrer le contenu de votre fichier requirements.txt ? On pourra ainsi vérifier les versions exactes des dépendances.

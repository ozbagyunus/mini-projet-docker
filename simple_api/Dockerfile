#Image de base
FROM python:3.13-slim

#Responsable de la maintenance
LABEL maintainer="OzbagYunus <yun73@hotmail.fr>"

# Copier le code de l'API à la racine "/"
COPY student_age.py /student_age.py

# Installer les dépendances système demandées par l'énoncé
RUN apt-get update -y \
  && apt-get install -y build-essential python3-dev libsasl2-dev libldap2-dev libssl-dev

# Copier d'abord requirements pour profiter du cache docker
COPY requirements.txt /requirements.txt

# Installation
RUN pip3 install -r /requirements.txt

# Dossier data + volume (pour monter student_age.json)
RUN mkdir -p /data
VOLUME ["/data"]

EXPOSE 5000

CMD ["python3", "/student_age.py"]

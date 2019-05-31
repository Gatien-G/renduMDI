# [ESIR2 IN] Rendu MDI - Groupe : Gatien Gaumerais et Paul Le Tannou

###Compte-rendu

### Etape 1
On se base sur une image Ubuntu pour la création de notre image Docker.
De base, cette image est très épurée, il faut donc récuperer un certain nombre de composantes.
Nous avons notamment besoin de Java, Python, git, cmake et maven pour la compilation et l'éxécution d'OpenCV et de l'application.
```bash
FROM ubuntu:18.04 

RUN apt-get update 
RUN apt-get install -y git cmake maven ant software-properties-common openjdk-8-jdk 
RUN apt-get install -y build-essential 
RUN apt-get install -y cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev 
RUN apt-get install -y python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libdc1394-22-dev 
```
### Etape 2
On récupère les fichiers source d'OpenCV, et on se place dans le dossier obtenu.
```bash
RUN git clone git://github.com/opencv/opencv.git 
WORKDIR /opencv
```

### Etape 3
On configure la variable d'environment pour indiquer l'emplacement de Java.
```bash
ENV JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64 
RUN export JAVA_HOME 
```

### Etape 4
On compile OpenCV, version 3.4.
```bash
RUN git checkout 3.4 && mkdir build && cd build && cmake -DBUILD_SHARED_LIBS=OFF .. && make -j8 
```

### Etape 5
On récupère une version corrigée de l'application. Nous avons du changer l'addresse d'OpenCV dans main.java.
```bash
WORKDIR / 
RUN git clone https://github.com/Gatien-G/ESIRTPDockerSampleApp
WORKDIR /ESIRTPDockerSampleApp 
```

### Etape 6
On installe toutes les dépendances de l'application
```bash
RUN mvn install:install-file \
 -Dfile=/opencv/build/bin/opencv-346.jar \ 
 -DgroupId=org.opencv \ 
 -DartifactId=opencv \ 
 -Dversion=3.4.6 \ 
 -Dpackaging=jar \ 
 -DgeneratePom=true && mvn package 
```

### Etape 7
Cette étape est nécéssaire pour récuperer la version 8 de java, qui est mise à jour lors de l'installation des composants précédents.
```bash
RUN update-java-alternatives -s /usr/lib/jvm/java-1.8.0-openjdk-amd64
```
### Etape 8
On configure la commande lancée au démarage du container pour executer l'application.
```bash
CMD ["java", "-Djava.library.path=/opencv/build/lib/", "-jar", "/ESIRTPDockerSampleApp/target/fatjar-0.0.1-SNAPSHOT.jar"]
```

----

### Création de l'image
On crée l'image grâce à la commande suivante :
```bash
docker build .
```

### Lancement de l'application
On peut alors créer un container à partir de l'image grâce à la commande suivante, en mappant le port nécéssaire :
```bash
docker run -p 8080:8080 ipimage
```
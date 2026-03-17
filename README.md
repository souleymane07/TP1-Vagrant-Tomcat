# Description
Ce projet montre comment automatiser le déploiement d'une application web Java en utilisant :
- **Vagrant** pour la création de la VM
- **Ubuntu 22.04** comme système d'exploitation
- **JDK 8, 11, 17** pour l'environnement Java
- **Tomcat 9** comme serveur d'application
- **Application JSP** simple avec affichage de la date

# Prérequis
- [VirtualBox](https://www.virtualbox.org/)
- [Vagrant](https://www.vagrantup.com/)
- Git


## 1. Cloner le dépôt
git clone https://github.com/votre-username/TP1-Vagrant-Tomcat.git
cd TP1-Vagrant-Tomcat

## 2. Démarrer la VM
vagrant up
<img width="1459" height="703" alt="image" src="https://github.com/user-attachments/assets/9c994582-e679-4fee-bc28-4c6d89c77475" />

## 3. Connecter à la VM
vagrant ssh
<img width="872" height="734" alt="image" src="https://github.com/user-attachments/assets/63aa44d2-3b19-4840-8a16-fac9a99bedcb" />

## 4. Installation des JDK
sudo apt update
sudo apt install -y openjdk-8-jdk openjdk-11-jdk openjdk-17-jdk
### 4.1. Verification des JDK installes
update-alternatives --list java
<img width="519" height="105" alt="image" src="https://github.com/user-attachments/assets/1839dbc1-dab0-42fd-afb5-eb4a841740ed" />

## 5. Installation de Tomcat 9
sudo apt install -y tomcat9 tomcat9-admin
sudo systemctl start tomcat9
sudo systemctl enable tomcat9
sudo systemctl status tomcat9
<img width="1386" height="448" alt="image" src="https://github.com/user-attachments/assets/f2261c50-544a-46a4-805c-bab95836aaae" />

## 6. Configuration de l'utilisateur admin
 sudo nano /etc/tomcat9/tomcat-users.xml
 ### 6.1. Contenu du fichier tomcat-users.xml
<role rolename="manager-gui"/>
<role rolename="admin-gui"/>
<user username="admin" password="admin123" roles="manager-gui,admin-gui"/>

## 7. Création de l'application web
  ### 7.1. Structure du Projet
<img width="179" height="102" alt="image" src="https://github.com/user-attachments/assets/38e7b790-2304-4c59-836e-c8fbe6146759" />
  ### 7.2. Code du fichier index.jsp
<!DOCTYPE html>
<html>
<head>
    <title>Mon Application Java</title>
    <style>
        body { font-family: Arial; margin: 40px; }
        .container { background: #f0f0f0; padding: 20px; border-radius: 10px; }
        .date { color: #666; }
    </style>
</head>
<body>
    <div class="container">
        <h1>Bienvenue sur mon application Java</h1>
        <p>Déployée avec Tomcat 9 sur Ubuntu 22.04</p>
        <p class="date">Date : <%= new java.util.Date() %></p>
        <p>Version Java : <%= System.getProperty("java.version") %></p>
    </div>
</body>
</html>

## 8. Deploiement

  ### 8.1. Création de l'archive WAR
cd ~
jar -cvf mywebapp.war -C mywebapp/ .

  ### 8.2. Déploiement dans Tomcat
sudo cp mywebapp.war /var/lib/tomcat9/webapps/

## 9. Contenu du script deploy.sh
TOMCAT_SERVICE="tomcat9"
TOMCAT_WEBAPPS="/var/lib/tomcat9/webapps"
MYWEBAPP_PATH="/home/vagrant/mywebapp"

show_menu() {
    clear
    echo "====================================="
    echo "     GESTION DU SERVEUR TOMCAT      "
    echo "====================================="
    echo "1. Démarrer Tomcat"
    echo "2. Arrêter Tomcat"
    echo "3. Redémarrer Tomcat"
    echo "4. Status de Tomcat"
    echo "5. Voir les logs de Tomcat"
    echo "6. Lister les applications déployées"
    echo "7. Redéployer l'application mywebapp"
    echo "8. Changer la version de Java (JDK)"
    echo "9. Tester l'application (URL)"
    echo "10. Quitter"
    echo "====================================="
    echo -n "Choisissez une option [1-10]: "
}

deploy_app() {
    echo "-----------------------------------"
    echo "Re-création de l'archive WAR..."
    cd /home/vagrant
    
    # Supprimer l'ancien WAR s'il existe
    rm -f mywebapp.war
    
    # Créer le nouveau WAR
    jar -cvf mywebapp.war -C mywebapp/ .
    
    # Déployer dans Tomcat
    echo "Copie vers Tomcat..."
    sudo rm -rf $TOMCAT_WEBAPPS/mywebapp*
    sudo cp mywebapp.war $TOMCAT_WEBAPPS/
    
    echo "-----------------------------------"
    echo "Application redéployée!"
    echo "URL: http://192.168.33.10:8080/mywebapp/"
    echo "-----------------------------------"
}

change_java_version() {
    echo "-----------------------------------"
    echo "Versions Java disponibles:"
    echo "-----------------------------------"
    sudo update-alternatives --config java
    echo "-----------------------------------"
    echo "Nouvelle version Java:"
    java -version
    echo "-----------------------------------"
}

show_status() {
    echo "-----------------------------------"
    echo "Status de Tomcat:"
    echo "-----------------------------------"
    sudo systemctl status $TOMCAT_SERVICE --no-pager -l
    echo ""
}

show_logs() {
    echo "-----------------------------------"
    echo "Dernières lignes des logs Tomcat:"
    echo "-----------------------------------"
    sudo tail -30 /var/log/tomcat9/catalina.out
    echo ""
}

list_apps() {
    echo "-----------------------------------"
    echo "Applications déployées dans Tomcat:"
    echo "-----------------------------------"
    ls -la $TOMCAT_WEBAPPS/
    echo ""
    echo "Applications accessibles:"
    for app in $TOMCAT_WEBAPPS/*/; do
        if [ -d "$app" ]; then
            app_name=$(basename $app)
            if [ "$app_name" != "ROOT" ]; then
                echo "  - http://192.168.33.10:8080/$app_name/"
            else
                echo "  - http://192.168.33.10:8080/ (ROOT)"
            fi
        fi
    done
    echo "-----------------------------------"
}

test_app() {
    echo "-----------------------------------"
    echo "Pour tester l'application, ouvrez dans votre navigateur:"
    echo ""
    echo "Application principale:"
    echo "  http://192.168.33.10:8080/mywebapp/"
    echo ""
    echo "Interface de gestion Tomcat:"
    echo "  http://192.168.33.10:8080/manager/html"
    echo "  (utilisateur: admin, mot de passe: admin123)"
    echo ""
    echo "Page d'accueil Tomcat:"
    echo "  http://192.168.33.10:8080/"
    echo "-----------------------------------"
}

while true; do
    show_menu
    read choice
    
    case $choice in
        1)
            echo "Démarrage de Tomcat..."
            sudo systemctl start $TOMCAT_SERVICE
            sleep 2
            show_status
            echo ""
            echo "Appuyez sur Entrée pour continuer..."
            read
            ;;
        2)
            echo "Arrêt de Tomcat..."
            sudo systemctl stop $TOMCAT_SERVICE
            sleep 2
            show_status
            echo ""
            echo "Appuyez sur Entrée pour continuer..."
            read
            ;;
        3)
            echo "Redémarrage de Tomcat..."
            sudo systemctl restart $TOMCAT_SERVICE
            sleep 3
            show_status
            echo ""
            echo "Appuyez sur Entrée pour continuer..."
            read
            ;;
        4)
            show_status
            echo ""
            echo "Appuyez sur Entrée pour continuer..."
            read
            ;;
        5)
            show_logs
            echo ""
            echo "Appuyez sur Entrée pour continuer..."
            read
            ;;
        6)
            list_apps
            echo ""
            echo "Appuyez sur Entrée pour continuer..."
            read
            ;;
        7)
            deploy_app
            echo ""
            echo "Appuyez sur Entrée pour continuer..."
            read
            ;;
        8)
            change_java_version
            echo ""
            echo "Appuyez sur Entrée pour continuer..."
            read
            ;;
        9)
            test_app
            echo ""
            echo "Appuyez sur Entrée pour continuer..."
            read
            ;;
        10)
            echo "Au revoir!"
            exit 0
            ;;
        *)
            echo "Option invalide! Choisissez entre 1 et 10."
            sleep 2
            ;;
    esac
done

## 10. Utilisation du script deploy.sh
chmod +x deploy.sh
./deploy.sh
<img width="402" height="304" alt="image" src="https://github.com/user-attachments/assets/a95b1cf6-c2f6-4425-a0ec-cbed1ab3b52b" />


## 11. Accès à l'application
### 11.1. Application : http://192.168.33.10:8080/mywebapp/
<img width="859" height="350" alt="image" src="https://github.com/user-attachments/assets/b23f47bc-773e-466b-8489-10123aef3e5f" />

### 11.2. Manager Tomcat : http://192.168.33.10:8080/manager/html (admin/admin123)
<img width="742" height="309" alt="image" src="https://github.com/user-attachments/assets/2080ba19-0be7-4023-ac5b-1bdfedc94ec9" />

### 11.3. Page d'accueil Tomcat : http://192.168.33.10:8080/
<img width="1348" height="509" alt="image" src="https://github.com/user-attachments/assets/399bfeb6-6668-4408-86b3-63f50fe32fbb" />

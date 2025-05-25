# Jenkins


## Instalacion

> **DOCUMENTACION OFICIAL**: [Documentacion](https://github.com/jenkinsci/helm-charts/blob/main/charts/jenkins/README.md)

La instalacion de jenkins la haremos usando el helm oficial de jenkins

``` shell
helm repo add jenkins https://charts.jenkins.io
helm repo update
kubectl create namespace jenkins
# helm show values jenkins/jenkins --version 5.7.14 > values.default.yaml

kubectl apply -f manifests
helm upgrade --install jenkins jenkins/jenkins -n jenkins -f values-5-7-14.yaml --version 5.7.14
#Si se cambia el configmap del certificado raiz cuidado con la key. URL en función de la app a resolver el SSL
kubectl exec -it jenkins-0 -n jenkins -- /bin/sh -c 'git config --global http.https://forgejo.alopezpa.homelab/.sslcainfo /mnt/certificate/ca-alopezpa-homelab-raiz.crt'
```

> NOTA IMPORTANTE: Después de realizar la primera instalación de jenkins en la que el atributo del values JCasC.defaultConfig = true, este se debe cambiar por JCasC.defaultConfig = false y ejecutar el siguiente comando. Esto evitara que cada vez que se reinicie el pod la configuración de jenkins se resetee a lo establecido en el plugin.
``` shell
helm upgrade --install jenkins jenkins/jenkins -n jenkins -f values.yaml --version 5.7.14
```

## Instalar/Actualizar plugins offline

> NOTA: Esto solo es una forma de hacerlo manual y offline, se va a trabajar en el deployment de jenkins para que sea automatico al desplegar un nuevo Jenkins. De momento se quedará de forma manual porque para poder hacer esto es necesario abrir la conexión de todos los sitios de donde descargar los plugins en el proxy corporativo y son muchisimos enlaces.

https://github.com/jenkinsci/plugin-installation-manager-tool

> Descargar todos los plugins indicados en el yml de la versión de jenkins indicada en el comando
``` shell
java -jar plugins/jenkins-plugin-manager-2.12.14.jar --jenkins-version 2.452 --plugin-download-directory plugins/pluginsDownload -f plugins/plugins.yaml 
java -jar plugins/jenkins-plugin-manager-2.13.0.jar --jenkins-version 2.452 --plugin-download-directory plugins/pluginsDownload -f plugins/plugins.yaml 

```
> Desplegar los plugins en el pod de jenkins
``` shell
kubectl cp plugins/pluginsDownload/. jenkins-0:/var/jenkins_home/plugins -c jenkins -n jenkins
```

> Si deseas realizar un backup del archivo de config y despues restaurarlo:
``` shell
kubectl cp jenkins-0:/var/jenkins_home/config.xml config.xml -c jenkins -n jenkins
kubectl cp config.xml jenkins-0:/var/jenkins_home/config.xml -c jenkins -n jenkins

```

> Si deseas realizar backups de los jobs (Cuando hagas cp la ruta debe existir en la persistencia y después reiniciar jenkins):
```shell
kubectl cp jenkins-0:/var/jenkins_home/jobs/templates/jobs/cd-generic-buildingblock-job/config.xml cd-generic-bb.xml -c jenkins -n jenkins 
kubectl cp jenkins-0:/var/jenkins_home/jobs/templates/jobs/cd-new-generic-job/config.xml cd-new-generic.xml -c jenkins -n jenkins 
kubectl cp jenkins-0:/var/jenkins_home/jobs/templates/jobs/ci-new-generic-job/config.xml ci-new-generic.xml -c jenkins -n jenkins

kubectl cp jenkins-0:/var/jenkins_home/jobs/devops/jobs/utils/jobs/create-git-repository/config.xml jobs/create-git-repository.xml -c jenkins -n jenkins
kubectl cp jenkins-0:/var/jenkins_home/jobs/devops/jobs/utils/jobs/maven-parent-pom/config.xml jobs/maven-parent-pom.xml -c jenkins -n jenkins
kubectl cp jenkins-0:/var/jenkins_home/jobs/devops/jobs/utils/jobs/maven-quarkus-parent-pom/config.xml jobs/maven-quarkus-parent-pom.xml -c jenkins -n jenkins
kubectl cp jenkins-0:/var/jenkins_home/jobs/devops/jobs/utils/jobs/passwords/config.xml jobs/passwords.xml -c jenkins -n jenkins


kubectl cp cd-new-generic.xml jenkins-0:/var/jenkins_home/jobs/templates/jobs/cd-generic-job/config.xml -c jenkins -n jenkins
kubectl cp ci-new-generic.xml jenkins-0:/var/jenkins_home/jobs/templates/jobs/ci-generic-job/config.xml -c jenkins -n jenkins

kubectl cp jobs/create-git-repository.xml jenkins-0:/var/jenkins_home/jobs/utils/jobs/create-git-repository/config.xml -c jenkins -n jenkins
kubectl cp jobs/maven-parent-pom.xml jenkins-0:/var/jenkins_home/jobs/utils/jobs/maven-parent-pom/config.xml -c jenkins -n jenkins
kubectl cp jobs/maven-quarkus-parent-pom.xml jenkins-0:/var/jenkins_home/jobs/utils/jobs/maven-quarkus-parent-pom/config.xml -c jenkins -n jenkins
kubectl cp jobs/passwords.xml jenkins-0:/var/jenkins_home/jobs/utils/jobs/passwords/config.xml -c jenkins -n jenkins

```

## TODO:

- Transoformar en cuadro helm
- Autoinstalar plugins
- Autoconfigurar plugins
- Autoconfigurar keycloak
- montar certificados confianza
- Que todo sea definido por codigo y no por la UI

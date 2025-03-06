# Red-Hat-Open-Shift-Kasten-Backup

En esta guía se presenta un paso a paso para desplegar una instancia de Kasten, realizar backups de aplicaciones en un clúster de Openshift y restaurar dichos backups en otro clúster de Openshift.


## Contenido
1. [Pre-Requisitos](#pre-requisitos-pencil)
2. [Instalación de Kasten en Red Hat Openshift](#instalación-de-kasten-en-red-hat-openshift-⚙️)
3. [Instalacion alternativa por Operadores.](#Instalacion-alternativa-por-Operadores.)  
4. [Configuración de una Location de IBM Cloud Object Storage](#configuración-de-una-location-de-ibm-cloud-object-storage-☁️)
5. [Creación y ejecución de una política de Backup](#creación-y-ejecución-de-una-política-de-backup-🧳)
6. [Restauración de un Backup alojado en IBM Cloud Object Storage](#restauración-de-un-backup-alojado-en-ibm-cloud-object-storage-📂)
7. [consideraciones importantes](#consideraciones-importantes-📌)
8. [Referencias](#referencias-📄)
9. [Autores](#autores-black_nib)

## Pre-Requisitos :pencil:
- Contar con un clúster de Openshift en [IBM Cloud](https://cloud.ibm.com/kubernetes/catalog/create?platformType=openshift&catalog_query=aHR0cHM6Ly9jbG91ZC5pYm0uY29tL2NhdGFsb2c%2FY2F0ZWdvcnk9Y29udGFpbmVycw%3D%3D)
- Contar con una instancia de [IBM Cloud Object Storage](https://cloud.ibm.com/objectstorage/create)

## Instalación de Kasten en Red Hat Openshift :gear:

1. Ingrese a su clúster de Red Hat Openshift en IBM Cloud. Allí, dé click en **Openshift Web Console** para ingresar a la consola web de Openshift

<img width="800" alt="" src="img/1cluster.png"> 

2. En la consola web, dé click en la esquina superior derecha donde está su nombre de usuario, y seleccione la opción **Copy Login Command** En la pestaña que se abre copie el comando de inicio de sesión, que inicia con las palabras ```oc login```

<img width="800" alt="" src="img/2logincommand.png"> 

3. Regrese a la página de inicio de su clúster, allí seleccione la opción **IBM Cloud Shell** en la barra superior. Apenas el shell cargue, ingrese el comando que obtuvo en el paso anterior, así entrará a su clúster desde la terminal.

<img width="800" alt="" src="img/3shell.png"> 

4. A continuación, se realizará la configuración de los requisitos de Kasten, así como los **Pre-Flight Checks**, para ello ingrese los siguientes comandos:

```
helm repo add kasten https://charts.kasten.io/
```

```
kubectl create namespace kasten-io
```

```
curl https://docs.kasten.io/tools/k10_primer.sh | bash
```

5. Ahora, se instalará Kasten en el clúster, para lo cual debe ingresar el siguiente comando:
Para clústers que no se encuentren alojados en la nube, use [este comando](#clústers-on-premise-ibm-satellite)

```
helm install k10 kasten/k10 --namespace=kasten-io --set scc.create=true --set route.enabled=true --set route.path="/k10" --set auth.tokenAuth.enabled=true
```

6. Cuando finalice la instalación, podrá ingresar al dashboard de Kasten. Regrese a la pestaña donde tiene la consola Web de Openshift, y en el ambiente **Developer** seleccione el proyecto **kasten-io**

<img width="800" alt="" src="img/6proyecto.png"> 

7. En la pantalla verá todos los componentes de Kasten, incluyendo el gateway, que es el único componente con ruta de acceso. Esto puede distinguirlo porque posee una flecha en la esquina superior derecha. Dé click en el ícono de flecha del gateway para acceder al dashboard de Kasten.

<img width="800" alt="" src="img/7gateway.png"> 

8. Para acceder al dashboard de Kasten necesitará un token de acceso. Este token lo puede encontrar en la página que abrió en el paso 2 (**Copy login command**), bajo el título de API token.

<img width="800" alt="" src="img/8token.png"> 

9. La primera vez que acceda al dashboard se le pedirá aceptar las condiciones del servicio, para ello ingrese los datos que se solicitan.

<img width="800" alt="" src="img/9email.png"> 


10. Con esto queda finalizada la instalación de Kasten en Red Hat Openshift.

## Instalacion alternativa por Operadores

1. Ingrese al Operator hub y procesada a buscar kasten.

![image](https://github.com/user-attachments/assets/08062075-fb35-4c94-90f5-fdec37d9b8c5)

2. Instale el operador seleccionándolo de la lista de operadores disponibles.

![image](https://github.com/user-attachments/assets/c45a77fd-4746-4e8d-a14a-45f86db9a4cf)

3. Una vez instalado, búsquelo en el proyecto kasten-io para verificar su disponibilidad.

![image](https://github.com/user-attachments/assets/be879ca3-9fd0-4b09-8c12-ac5364705a3f)


5. Proceda a crear un instancia de K10.

![image](https://github.com/user-attachments/assets/cfbc6946-6ef8-412a-bb42-1b7532af6c3b)


6. Utilice la siguiente configuracion para crear el K10.

![image](https://github.com/user-attachments/assets/36a2cd40-b7a4-406c-b75c-aa02fee4a9f0)

## Configuración de una Location de IBM Cloud Object Storage :cloud:

Para almacenar los backups que se van a generar se usará una instancia de IBM Cloud Object Storage, que es compatible con S3. 

1. Ingrese a su instancia de IBM Cloud Object Storage, en la pestaña **Service Credentials**. 

<img width="800" alt="" src="img/1icos.png"> 


2. Dé click en **New Credential** y asigne un nombre a las credenciales que está creando. Asigne el rol de **Writer** y habilite la opción de credenciales HMAC. Dé click en **Add**

<img width="800" alt="" src="img/2credentials.png"> 

3. Regrese a la pestaña **Buckets** y dé click en **Create Bucket**. Seleccione **Customize yout bucket**

<img width="800" alt="" src="img/3customize.png"> 

4. Asigne un nombre a su bucket, seleccione la resiliencia regional y la ubicación. Finalmente, Seleccione el tipo de almacenamiento que más se ajuste al uso que hará de los backups. En este caso y como se planea acceder a la data constantemente para pruebas se selecciona la opción **Smart Tier**

<img width="800" alt="" src="img/4bucket.png"> 

5. Ya con el bucket de IBM Cloud Object Storage creado, regrese al dashboard de Kasten y seleccione la pestaña **Settings**, opción **Locations > New Profile**

6. Asigne un nombre al perfil de almacenamiento, seleccione la opción **S3 Compatible** y diligencie los campos con la información del bucket y las credenciales HMAC que creó previamente. Dé click en **Save Profile**

<img width="400" alt="" src="img/6profile.png"> 

7. Cuando termine de crear el perfil, deberá ver un recuadro con información de este y el indicador de status **valid**

<img width="800" alt="" src="img/7validProfile.png"> 

## Creación y ejecución de una política de Backup :luggage:

1. En el dashboard de Kasten, dé click en la opción **Policies > Create New Policy**


2. Diligencie el formulario de la siguiente forma:
- **Name**: Asigne un nombre para su política de backup
- **Action**: Snapshot
- **Backup Frequency**: Seleccione la frecuencia con la que quiere realizar snapshots, en este caso se selecciona **On Demand**
- **Enable Backups via snapshot exports**: Habilite esta opción y elija el perfil de almacenamiento que se configuró previamente
- **Select Applications**: By Name. Seleccione las aplicaciones a las que desea hacer backup. 
- **Select Application Resources**: All Resources

3. Cuando termine de crear la política, esta aparecerá con el indicador **valid**. Puede dar click en **run once** para realizar el backup manualmente. Si configuró backups automáticos estos se realizarán de acuerdo a la frecuencia seleccionada.

<img width="800" alt="" src="img/3validPolicy.png"> 

4. En la parte inferior de la política encontrará un botón que dice **Show import details**, deberá guardar la cadena de texto allí alojada para poder restaurar el backup en un clúster diferente (se realiza en la siguiente sección)


## Restauración de un Backup alojado en IBM Cloud Object Storage :open_file_folder:

Para restaurar un backup realizado previamente en otro cluster, repita los pasos de las tres primeras secciones de la guía en el clúster en el que desea restaurar el backup:
1. [Pre-Requisitos](#pre-requisitos-pencil)
2. [Instalación de Kasten en Red Hat Openshift](#instalación-de-kasten-en-red-hat-openshift-⚙️)
3. [Configuración de una Location de IBM Cloud Object Storage](#configuración-de-una-location-de-ibm-cloud-object-storage-☁️)

Luego de haber repetido los pasos anteriores en el nuevo clúster, ingrese al dashboard del kasten recién instalado.

1. Dé click en **policies > create new policy**
2. Diligencie el formulario de la siguiente forma:
- **Name**: Asigne un nombre para su política de backup
- **Action**: Import
- **Restore After Import**: Habilite esta opción
- **Import Frequency**: Seleccione la frecuencia con la que quiere realizar snapshots, en este caso se selecciona **On Demand**
- **Config Data for Import**: ingrese la cadena de texto que guardó cuando realizó el backup
- **Profile for Import**: Seleccione el perfil de storage que configuró previamente. Recuerde que debe ser el mismo storage en el que almacenó el backup en primer lugar. 

3. Seleccione **Create Policy**, y del mismo modo que generó el backup, dé click en **run once**, en el dashboard podrá ver el avance en la restauración de la aplicación. Cuando esta se complete, podrá verificar en su clúster que ahora existe un namespace con el mismo nombre y los mismos contenidos de la aplicación que tenía en el clúster anterior.

<img width="800" alt="" src="img/dashboard.png"> 

## Consideraciones importantes :pushpin:

### Token de Ingreso
Si no tiene acceso a su token de ingreso a través de la consola de Openshift, puede obtenerlo desde la línea de comandos:

```
oc whoami --show-token
```

### Clústers On-Premise (IBM Satellite)
En caso de que su clúster de Openshift no esté desplegado en IBM Cloud sino en un ambiente on-premise, como lo es Satellite, el comando de instalación varía un poco:

```
helm install k10 kasten/k10 --namespace=kasten-io --set scc.create=true --set route.enabled=true --set route.path="/k10" --set auth.tokenAuth.enabled=true   --set global.persistence.storageClass=<NOMBRE_DE_LA_STORAGE_CLASS>
```

Por ejemplo, si su clúster se encuentra en Satellite con un almacenamiento local-file, el comando sería:
```
helm install k10 kasten/k10 --namespace=kasten-io --set scc.create=true --set route.enabled=true --set route.path="/k10" --set auth.tokenAuth.enabled=true   --set global.persistence.storageClass=sat-local-file-gold
```

Adicionalmente, tenga en cuenta que para esta instalación necesita attachar 6 instancias de 20GB de File Storage en sus máquinas, para permitir la persistencia de información de los microservicios de la aplicación. Para más detalles del proceso de configuración de File Storage, remítase a [esta guía](https://github.com/emeloibmco/IBM-Cloud-Satellite-Configuracion)

### Desinstalación
Si desea realizar una desinstalación limpia de Kasten, use este comando:
```
helm uninstall k10 --namespace=kasten-io
```


## Referencias :page_facing_up:
- [https://docs.kasten.io/latest/install/requirements.html](https://docs.kasten.io/latest/install/requirements.html)
- [https://docs.kasten.io/latest/install/openshift/helm.html](https://docs.kasten.io/latest/install/openshift/helm.html)
- [https://docs.kasten.io/latest/access/dashboard.html#access-via-openshift-routes](https://docs.kasten.io/latest/access/dashboard.html#access-via-openshift-routes)
- [https://docs.kasten.io/latest/usage/configuration.html#amazon-s3-or-s3-compatible-storage](https://docs.kasten.io/latest/usage/configuration.html#amazon-s3-or-s3-compatible-storage)
- [ https://docs.kasten.io/latest/usage/protect.html#backups]( https://docs.kasten.io/latest/usage/protect.html#backups)
- [https://docs.kasten.io/latest/usage/restore.html](https://docs.kasten.io/latest/usage/restore.html)
- [https://docs.kasten.io/latest/usage/migration.html#importing-applications](https://docs.kasten.io/latest/usage/migration.html#importing-applications)


## Autores :black_nib:
Equipo IBM Cloud Tech Sales Colombia

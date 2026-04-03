------------------------------------------------------------------------------------------------------
ATELIER API-DRIVEN INFRASTRUCTURE
------------------------------------------------------------------------------------------------------
L’idée en 30 secondes : **Orchestration de services AWS via API Gateway et Lambda dans un environnement émulé**.  
Cet atelier propose de concevoir une architecture **API-driven** dans laquelle une requête HTTP déclenche, via **API Gateway** et une **fonction Lambda**, des actions d’infrastructure sur des **instances EC2**, le tout dans un **environnement AWS simulé avec LocalStack** et exécuté dans **GitHub Codespaces**. L’objectif est de comprendre comment des services cloud serverless peuvent piloter dynamiquement des ressources d’infrastructure, indépendamment de toute console graphique.Cet atelier propose de concevoir une architecture API-driven dans laquelle une requête HTTP déclenche, via API Gateway et une fonction Lambda, des actions d’infrastructure sur des instances EC2, le tout dans un environnement AWS simulé avec LocalStack et exécuté dans GitHub Codespaces. L’objectif est de comprendre comment des services cloud serverless peuvent piloter dynamiquement des ressources d’infrastructure, indépendamment de toute console graphique.
  
-------------------------------------------------------------------------------------------------------
Séquence 1 : Codespace de Github
-------------------------------------------------------------------------------------------------------
Objectif : Création d'un Codespace Github  
Difficulté : Très facile (~5 minutes)
-------------------------------------------------------------------------------------------------------
RDV sur Codespace de Github : <a href="https://github.com/features/codespaces" target="_blank">Codespace</a> **(click droit ouvrir dans un nouvel onglet)** puis créer un nouveau Codespace qui sera connecté à votre Repository API-Driven.
  
---------------------------------------------------
Séquence 2 : Création de l'environnement AWS (LocalStack)
---------------------------------------------------
Objectif : Créer l'environnement AWS simulé avec LocalStack  
Difficulté : Simple (~5 minutes)
---------------------------------------------------

Dans le terminal du Codespace copier/coller les codes ci-dessous etape par étape :  

**Installation de l'émulateur LocalStack**  
```
sudo -i mkdir rep_localstack
```
```
sudo -i python3 -m venv ./rep_localstack
```
```
sudo -i pip install --upgrade pip && python3 -m pip install localstack && export S3_SKIP_SIGNATURE_VALIDATION=0
```
```
localstack start -d
```
**vérification des services disponibles**  
```
localstack status services
```
**Réccupération de l'API AWS Localstack** 
Votre environnement AWS (LocalStack) est prêt. Pour obtenir votre AWS_ENDPOINT cliquez sur l'onglet **[PORTS]** dans votre Codespace et rendez public votre port **4566** (Visibilité du port).
Réccupérer l'URL de ce port dans votre navigateur qui sera votre ENDPOINT AWS (c'est à dire votre environnement AWS).
Conservez bien cette URL car vous en aurez besoin par la suite.  

Pour information : IL n'y a rien dans votre navigateur et c'est normal car il s'agit d'une API AWS (Pas un développement Web type UX).

---------------------------------------------------
Séquence 3 : Exercice
---------------------------------------------------
Objectif : Piloter une instance EC2 via API Gateway
Difficulté : Moyen/Difficile (~2h)
---------------------------------------------------  
Votre mission (si vous l'acceptez) : Concevoir une architecture **API-driven** dans laquelle une requête HTTP déclenche, via **API Gateway** et une **fonction Lambda**, lancera ou stopera une **instance EC2** déposée dans **environnement AWS simulé avec LocalStack** et qui sera exécuté dans **GitHub Codespaces**. [Option] Remplacez l'instance EC2 par l'arrêt ou le lancement d'un Docker.  

**Architecture cible :** Ci-dessous, l'architecture cible souhaitée.   
  
![Screenshot Actions](API_Driven.png)   
  
---------------------------------------------------  
## Processus de travail (résumé)

1. Installation de l'environnement Localstack (Séquence 2)
2. Création de l'instance EC2
3. Création des API (+ fonction Lambda)
4. Ouverture des ports et vérification du fonctionnement

---------------------------------------------------
Séquence 4 : Documentation  
Difficulté : Facile (~30 minutes)
API-Driven EC2 Controller — LocalStack + GitHub Codespaces
Piloter une instance EC2 via API Gateway et une fonction Lambda, dans un environnement AWS simulé avec LocalStack.
---------------------------------------------------
   Prérequis
Un compte GitHub avec accès aux Codespaces
Un compte LocalStack (version Pro ou snooze activé)
Python 3.11 (disponible par défaut dans les Codespaces)
---------------------------------------------------
Installation de l'environnement
1. Démarrer LocalStack
   LOCALSTACK_ACKNOWLEDGE_ACCOUNT_REQUIREMENT=1 localstack start -d
   Le -d lance LocalStack en arrière-plan (mode daemon)
   ---------------------------------------------------
   2. Vérifier que LocalStack tourne
      localstack status services
      Tous les services doivent afficher available
      ---------------------------------------------------
      3.Installer l'AWS CLI
pip install awscli --break-system-packages
      ---------------------------------------------------
      4. Configurer les credentials AWS (factices pour LocalStack)
         aws configure
         Renseigner les valeurs suivantes :
AWS Access Key ID     : test
AWS Secret Access Key : test
Default region name   : us-east-1
Default output format : json
         ---------------------------------------------------
         5. Exposer le port 4566
Dans l'onglet PORTS de VS Code (GitHub Codespaces) :
Cliquer sur "Transférer un port"
Saisir 4566 et appuyer sur Entrée
Clic droit sur le port → Visibilité du port → Public
L'URL publique ressemblera à :
https://<nom-du-codespace>-4566.app.github.dev
            ---------------------------------------------------
            Déploiement pas à pas
Étape 1 — Créer l'instance EC2
aws --endpoint-url=http://localhost:4566 ec2 run-instances \
  --image-id ami-024f768332f0 \
  --instance-type t2.micro \
  --count 1 \
  --region us-east-1
 Noter l'InstanceId retourné (ex: i-f76da277dbd264cfc), il sera utilisé dans la Lambda.
Vérifier que l'instance est bien démarrée :
aws --endpoint-url=http://localhost:4566 ec2 describe-instances \
  --instance-ids <INSTANCE_ID> \
  --region us-east-1 \
  --query 'Reservations[0].Instances[0].State.Name'
Résultat attendu : "running"
             ---------------------------------------------------
            Étape 2 — Créer la fonction Lambda
Créer le fichier lambda_function.py :
import boto3
import json
def lambda_handler(event, context):
    ec2 = boto3.client(
        'ec2',
        region_name='us-east-1',
        endpoint_url='http://localhost.localstack.cloud:4566'
    )
instance_id = '<INSTANCE_ID>'  # Remplacer par votre InstanceId
    path = event.get('path', '')
if path == '/start':
        ec2.start_instances(InstanceIds=[instance_id])
        message = f'Instance {instance_id} démarrée'
    elif path == '/stop':
        ec2.stop_instances(InstanceIds=[instance_id])
        message = f'Instance {instance_id} stoppée'
    elif path == '/status':
        response = ec2.describe_instances(InstanceIds=[instance_id])
        state = response['Reservations'][0]['Instances'][0]['State']['Name']
        return {
            'statusCode': 200,
            'body': json.dumps({'instanceId': instance_id, 'state': state})
        }
    else:
        return {
            'statusCode': 400,
            'body': json.dumps({'error': 'Route invalide. Utilise /start, /stop ou /status.'})
        }
 return {
        'statusCode': 200,
        'body': json.dumps({'message': message, 'instanceId': instance_id})
    }
             ---------------------------------------------------
            Étape 3 — Créer l'API Gateway
Créer l'API :
aws --endpoint-url=http://localhost:4566 apigateway create-rest-api \
  --name "ec2-api" \
  --region us-east-1
Noter l'id (API_ID) et le rootResourceId retournés
            ---------------------------------------------------
            Créer les 3 ressources /start, /stop, /status :
for ROUTE in start stop status; do
  aws --endpoint-url=http://localhost:4566 apigateway create-resource \
    --rest-api-id <API_ID> \
    --parent-id <ROOT_RESOURCE_ID> \
    --path-part $ROUTE \
    --region us-east-1
done
Attacher la méthode POST et l'intégration Lambda sur chaque ressource :
for RESOURCE_ID in <ID_START> <ID_STOP> <ID_STATUS>; do
  aws --endpoint-url=http://localhost:4566 apigateway put-method \
    --rest-api-id <API_ID> \
    --resource-id $RESOURCE_ID \
    --http-method POST \
    --authorization-type NONE \
    --region us-east-1

  aws --endpoint-url=http://localhost:4566 apigateway put-integration \
    --rest-api-id <API_ID> \
    --resource-id $RESOURCE_ID \
    --http-method POST \
    --type AWS_PROXY \
    --integration-http-method POST \
    --uri arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:000000000000:function:ec2-controller/invocations \
    --region us-east-1
done
Déployer l'API :
aws --endpoint-url=http://localhost:4566 apigateway create-deployment \
  --rest-api-id <API_ID> \
  --stage-name prod \
  --region us-east-1
  Utilisation de l'API
  Les 3 endpoints s'utilisent avec une requête HTTP POST :
Démarrer l'instance
curl -X POST https://<CODESPACE_URL>/restapis/<API_ID>/prod/_user_request_/start
Réponse :
{"message": "Instance i-xxxx démarrée", "instanceId": "i-xxxx"}
Stopper l'instance
curl -X POST https://<CODESPACE_URL>/restapis/<API_ID>/prod/_user_request_/stop
Réponse :
{"message": "Instance i-xxxx stoppée", "instanceId": "i-xxxx"}
Vérifier le statut
curl -X POST https://<CODESPACE_URL>/restapis/<API_ID>/prod/_user_request_/status
Réponse :
{"instanceId": "i-xxxx", "state": "running"}
Endpoints publics
---------------------------------------------------
Start  → https://upgraded-happiness-v6x77jrj9r593x6v9-4566.app.github.dev/restapis/kqf8ly3sti/prod/_user_request_/start 
Stop   → https://upgraded-happiness-v6x77jrj9r593x6v9-4566.app.github.dev/restapis/kqf8ly3sti/prod/_user_request_/stop  Status → https://upgraded-happiness-v6x77jrj9r593x6v9-4566.app.github.dev/restapis/kqf8ly3sti/prod/_user_request_/status
---------------------------------------------------
Evaluation
---------------------------------------------------
Cet atelier, **noté sur 20 points**, est évalué sur la base du barème suivant :  
- Repository exécutable sans erreur majeure (4 points)
- Fonctionnement conforme au scénario annoncé (4 points)
- Degré d'automatisation du projet (utilisation de Makefile ? script ? ...) (4 points)
- Qualité du Readme (lisibilité, erreur, ...) (4 points)
- Processus travail (quantité de commits, cohérence globale, interventions externes, ...) (4 points) 

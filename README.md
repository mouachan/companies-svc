# Comapnies services 

## Prerequesties 
Install :
- oc client
- knative client (kn)
- openshift serverless operator from the operatorhub on your Openshift cluster

## Create a registry secret

```
oc create secret docker-registry quay-secret \
    --docker-server=quay.io/mouachan \
    --docker-username= \
    --docker-password=
oc secrets link builder quay-secret
oc secrets link default quay-secret --for=pull
```

## Clone the source from github
```
https://github.com/mouachan/companies-svc.git

```
## Create a new mongodb app

```
oc new-app mongodb-persistent --name=mongodb -p DATABASE_SERVICE_NAME=mongodb -p MONGODB_DATABASE=companies -l -p app=companies-svc  -p MONGODB_ADMIN_PASSWORD=r3dhat2020! -n companies-credit
```
## Create DB and collection
```
#create db
use companies
#create collection
db.createCollection( "companyInfo", {
   validator: { $jsonSchema: {
      bsonType: "object",
      required: [ "siren" ],
      properties: {
         siren: {
            bsonType: "string",
            description: "must be a string and is required"
         },
         denomination: {
            bsonType : "string",
            description: "must be a string"
         },
         siret: {
            bsonType : "string",
            description: "must be a string"
         },
         address: {
            bsonType : "string",
            description: "must be a string"
         },
         capitalSocial: {
            bsonType : "string",
            description: "must be a String"
         },
         chiffreAffaire: {
            bsonType : "string",
            description: "must be a String"
         },
          trancheEffectif: {
            bsonType : "string",
            description: "must be a String"
         },
          tva: {
            bsonType : "string",
            description: "must be a string"
         },
         immatriculationDate: {
            bsonType : "date",
            description: "must be a Date"
         },
          type: {
            bsonType : "string",
            description: "must be a string"
         },
         updateDate: {
            bsonType : "date",
            description: "must be a Date"
         }
      }
   } }
} )
```
## Add companies
```
  var immatriculationDate = new Date(1954, 12, 24);
    var updateDate =  new Date(2020, 04, 11);
  db.companyInfo.insert({  siren: "542107651",
         "denomination": "ENGIE",
         "siret": "54210765113030",
         "address": "ENGIE, 1 PL SAMUEL DE CHAMPLAIN 92400 COURBEVOIE",
         "tva": "FR03542107651",
         "immatriculationDate": immatriculationDate,
         type: "SA à conseil d'administration",
         "updateDate": updateDate,
         "capitalSocial": "2 435 285 011,00 €",
         "chiffreAffaire": "27 833 000 000.00 €",
         "trancheEffectif": "5000 à 9999 salariés"
              }
        )

     var immatriculationDate = new Date(1920, 12, 01);
    var updateDate =  new Date(2020, 04, 01);
        db.companyInfo.insert({  siren: "423646512",
         "denomination": "FOURNIL SAINT JACQUES",
         "siret": "    42364651200016",
         "address": "11 RUE DE LA TOMBE ISSOIRE 75014 PARIS",
         "tva": "FR03423646512",
         "immatriculationDate": immatriculationDate,
         "type": "Société à responsabilité limitée",
         "updateDate": updateDate,
         "capitalSocial": "7 622,45 €",
         "chiffreAffaire": "NON Connu",
         "trancheEffectif": "1 à 2 salariés"
        })

     var immatriculationDate = new Date(2009, 02, 26);
    var updateDate =  new Date(2020, 04, 01);
      db.companyInfo.insert({  siren: "510662190",
         denomination: "ASHILEA",
         siret: "    51066219000014",
         address: "49 AV DE SAINT OUEN 75017 PARIS",
         tva: "FR0510662190",
         "immatriculationDate": immatriculationDate,
         "type": "Société par actions simplifiée",
         "updateDate": updateDate,
         "capitalSocial": "7 622,45 €",
         "chiffreAffaire": "21 000,00 €",
         "trancheEffectif": "10 salariés"
        }
   )
```
## Build and generate native container image

```
./mvnw clean package  -Dquarkus.container-image.build=true -Dquarkus.container-image.name=companies-svc -Dquarkus.container-image.tag=native-1.0 -Pnative  -Dquarkus.native.container-build=true
```

## Push the image to your registry (change the username by yours)

Make sure that you are connected to your images registry and create a repo named frequent-flyer
```
docker tag mouachani/companies-svc:native-1.0 quay.io/mouachan/companies-svc:native-1.0
docker push quay.io/mouachan/companies-svc:native-1.0
```

## Create serverless service
```
echo "apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: companies-svc-native
spec:
  template:
    metadata:
      name: companies-svc-native-v1
    spec:
      containers:
        - image: >-
            quay.io/mouachan/companies-svc:native-1.0
          env:
            - name: JAVA_OPTS
              value: "-Dvertx.cacheDirBase=/work/vertx"
      imagePullSecrets:
        - name: quay-secret" | oc apply -f -
```  
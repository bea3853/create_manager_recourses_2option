# create_manager_recourses_2option


**1 cria a instacia da vm 

crie a maquina manualmente 

existe aopção de não ser manual, mas gera erro. 

..............................................................................

**2 crie o cluster e exponha

declare sua zona e regiao nas variaveis

export zone=europe-west4-b
export region=europe-west4



gcloud config set compute/zone europe-west4-b

gcloud container clusters create cluster1

gcloud container clusters get-credentials cluster1

kubectl create deployment hello-server \
          --image=gcr.io/google-samples/hello-app:2.0


kubectl expose deployment hello-server --type=LoadBalancer --port 8082

..............................................................................


**3

#guardar a região em uma variavel
export REGION=europe-west4




gcloud compute instance-templates create web-server-template \
          --metadata-from-file startup-script=startup.sh \
          --network default \
          --machine-type e2-medium \
          --region $REGION



gcloud compute target-pools create nginx-pool --region $REGION




gcloud compute instance-groups managed create web-server-group \
--base-instance-name web-server \
--size 2 \
--template web-server-template \
--region $REGION



#mudar as regras
gcloud compute firewall-rules create grant-tcp-rule-456  \
          --allow tcp:80 \
          --network default



gcloud compute http-health-checks create http-basic-check



gcloud compute instance-groups managed \
       set-named-ports web-server-group \
       --named-ports http:80 \
       --region $REGION





gcloud compute backend-services create web-server-backend \
       --protocol HTTP \
       --http-health-checks http-basic-check \
       --global


gcloud compute backend-services add-backend web-server-backend \
       --instance-group web-server-group \
       --instance-group-region $REGION \
       --global


gcloud compute url-maps create web-server-map \
       --default-service web-server-backend




gcloud compute target-http-proxies create http-lb-proxy \
       --url-map web-server-map




#mudar as regras 
gcloud compute forwarding-rules create grant-tcp-rule-456 \
     --global \
     --target-http-proxy http-lb-proxy \
     --ports 80





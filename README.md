
------------------
Service Mesh demo
------------------

### Get Gateway URL
oc -n <name space for isio control pane> get route istio-ingressgateway -o jsonpath='{.spec.host}'
oc -n bookinfo get route istio-ingressgateway -o jsonpath='{.spec.host}'


#########################################################

# Deploy app step (redhad lab)

1. create project
Project name
- istio-system
- demo-bookinfo
- demo-catalog
- demo-inventory

command

	oc new-project istio-system
	oc new-project demo-bookinfo
	oc new-project demo-catalog
	oc new-project demo-inventory
	

2. create the ServiceMeshMemberRoll under namespace istio-system

 apiVersion: maistra.io/v1
 kind: ServiceMeshMemberRoll
 metadata:
   name: default
   namespace: istio-system
 spec:
   members:
     - demo-bookinfo 
     - demo-catalog
     - demo-inventory
  

3. Deploy Bookinfo micorservice web app

oc apply -n demo-bookinfo -f https://raw.githubusercontent.com/ratthanin-pu/bookinfo-microservice/main/istio/bookinfo.yaml

4. Deploy gateway 
oc apply -n demo-bookinfo -f https://raw.githubusercontent.com/ratthanin-pu/bookinfo-microservice/main/istio/bookinfo-gateway.yaml

5. Deploy destination rule for istio
oc apply -n demo-bookinfo -f https://raw.githubusercontent.com/ratthanin-pu/bookinfo-microservice/main/istio/destination-rule-all.yaml

6. Get destinationrules
oc get -n demo-bookinfo destinationrules

7. Get istio route url
### Get Gateway URL
oc -n <name space for isio control pane> get route istio-ingressgateway -o jsonpath='{.spec.host}')

oc -n istio-system get route istio-ingressgateway -o jsonpath='{.spec.host}'

8. Apply proxy sidecar envoy
oc patch -n demo-bookinfo virtualservice/bookinfo --type='json' -p '[{"op":"add","path":"/spec/hosts","value": ["<Gateway URL>"]}]'

# example
oc patch -n demo-bookinfo virtualservice/bookinfo --type='json' -p '[{"op":"add","path":"/spec/hosts","value": ["istio-ingressgateway-istio-system.apps.cluster-kqv6n.kqv6n.sandbox1267.opentlc.com"]}]'


### Test open web URL for bookinfo
http://istio-ingressgateway-istio-system.apps.cluster-kqv6n.kqv6n.sandbox1267.opentlc.com/productpage
user : jason
pass : jason

9. configure label and annotate
oc project demo-bookinfo && \
oc label deployment/productpage-v1 app.openshift.io/runtime=python --overwrite && \
oc label deployment/details-v1 app.openshift.io/runtime=ruby --overwrite && \
oc label deployment/reviews-v1 app.openshift.io/runtime=java --overwrite && \
oc label deployment/reviews-v2 app.openshift.io/runtime=java --overwrite && \
oc label deployment/reviews-v3 app.openshift.io/runtime=java --overwrite && \
oc label deployment/ratings-v1 app.openshift.io/runtime=nodejs --overwrite && \
oc label deployment/details-v1 app.kubernetes.io/part-of=bookinfo --overwrite && \
oc label deployment/productpage-v1 app.kubernetes.io/part-of=bookinfo --overwrite && \
oc label deployment/ratings-v1 app.kubernetes.io/part-of=bookinfo --overwrite && \
oc label deployment/reviews-v1 app.kubernetes.io/part-of=bookinfo --overwrite && \
oc label deployment/reviews-v2 app.kubernetes.io/part-of=bookinfo --overwrite && \
oc label deployment/reviews-v3 app.kubernetes.io/part-of=bookinfo --overwrite && \
oc annotate deployment/productpage-v1 app.openshift.io/connects-to=reviews-v1,reviews-v2,reviews-v3,details-v1 && \
oc annotate deployment/reviews-v2 app.openshift.io/connects-to=ratings-v1 && \
oc annotate deployment/reviews-v3 app.openshift.io/connects-to=ratings-v1

10. configure rollout
oc rollout status -n demo-bookinfo -w deployment/productpage-v1 && \
oc rollout status -n demo-bookinfo -w deployment/reviews-v1 && \
oc rollout status -n demo-bookinfo -w deployment/reviews-v2 && \
oc rollout status -n demo-bookinfo -w deployment/reviews-v3 && \
oc rollout status -n demo-bookinfo -w deployment/details-v1 && \
oc rollout status -n demo-bookinfo -w deployment/ratings-v1


oc get pods -n demo-bookinfo --selector app=reviews


# Generating application load

BOOK_URL=istio-ingressgateway-istio-system.apps.cluster-kqv6n.kqv6n.sandbox1267.opentlc.com

Teseting workload web

siege -c 4 http://$BOOK_URL/productpage
or
for i in {1..1000} ; do curl -o /dev/null -s -w "%{http_code}\n" http://$BOOK_URL/productpage ; sleep 2 ; done
  
  
  
  
   

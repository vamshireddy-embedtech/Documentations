Nginx Ingress Controller
*********************************************************************************************
1) curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
2) helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
3)helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.publishService.enabled=true --namespace=nginx-ingress
4)kubectl --namespace nginx-ingress get services -o wide -w nginx-ingress-ingress-nginx-controller


**********************any web hooks issue ****************************************************************
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
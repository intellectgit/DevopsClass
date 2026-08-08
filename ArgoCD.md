Why “kubectl apply” is not enough in real production

How Argo CD keeps Live State matching Git Desired State

Day 1 demo: Install Argo CD + first application sync
Github Link: https://github.com/TechITFactory/ARGO...

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/arg...

kubectl get pods -n argocd -w
kubectl port-forward svc/argocd-server -n argocd 8080:443

argocd admin initial-password -n argocd
argocd login localhost:8080 --username admin --password PASTE_PASSWORD --insecure

argocd app create guestbook \
  --repo [https://github.com/argoproj/argocd-ex...](https://github.com/argoproj/argocd-example-apps/tree/master/guestbook) \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default


argocd app sync guestbook
argocd app get guestbook

kubectl scale deploy guestbook-ui -n default --replicas=3

argocd app sync guestbook

# k8s-ingress-demo
Nginx-Ingress demo on Kubernetes  

**デモ環境**  
K3sで3台構成のクラスターを構成したもの。  
バージョンは以下の通り。  
~~~
$ kubectl version --short
Client Version: v1.23.4
Server Version: v1.22.5+k3s1
$ helm version --short
v3.8.0+gd141386
~~~

**ソース**
今回使用するIngressのリポジトリ※Nginxのものではない  
https://github.com/kubernetes/ingress-nginx/  
公式のインストールドキュメント  
https://github.com/kubernetes/ingress-nginx/blob/main/docs/deploy/index.md  

**前提条件**  
ロードバランサーがKubernetes上に設定されていることが条件となる。  
デモの実施環境はK3sのためMetalLBに変更している。  
KlipperLBはちゃんと動かないし、MetalLBにすることで設定をちゃんと入れることができるので、  
以下のリンクに従って、KlipperLBを無効化  
https://hassiweb.gitlab.io/memo/docs/memo/k3s/k3s-disable-klipper-lb/  
続いて、MetalLBをインストール  
https://metallb.universe.tf/installation/#installation-by-manifest  
続いて、ConfigMapを設定  
https://metallb.universe.tf/configuration/  

**インストール**  
~~~
$ helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
~~~

**デプロイ**
~~~
$ kubectl create -f nginx-deploy-main.yaml -f nginx-deploy-blue.yaml -f nginx-deploy-green.yaml
deployment.apps/nginx-deploy-main created
deployment.apps/nginx-deploy-blue created
deployment.apps/nginx-deploy-green created
~~~
ポートも公開(サービスを作成と同じこと)
~~~
$ kubectl expose deploy nginx-deploy-main --port 80
service/nginx-deploy-main exposed
$ kubectl expose deploy nginx-deploy-blue --port 80
service/nginx-deploy-blue exposed
$ kubectl expose deploy nginx-deploy-green --port 80
service/nginx-deploy-green exposed
~~~

**各Ingressの設定と削除**
~~~
$ kubectl create -f ingress-resource-1.yaml
ingress.networking.k8s.io/ingress-resource-1 created
$ kubectl get ing
NAME                 CLASS    HOSTS               ADDRESS         PORTS   AGE
ingress-resource-1   <none>   nginx.example.com   192.168.1.XXX   80      31s
##ブラウザで動作確認
$ kubectl delete ing ingress-resource-1
ingress.networking.k8s.io "ingress-resource-1" deleted
~~~
~~~
$ kubectl create -f ingress-resource-2.yaml
ingress.networking.k8s.io/ingress-resource-2 created
$ kubectl get ing
NAME                 CLASS    HOSTS                                                              ADDRESS   PORTS   AGE
ingress-resource-2   <none>   nginx.example.com,blue.nginx.example.com,green.nginx.example.com             80      6s
##ブラウザで動作確認
$ kubectl delete ing ingress-resource-2
ingress.networking.k8s.io "ingress-resource-2" deleted
~~~
~~~
$ kubectl create -f ingress-resource-3.yaml
ingress.networking.k8s.io/ingress-resource-3 created
$ kubectl get ing
NAME                 CLASS    HOSTS               ADDRESS   PORTS   AGE
ingress-resource-3   <none>   nginx.example.com             80      17s
##ブラウザで動作確認
$ kubectl delete ing ingress-resource-3
ingress.networking.k8s.io "ingress-resource-3" deleted
~~~

**デモが終わったら後始末**
~~~
$ kubectl delete service nginx-deploy-main
service "nginx-deploy-main" deleted
$ kubectl delete service nginx-deploy-blue
service "nginx-deploy-blue" deleted
$ kubectl delete service nginx-deploy-green
service "nginx-deploy-green" deleted

$ kubectl delete deployment/nginx-deploy-main
deployment.apps "nginx-deploy-main" deleted
$ kubectl delete deployment/nginx-deploy-blue
deployment.apps "nginx-deploy-blue" deleted
$ kubectl delete deployment/nginx-deploy-green
deployment.apps "nginx-deploy-green" deleted
~~~

さくっと、Ingressのドメイン毎のマイクロサービスへの振り分け、パスの振り分けの実験が終了。  

こちらのビデオに感謝。  
しかし、バージョンが変わったらアップデートして欲しいものだ。  
https://www.youtube.com/watch?v=2VUQ4WjLxDg  

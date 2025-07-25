
### To install kube, get the script from this repo

```
https://github.com/networknuts/kubernetes/blob/master/scripts/packages-kubernetes-1.32
```

```
vim packages.sh # paste the copied code from above repo
sudo sh packages.sh
```


```
sudo kubeadm config images pull
```

```
sudo kubeadm init --apiserver-advertise-address 10.0.0.100 --pod-network-cidr 172.17.0.0/16
```
=> Note: Then you shoud run the three commands to start the cluster, you can see it in the end of init result

<img width="645" height="102" alt="image" src="https://github.com/user-attachments/assets/4abd842a-558e-4bf6-8e00-e85ef3378b25" />

=> then install calico => get the script from `https://github.com/networknuts/kubernetes/blob/master/scripts/calico.yaml`
```
vim calico.yaml
kubectl apply -f calico.yaml
```

=> then check the status of kube-system
```
kubectl -n kube-system get pods
```




1. Compare the application logs before and after exposing it as a Service<br>

![Image](https://github.com/user-attachments/assets/f13f75e8-10fa-4848-a6ab-fd94e05c858a)

Before exposing the application as a Service, the logs only showed startup messages such as "Started HTTP server on port 8080" and "Started UDP server on port 8081," indicating that the container was running but not receiving any external traffic. However, after using kubectl expose to expose the app as a LoadBalancer Service and accessing it through minikube service hello-node, the logs started to include additional entries like GET /. These repeated log entries confirm that each time I opened the app in the browser, a new HTTP request was received and processed by the container. This demonstrated that exposing the app as a Service successfully made the Pod accessible to external traffic, and every incoming request was correctly logged by the container, verifying that the Service was working as intended.

2. What is the purpose of the -n option in kubectl get?<br>

kubectl get services

![Image](https://github.com/user-attachments/assets/172f1ec3-3ef8-42eb-a5a4-549a838f0cb3)

kubectl get pods,services -n kube-system

![Image](https://github.com/user-attachments/assets/8c20307b-a04a-43b1-ba46-23dde4943047)

The -n option in kubectl get is used to specify the namespace from which to list resources. When I used kubectl get without the -n flag, it showed resources from the default namespace, where my hello-node deployment and service were created. That’s why I could see my user-created resources there. However, when I ran kubectl get pods,services -n kube-system, it showed only the system-managed components like the metrics-server, kube-dns, etcd, and kube-apiserver. These system-level components run in the kube-system namespace, not in the default one. That’s why my application’s deployment and service did not appear when I queried the kube-system namespace. In summary, the -n option is crucial to switch the namespace context in Kubernetes and see resources specific to that namespace.

Difference Between Rolling Update and Recreate Deployment Strategies
The Rolling Update deployment strategy updates Pods gradually, one at a time or in small batches, without causing downtime. It ensures that at least some Pods of the previous version remain running while the new version is being rolled out. In contrast, the Recreate deployment strategy terminates all existing Pods first, then creates new ones with the updated version. This causes a short downtime because there is a gap between old Pods being terminated and new ones being ready.
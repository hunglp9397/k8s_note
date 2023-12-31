# Kubernetes Note 

### 1. Kiến trúc của Kubernetes
- Kubernetes cluster bao gồm 2 thành phần chính sau:
  + Master nodes
  + Worker notes
- Master node bao gồm 4 thành phần chính là : API Manager, Controller Manager, Scheduler và Etcd
  + Api Server : Thành phần chính giao tiếp với các thành phần khác
  + Controller Manager : gồm nhiều controller riêng cụ thể cho từng resource và thực hiện các chứng năng cụ thể cho từng thằng resource trong kube như create pod, create deployment
  + Scheduler: schedules ứng dụng tới node nào
  + Etcd: là một database để lưu giữ trạng thái và resource của cluster

### 2. Ví dụ tạo Pod
- Source code : [example_pod](/example_pod)
- Tạo file index.js
- Tạo file Dockerfile
- Tạo file hello-kube.yml
- Build images :
```shell
docker build -t 123497/hello-kube .
```
- Run chạy thử container từ images vừa build:
```shell
docker run -d --name hello-kube -p 3000:3000 123497/hello-kube
```
- Truy cập được vào localhost:3000 OK
- Push lên docker hub
```shell
docker push 123497/hello-kube
```
- Apply file tạo pod hello-kube.yml
```shell
kubectl apply -f hello-kube.yml
```
- Kiểm tra pod: 
```shell
kubectl get pod
```

### 3. Tự động tạo và quản lý pod : ReplicationControllers , ReplicaSets
- ReplicationControllers:
  + Là Một resource mà sẽ tạo và quản lý pod, đảm bảo số lượng pod luôn running
  + Bằng cách tạo ra số lượng pod bằng với giá trị chỉ định ở thuộc tính "replica", và quản lý thông qua labels của pod
    ![1.png](img_guide/1.png)
  + Ta biết rằng Pod giám sát container và tự động restart lại container khi nó fail
  + Trong trường hợp toàn bộ worker node fail thì pod sẽ ko chạy nữa ?
  + Nếu chạy cluster với nhiều hơn 1 Worker node, Th ReplicationControllers sẽ giúp giải quyết vấn đề trên
  + Replication Controller nếu phát hiện ra số lượng pod của một worker_node nào đó = 0 -> Sẽ tạo một Pod ở worker node khác để
  + Ngoài ra, Replication Controller còn giúp tăng performance của ứng dụng bằng cách chỉ định số lượng replicas trong Replcation Controller để tạo ra nhiều pod chạy một ứng dụng
  + Ví dụ ta có một webservice, nếu ta chỉ deploy một pod để chạy ứng dụng, thì ta chỉ có 1 container để xử lý request của user, nhưng nếu ta dùng RC và chỉ định replicas = 3, ta sẽ có 3 pod chạy 3 container của ứng dụng, và request của user sẽ được gửi tới 1 trong 3 pod này, giúp quá trình xử lý của chúng ta tăng gấp 3 lần
    ![2.png](img_guide/2.png)


#### 3.1. Ví dụ tạo ReplicationController
- Source code : [example_replication_controller](/example_replication_controller)
  - Kiến trúc của 1 Replication Controller gồm 3 thành phần chính sau:
    + label selector : chỉ định pod nào sẽ được RC giám sát
    ```dockerfile
    spec:
      replicas: 2
      selector:
        app: hello-kube
    ```

  + Replica count : Số lượng pod sẽ được tạo
    ```dockerfile
    spec:
    replicas: 2
    selector:
    app: hello-kube
    ```
    + Pod template : Config của pod sẽ được tạo
    ```dockerfile
    template:
      metadata:
        labels:
          app: hello-kube
      spec:
        containers:
          - image: 123497/hello-kube
            name: hello-kube
            ports:
              - containerPort: 3000
    ```
- Tạo Replication Controller bằng lệnh: `kubectl apply -f hello-rc.yaml`
- Kiểm tra xem đã tạo RC thành công chưa :` kubectl get rc`
  ![3.png](img_guide/3.png)
  - Kiểm tra tình trạng pod: ` kubectl get pod`
    ![4.png](img_guide/4.png)
- Khi xóa một pod thì Replication Controller sẽ tự động tạo một pod khác để đảm bảo số lượng replication = 2
  ![5.png](img_guide/5.png)


#### 3.2. Ví dụ sử dụng Replica Sets
- Replica Sets là một resource tương tự Replication Controller nhưng nó là một phiên bản mới hơn của Replication Controller
- Tạo file ReplicaSet: hello-rs.yaml
```dockerfile

apiVersion: apps/v1
kind: ReplicaSet 
metadata:
  name: hello-rs
spec:
  replicas: 2
  selector:
    matchLabels: 
      app: hello-kube
  template:
    metadata:
      labels:
        app: hello-kube
    spec:
      containers:
      - image: 123497/hello-kube
        name: hello-kube
        ports:
          - containerPort: 3000
```

- Apply file replica-set : kubectl apply -f hello-rs.yaml


- So sánh ReplicaSets và ReplicationController
- RS và RC sẽ hoạt động tương tự nhau. Nhưng RS linh hoạt hơn ở phần label selector, trong khi label selector thằng RC chỉ có thể chọn pod mà hoàn toàn giống với label nó chỉ định, thì thằng RS sẽ cho phép dùng một số expressions hoặc matching để chọn pod nó quản lý.


### 4. Service trong k8s
- Mỗi Service có một IP và một port không đổi, trừ khi xóa service đi và tạo lại
- Client chỉ cần tương tác với endpoint là service
- Service có 4 loại cơ bản là :
  + Cluster IP
  + Node port
  + External name
  + Load Balancer


#### 4.1 Cluster IP
- Là 1 loại service mà sẽ tạo ra một IP và một local DNS mà có thể truy cập bên trong cluster (node), ko thể truy cập ra bên ngoài, được dùng chủ yếu cho các pod trong cluster giao tiếp với nhau
- **Source code** : [example_service/service_cluster_ip](/example_service/service_cluster_ip)


#####  4.1.1 Build images & push docker
- Tạo file index.js
- Tạo file Dockerfile
- Build images: `docker build -t 1234997/demo-redis`
- Push lên docker hub : `docker push 1234997/demo-redis:latest`

##### 4.1.2: Tạo pod redis & Service
- Tạo file redis-service.yaml

- Apply file redis-service.yaml : `kubectl apply -f redis-service.yaml`
- Kết quả : 
   + Pod: ![5.png](img_guide/5.png)
   + Service : ![6.png](img_guide/6.png

- Đang lôi không run đc để demo :()


#### 4.2 NodePort:
- Tương tự như Cluster IP là tạo endpoint để các container truy cập, ngoài ra nó sử dụng một port của toàn bộ worker node để client bên ngoài có thể truy cập vào.
- **Source code** : [example_service/service_node_port](/example_service/service_node_port)
- Tạo file hello-nodeport.yaml
- Apply replicaset và service đã được cấu hình trong file vừa tạo: `kubectl apply -f hello-nodeport.yaml`
- Kiểm tra Pod với 2 replication mà đã khai báo trong replicaset:
````dockerfile
kind: ReplicaSet
metadata:
  name: hello-rs
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-kube
````
![7.png](img_guide/7.png)
- Kiểm tra service: `kubectl get svc`

```dockerfile
kind: Service
metadata:
  name: hello
spec:
  selector:
    app: hello-kube
  type: NodePort
```
  ![8.png](img_guide/8.png)
- Start service thì sẽ ánh xạ cổng 3000 của container ra cổng 32000
  ![9.png](img_guide/9.png)
- Kết quả: 
  ![11.png](img_guide/11.png)

#### 4.3 LoadBalancer
#### 4.4 Ingress resource




### 5. Vấn đề khi upcode một ứng dụng. Deployment trong K8S
#### 5.1 Đặt vấn đề 
- Giả sử ứng dụng đang chạy với ReplicaSet, replicas = 3 (3 POD), Một service để expose cho client
- Giờ cần upcode chức năng mới, ta cần update lại các Pod đang chạy với một image mới 
- Thì sẽ có 2 cách thông dụng nhất là **Recreate** và **Rolling Update**

#### 5.2 Recreate
- ![12.png](img_guide/12.png)
- Cách deploy này là: 
  + Đầu tiên sẽ xóa toàn bộ mọi version cũ của ứng dụng, Sau đó sẽ deploy một version mới lên
  + Còn đối với ứng dụng triển khai k8s thì đầu tiên sẽ cập nhật pod template của ReplicaSet, Sau đó xóa toàn bọ pod hiện tại, Để replicaSet tạo ra Pod với image mới
  + -> Với cách deploy này thì quá trình deploy dễ dàng nhưng gặp một vấn đề là downtime của ứng dụng lớn, client ko thể request tới ứng dụng cho đến khi ứng dụng đc up lên
  + -> Cách này ko ổn đối với các hệ thống ngân hàng cần request liên tục


### 5.3 Rolling Update
- ![13.png](img_guide/13.png)
- Cách deploy này là: Xóa từng pod cũ, Dẫn request tới version cũ, Lặp lại quá trình này cho tới khi toàn bộ version mới của ứng dụng được deploy, và tất cả version cũ bị xóa
- Ưu điểm của cách này : Giảm tgian downtime của ứng dụng, client sẽ mất ít tgian chờ để deploy 
- Nhợc điểm : Ta có version mới và version cũ của một pod chạy song song, 


### 6. Deployment: Giải quyết được vấn đề upcode một ứng dung
- Deployment là 1 resource trong K8S cung cấp sẵn 2 cách Recreate và RollingUpdate
- Tất cả đều đc thực hiện tự động bên dưới, các version deploy sẽ có history đằng sau do vậy có thể rollback hoặc rollout giữa các phiên bản mà ko cần chạy lại CI/CD
- Luồng hoạt động cơ bản là : Deployment tạo ReplicaSet, ReplicaSet tạo Pods
  ![14.png](img_guide/14.png)


#### 6.1 : Ví dụ upcode đơn giản sử dụng Deployment
- Source code : [example_deployment](/example_deployment/ex_1)
- Deploy version1: 
  + Tạo file index.js
  + Tạo file Dockerfile
  + Build images : `docker build -t 123497/hello-deployment:v1 .`
  + Push lên Docker Hub cho ver1 : `docker push 123497/hello-deployment:v1`
  + Tạo file deploy (Bao gồm khai báo deployment và Service): hello-deploy.yaml
  + Apply file deploy : `kubectl apply -f hello-deployment.yaml`
  + Kết quả deploy version 1:
    ![16.png](img_guide/16.png)
    ![17.png](img_guide/17.png)
    ![18.png](img_guide/18.png)
  + Ứng dụng chạy thành công
- Deploy version2:
  + Sửa file index.js đổi code thành ver2:
  + Build lại images `docker build -t 123497/hello-deployment:v2 .`
  + ![19.png](img_guide/19.png)
  + Update lại pod bằng cấu trúc lệnh :  kubectl set image deployment <deployment-name> <container-name>=<new-image>
  + `kubectl set image deployment hello-deployment hello-deployment=123497/hello-deployment:v2 `
  + Kiểm tra quá trình update đã xong chưa : `kubectl rollout status deploy hello-deployment`
    ![20.png](img_guide/20.png)
  + Kết quả: (chỉ cẩn refresh lại ứng dụng)
    ![21.png](img_guide/21.png)
  + Cập nhật ứng dụng thành công
- Giải thích nguyên lý hoạt động: 
  + Khi thay đổi image của deployment, K8S sẽ tạo ra một replicaset mới, ReplicaSet mới này sẽ có template Pod mới 
  + ReplicaSet cũ sẽ ko bị xóa, mà thuốc tính 'replicas' của nó có giá trị bằng 0
  + Khi muốn rollback lại version cũ thì ReplicaSet cũ sẵn sàng

#### 6.1 : Rollback lại version trước khi version mới của ứng dụng bị lỗi
- Source code : [example_deployment](/example_deployment/ex_2_rollback)
- Giả sử deploy version3 như file index sau: Gọi request 3 lần thì trả ra lỗi
- Deploy version 3:
  + Build lại images với tag v3 `docker build -t 123497/hello-deployment:v3 .`
  + Push lên hub : `docker push 123497/hello-deployment:v3`
  + Cập nhật deployment `kubectl set image deployment hello-deployment hello-deployment=123497/hello-deployment:v3`
  + Chạy lại service : `minikube service hello-deployment`
  + Kết quả:
    ![23.png](img_guide/23.png)
  + Nếu truy cập lần thứ 3 thì app sẽ báo lỗi
    ![22.png](img_guide/22.png)
- Rollback lại version 2 :
  + Đầu tiên, kiểm tra lịch sử các lần ứng dụng được cập nhật : `kubectl rollout history deploy hello-deployment`
  + Sau đó chạy lệnh sau để rollback về ver 2 : `kubectl rollout undo deployment hello-deployment -to-revision=2`
  + Kết quả:
  ```bash
   PS D:\Workspace\Learning\k8s_note\example_deployment\ex_2_rollback> kubectl rollout undo deployment hello-deployment --to-revision=2
   deployment.apps/hello-deployment rolled back
  ```
  + ![24.png](img_guide/24.png)


### 7 : Kubernetes Volume:
- Volume hiểu đơn giản là một mount point từ hệ thống file của server vào trong container
- Vì data ghi vào chỉ tồn tại khi container chạy. Còn khi một Pod bị xóa, Thì container mới sẽ được tạo ra. Khi đó data ghi ở container trước sẽ bị mất đi. Vì vậy ta cần dùng volume
- Trong K8S có các loại volume sau:
  + emptyDir
  + hostPath
  + gitRepo
  + nfs
  + configMap, secret, downwardAPI
  + PersistentVolumeClaim


#### 7.1 emptyDir : share data giữa các container
- emptyDir là loại volume đơn giản nhất, nó sẽ tạo ra một emptyDirectory bn trong các Pod
- Các container bên trong 1 Pod có thể ghi dữ liệu vào bên trong nó
- Volumne emptyDir sẽ chỉ tồn tại trong vòng đời của một Pod do vậy dữ liệu sẽ bị mất đi khi Pod bị xóa
- Lưu ý là dùng emptyDir khi chia sẻ dữ liê giữa các container 
- Source code : [example_volume](/example_volume/emptyDir)
- Implement emptyDir volumne:
  + Pull image _luksa/fortune_ : `docker pull luksa/fortune`